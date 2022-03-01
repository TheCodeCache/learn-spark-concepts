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
`if `n < # of data-nodes `then drastic coalesce takes place which leads to data shuffle, o'wise shuffle doesn't take place`  

![image](https://user-images.githubusercontent.com/26399543/156186112-4d6bf382-97f1-425f-a2a5-10024bff0dca.png)  

**Best Practice:**  
if we've imbalanced size of partitions, it's better to re-distribute then using different partitioning logic  

**Reference:**  
1. https://stackoverflow.com/a/67915928/6842300
2. https://spark.apache.org/docs/latest/api/python/_modules/pyspark/sql/dataframe.html#DataFrame.coalesce

