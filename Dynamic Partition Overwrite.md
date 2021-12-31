# spark.sql.sources.**`partitionOverwriteMode - dynamic`**  

For code/document snippet —  
[HadoopMapReduceCommitProtocol.scala#L43](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/internal/io/HadoopMapReduceCommitProtocol.scala#L43)

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


**Reference:**  
1. https://medium.com/nmc-techblog/spark-dynamic-partition-inserts-part-1-5b66a145974f
2. https://paulstaab.de/blog/2020/spark-dynamic-partition-overwrite.html

