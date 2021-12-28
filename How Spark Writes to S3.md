# EMRFS S3-optimized committer - 
1. run Spark jobs that use Spark SQL, DataFrames, or Datasets to write files to Amazon S3
2. Multipart uploads must be enabled in Amazon EMR. This is enabled by default

however, little more important to know is when EMRFS S3-optimized committer is not used - 

![image](https://user-images.githubusercontent.com/26399543/147544422-e4401f00-03d7-48e8-a7e4-451328bd8024.png)

there's following special cases where it's not used, which is as follows:

**Case-1**:  
**Dynamic partition overwrite mode**

The following Scala example instructs Spark to use a different commit algorithm,  
which prevents use of the EMRFS S3-optimized committer altogether.  
The code sets the partitionOverwriteMode property to `dynamic`, to overwrite only those partitions to which we're writing data.  
Then, dynamic partition columns are specified by `partitionBy`, and the `write mode` is set to `overwrite`.  

We must configure all three settings to avoid using the EMRFS S3-optimized committer.  

When we do so, Spark executes a different commit algorithm that uses Spark's staging directory,  
which is a temporary directory created under the output location that starts with .spark-staging.  
The algorithm sequentially renames partition directories, which can **`negatively impact performance`**  

```scala
val dataset = spark.range(0, 10)
  .withColumn("dt", expr("date_sub(current_date(), id)"))

dataset.write.mode("overwrite")
  .option("partitionOverwriteMode", "dynamic")
  .partitionBy("dt")
  .parquet("s3://EXAMPLE-DOC-BUCKET/output")
```

The algorithm in Spark 2.4.0 follows these steps:  

1. Task attempts write their output to partition directories under Spark's staging directory  
—for example, `${outputLocation}/spark-staging-${jobID}/k1=v1/k2=v2/`  
2. For each partition written, the task attempt keeps track of relative partition paths—for example, `k1=v1/k2=v2`  
3. When a task completes successfully, it provides the driver with all relative partition paths that it tracked.  
4. After all tasks complete, the job `commit phase`  
  collects all the partition directories that successful task attempts wrote under Spark's staging directory.  
  Spark sequentially renames each of these directories to its final output location using directory tree rename operations.  
5. The staging directory is deleted before the job `commit phase` completes.  

**Case-2**:  
**Custom partition location**  

```scala

val table = "dataset"
val location = "s3://bucket/table"
                            
spark.sql(s"""
  CREATE TABLE $table (id bigint, dt date) 
  USING PARQUET PARTITIONED BY (dt) 
  LOCATION '$location'
""")
                            
// Add a partition using a custom location
val customPartitionLocation = "s3://bucket/custom"
spark.sql(s"""
  ALTER TABLE $table ADD PARTITION (dt='2019-01-28') 
  LOCATION '$customPartitionLocation'
""")
                            
// Add another partition using default location
spark.sql(s"ALTER TABLE $table ADD PARTITION (dt='2019-01-29')")
                            
def asDate(text: String) = lit(text).cast("date")
                            
spark.range(0, 10)
  .withColumn("dt",
    when($"id" > 4, asDate("2019-01-28")).otherwise(asDate("2019-01-29")))
  .write.insertInto(table)
```
The scala code creates following Amazon S3 objects - 
```scala
custom/part-00001-035a2a9c-4a09-4917-8819-e77134342402.c000.snappy.parquet
custom_$folder$
table/_SUCCESS
table/dt=2019-01-29/part-00000-035a2a9c-4a09-4917-8819-e77134342402.c000.snappy.parquet
table/dt=2019-01-29_$folder$
table_$folder$
```
When writing to partitions at custom locations,  
Spark uses a similar commit algorithm as with previous case, is outlined as follows:  

1. When writing output to a partition at a custom location, tasks write to a file under Spark's staging directory,  
  which is created under the final output location. The name of the file includes a random UUID to protect against file collisions.  
  The task attempt keeps track of each file along with the final desired output path.  
2. When a task completes successfully, it provides the driver with the files and their final desired output paths.  
3. After all tasks complete, the job commit phase sequentially renames all files,  
  that were written for partitions at custom locations to their final output paths.  
4. The staging directory is deleted before the job commit phase completes.


**Note:**  
  Amazon EMR uses the `AWS SDK for Java` with Amazon S3 to store input data, log files, and output data.

**Reference:**  
1. https://docs.aws.amazon.com/emr/latest/ReleaseGuide/emr-spark-committer-reqs.html
