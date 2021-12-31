# spark.sql.sources.**`partitionOverwriteMode - dynamic`**  

For incremental load, we usually use 'SaveMode.Append` mode to write the processed data to S3 as follows -  

```scala
val bigDataFrame = spark.read.parquet("s3://data-lake/date=2020–02–29")
val transformedDataFrame = bigDataFrame.map(…).filter(…)
transformedDataFrame.write
 .partitionBy("campaign", "date")
 .mode(SaveMode.Append)
 .parquet("s3://spark-output"))
```
Outcome:  
![image](https://user-images.githubusercontent.com/26399543/147831625-370ba7e6-0598-47af-8e16-d8f067015ce8.png)

The above will work perfectly fine as long as there is no failure in the write job action.  
**Actual Problem** -  
But in the real-world, applications do fail, so what happens if the application failed mid-execution?  
In that case, some output files might have already been created in the destination folder, while others might not.
To make sure we processed all the data, we will need to re-run the application,  
but then we will end up with duplicate records,  
i.e some records were already added during the first execution, and were added again during the second execution.  
This means the application is not idempotent.  

Write action got failed:  
![image](https://user-images.githubusercontent.com/26399543/147831728-7200b3b3-a8e4-4021-951f-ed60e8537df6.png)

Write action resumed:  
![image](https://user-images.githubusercontent.com/26399543/147831750-308c4395-db1f-48a8-809b-c0c16878d88b.png)

To mitigate this issue, the 'trivial" solution in Spark would be to use `SaveMode.Overwrite`,  
so Spark will overwrite the existing data in the partitioned folder with the data processed in the second execution:  

```scala
val bigDataFrame = spark.read.parquet(“s3://data-lake/date=2020–02–29”)
val transformedDataFrame = bigDataFrame.map(…).filter(…)
transformedDataFrame.write
 .partitionBy(“campaign”, “date”)
 //.mode(SaveMode.Append) // Commented `Append` mode
 .mode(SaveMode.Overwrite) // Using `Overwite` mode 
 .parquet(“s3://spark-output”))
```

The problem with this approach is that it will `overwrite the entire root folder` (s3://spark-output in our example),  
or basically all the partitions (hereinafter `full-overwrite`).  

Implementing a "naive" solution:  
step-1:  Create a map of DataFrames (one per campaign).  
step-2:  Iterate through the map, and for each entry (i.e each campaign) —  
write all the records to the specific partitioned directory  
(e.g "campaign=1/date=2020–02–29/").  

sample code:  
```scala
// NOTE: dataFramesMap is of type Map[code, dataframe], where each // entry represents a campaign
dataFramesMap.foreach(campaign => { 
  val outputPath = “s3://spark-output/”+
    ”campaign=”+campaign.code+”/date=”+date 
  campaign.dataframe.write.mode(SaveMode.Overwrite).
    parquet(outputPath)
})
```  

This solved the `full-overwrite` problem,  
But, after a while, it was observed that the application's performance took a hit  
There was too much of idle time being spent while writing the output to S3.  
This means - the above "naive" solution caused the application to be very inefficient  
in terms of resource utilization, resulting in very long execution time  

So, the Spark `SaveMode.Overwrite`, as well the above naive impl. fail to work.  

**To fix this precise problem,  
`Spark` now writes data partitioned just as `Hive` would —  
which means only the partitions that are touched by the INSERT query get overwritten and the others are not touched**  

```
// NOTE: Setting the following is required, since the default is 
// "static"
sparkSession.conf
 .set("spark.sql.sources.partitionOverwriteMode", "dynamic") // added the configuration part for Dynamic Partition Overwrite 
val bigDataFrame = spark.read.parquet("s3://data-lake/date=2020–02–29")
val transformedDataFrame = bigDataFrame.map(…).filter(…)
transformedDataFrame.write
 .partitionBy("campaign", "date")
 .mode(SaveMode.Overwrite) // with mode Overwrite  
 .parquet("s3://spark-output"))
```

So, with the above spark configuration for `dynamic partition overwrite` now our application:  
- Is idempotent.
- Efficiently utilizes the cluster resources.
- No longer relies on custom code to overwrite only relevant partitions.  


# How does Dynamic Partition Inserts work behind the scenes? — 

1. During the job execution, each task writes its output under a staging directory with a partition path,  
e.g. /path/to/staging/a=1/b=1/xxx.parquet. Note that each partition path can contain 1 or more files  
For code/document snippet —  
[HadoopMapReduceCommitProtocol.scala#L43](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/internal/io/HadoopMapReduceCommitProtocol.scala#L43)

![image](https://user-images.githubusercontent.com/26399543/147832358-567e1dc7-de87-4ebb-9d76-804afaf638d9.png)

2. When committing the job
For code/document snippet —  
[HadoopMapReduceCommitProtocol.scala#L184-L205](https://github.com/apache/spark/blob/v2.4.4/core/src/main/scala/org/apache/spark/internal/io/HadoopMapReduceCommitProtocol.scala#L184-L205)  

(a) We first clean up the corresponding partition directories at destination path, e.g. /path/to/destination/a=1/b=1;  
(b) And then move (copy) files from staging directory to the corresponding partition directories under destination path  
(c) Finally, we delete the staging directory  

These operations (clean up, move, etc.) are filesystem operations,  
and each filesystem that spark supports (e.g local filesystem, HDFS, S3, etc.) has its own implementation of the operations,  
packaged in a filesystem connector.  
For example, renaming a directory is an atomic and cheap action within a local filesystem or HDFS,  
whereas the same operation within object stores (like S3) is usually expensive  
and involves copying the entire data to the destination path and deleting it from the source path.  

Specifically, step 2(b) above invokes fs.rename() for each partition in the staging directory,  
which essentially invokes the rename method of the filesystem connector in use.  
So, if the HDFS connector is in use, the rename operation will be atomic and cheap,  
whereas if an S3 connector is in use, the rename operation will be expensive  


Spark can read and write data in object stores through filesystem connectors implemented in Hadoop [e.g S3A]  
or provided by the infrastructure suppliers themselves [e.g EMRFS by AWS].  
These connectors make the object stores look almost like file systems,  
with directories and files and the classic operations on them such as list, delete and rename.  

When running Spark on an EMR cluster and using S3:// URI,  
the underlying implementation will default to AWS proprietary S3 connector named EMRFS.  

```xml
<property>
 <name>fs.s3.impl</name>
 <value>com.amazon.ws.emr.hadoop.fs.EmrFileSystem</value>
</property>
```

And when we are not running on EMR (for ex: on-prem YARN cluster, Kubernetes or even Spark in local mode),  
and still need to access data on S3, we should use S3A:// URI  

we should use S3A:// URI (as s3:// and s3n:// URI’s will default to legacy connectors that have been removed from Hadoop).  
For S3A:// URI, the underlying implementation will default to Hadoop’s open-source S3 connector named S3A  

Every filesystem connector implements the filesystem operations  
(rename, copy, delete, read) in a different way (which may, or may not, be optimal).  
So even if your application interacts with the same object store (S3 in this case),  
the actual implementation of the filesystem operations can be different  
(depending on which filesystem connector is being used, e.g EMRFS or S3A):  

For S3A: renaming a directory is done sequentially,  
i.e iterating the files within that directory,  
copying each file to the destination directory and periodically deleting the copied files  

EMRFS, on the other hand, renames a directory in a multi-threaded way,  
so files within that directory are handled in parallel  
(it's been concluded based on our observations via stack traces, etc., since the actual code is not open-sourced).  
Thus, the rename phase at the end of our job is much shorter   


**Reference:**  
1. https://medium.com/nmc-techblog/spark-dynamic-partition-inserts-part-1-5b66a145974f
2. https://paulstaab.de/blog/2020/spark-dynamic-partition-overwrite.html

