**Optimized way of joining two large tables in Spark:**  

Spark uses SortMerge joins to join large table.  
It consists of hashing each row on both table and shuffle the rows with the same hash into the same partition.  
There the keys are sorted on both side and the sortMerge algorithm is applied.  
This is probably the best approach, though suggestions are open.  

To drastically speed up your sortMerges,  
write your large datasets as a Hive table with pre-bucketing and pre-sorting option (same number of partitions) instead of flat parquet dataset.  

```scala
tableA
  .repartition(2200, $"A", $"B")
  .write
  .bucketBy(2200, "A", "B")
  .sortBy("A", "B")   
  .mode("overwrite")
  .format("parquet")
  .saveAsTable("my_db.table_a")


tableb
  .repartition(2200, $"A", $"B")
  .write
  .bucketBy(2200, "A", "B")
  .sortBy("A", "B")    
  .mode("overwrite")
  .format("parquet")
  .saveAsTable("my_db.table_b")
```
The overhead cost of writing pre-bucketed/pre-sorted table is modest compared to the benefits.

The underlying dataset will still be parquet by default,  
but the Hive metastore (can be Glue metastore on AWS) will contain precious information about how the table is structured.  
Because all possible "joinable" rows are colocated,  
Spark won't shuffle the tables that are pre-bucketd (big savings!) and won't sort the rows within the partition of table that are pre-sorted.  

```scala
val joined = tableA.join(tableB, Seq("A", "B"))
```
Look at the execution plan with and without pre-bucketing.

This will not only save us a lot of time during our joins,  it will make it possible to run very large joins on relatively small cluster without OOM.  
This strategy is commonly used at Amazon  

**Reference:**  
1. https://stackoverflow.com/a/66772672/6842300
2. https://developer.hpe.com/blog/tips-and-best-practices-to-take-advantage-of-spark-2x/
