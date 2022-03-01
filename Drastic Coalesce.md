# Coalesce in Spark — 

Spark offers two transformations to resize(increase/decrease) the RDD/DF/DS  
1. repartition
2. coalesce

`repartition` results in shuffle, can be used to increase/decrease the partitions  
`coalesce` may or may not result in shuffle, directly depends on the # of partitions  

In case of partition increase, coalesce behavior is same as repartition  
but in case of patition decrease, `coalesce optimized data shuffle by merging local partitions ie. within executor`  

We've 2 types of coalesce as follows:  
1. coalesce
2. **drastic coalesce**

sample example:  

`dataframe.coalesce(n)`, where `n` is # of partitions.  
`if `n < # of partitions `then drastic coalesce takes place which leads to data shuffle, o'wise shuffle doesn't take place`  

if we go from 1000 partitions to 100 partitions,  
there will not be a shuffle, instead each of the 100 new partitions will  
claim 10 of the current partitions. If a larger number of partitions is requested,  
it will stay at the current number of partitions.  

However, if we're doing a drastic coalesce, e.g. to numPartitions = 1,  
this may result in your computation taking place on fewer nodes than we like  
(e.g. one node in the case of numPartitions = 1). 
To avoid this, we can call repartition(). 
This will add a shuffle step, but means the current upstream partitions will be executed in parallel  

best practice:  
if we've imbalanced size of partitions, it's better to re-distribute then using different partitioning logic  

**Reference:**  
1. https://stackoverflow.com/a/67915928/6842300
2. https://spark.apache.org/docs/latest/api/python/_modules/pyspark/sql/dataframe.html#DataFrame.coalesce

