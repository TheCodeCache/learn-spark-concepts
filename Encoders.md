# ToDo

# Encoders:
Central to the concept of `Dataset` is an `Encoder framework`,  
that provides Dataset with `storage` and `execution` efficiency gains as compared to `RDDs`.  

> An encoder of a particular type encodes either an Java object or a data record into the binary format backed by raw memory and vice-versa.  
> `Encoders are part of Spark’s tungusten framework.` Being backed by the raw memory, updation or querying of relevant information from the encoded binary text is done via Java Unsafe APIs.


**Encoder outputs - Binary Format:**  

![image](https://user-images.githubusercontent.com/26399543/146454815-cf8b586c-c683-4487-a945-c8d4e82a76ed.png)

There are the 3 broad benefits provided by Encoders empowering Datasets to their present glory:  
1. Storage efficiency:
2. Query efficiency:
3. Shuffle efficiency:

**Reference:**  
1. https://towardsdatascience.com/apache-spark-dataset-encoders-demystified-4a3026900d63

