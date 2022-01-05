# Coalesce in Spark — 

Spark offers two transformations to resize(increase/decrease) the RDD/DF/DS  
1. repartition
2. coalesce

`repartition` results in shuffle, can be used to increase/decrease the partitions  
`coalesce` may or may not result in shuffle, directly depends on the # of partitions  

In case of partition increase, coalesce behavior is same as repartition  
but in case of patition decrease, `coalesce optimized data shuffle by merging local partitions ie. within executor`  

We've 3 tyes of coalesce as follows:  
1. coalesce
2. **drastic coalesce**

sample example:  

`dataframe.coalesce(n)`, where `n` is # of partitions.  
`if `n < # of partitions `then drastic coalesce takes place which leads to data shuffle, o'wise shuffle does'nt take place`  

best practice:  
if we've imbalanced size of partitions, it's better to re-distribute then usinf different partitioning logic  

**Reference:**  
1. https://stackoverflow.com/a/67915928/6842300

