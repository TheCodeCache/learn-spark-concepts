**An `Off-Heap Serializer`**

# Motivation:
Spark workloads are increasingly bottlenecked by CPU and memory use rather than IO and network communication.  

![image](https://user-images.githubusercontent.com/26399543/143824752-9b822b1c-6c60-48cc-93e3-fae512f48ffb.png)

# Goal:
The goal of tungsten substantially improve the memory and CPU efficiency of the Spark applications,  
and push the limits of the underlying hardware.  

**Memory Optimization:**  
As there are many memory overheads while writing the object to java heap.  

Consider a simple string “abcd” that would take 4 bytes to store using UTF-8 encoding.  
JVM’s native String implementation, however, stores this differently to facilitate more common workloads.  
It encodes each character using 2 bytes with UTF-16 encoding, and each String object also contains a 12 byte header and 8 byte hash code.  

Manual memory management by leverage application semantics, which can be very risky if you do not know what you are doing, is a blessing with Spark.  
We used knowledge of data schema (DataFrames) to directly layout the memory ourselves.  
It not only gets rid of GC overheads but lets you minimize the memory footprint. Schema information help to serialized data in less memory.  

There are encoders available for Primitive types (Int, String, etc) and Product types (case classes),  
which are supported by importing `sqlContext.implicits._` for serializing data.  
Aggregation and sorting operation can be done over serialized data itself.  

Aggregation and sorting operation can be done over serialized data itself.  

**Code Generation:**  
Code generation can be used to optimize the CPU efficiency of internal components.  
code generation is to speed up the conversion of data from in-memory binary format to wire-protocol for the shuffle.  
As mentioned earlier, the shuffle is often bottlenecked by data serialization rather than the underlying network.  
With code generation, we can increase the throughput of serialization, and in turn, increase shuffle network throughput.  

The code generated serializer exploits the fact that all rows in a single shuffle have the same schema and generates specialized code for that.  
This made the generated version over 2X faster to shuffle than the Kryo version.  

Code Generation also improves efficiency for generating better and optimized bytecodes for relational expression.  

**Encoders:**  

![image](https://user-images.githubusercontent.com/26399543/143828003-6b7451a9-66af-4f10-8e2a-231572ae9f4d.png)

**Catalyst Query optimizer:**  

`Catalyst Compiles Spark SQL programs to an RDD.` It optimizes relational expression on DataFrame/DataSet to speed up data processing.

![image](https://user-images.githubusercontent.com/26399543/143828520-7142374d-9a5c-4469-b4b8-e0f923690310.png)

Catalyst has knowledge of all the data types and knows the exact schema of our data,  
and has detailed knowledge of computation of we like to perform which helps it to optimize the operations.  

**Optimizations by Catalyst:**  
1. Reordering Operations:  
  The laziness of transformation operations gives us the opportunity to rearrange/reorder the transformations operations before they are executed.  
  Catalyst can decide to rearrange the filter operations pushing all filters as early as possible so that expensive operation like join/count is performed on fewer data.  
2. Reduce the amount of data we must-read:  
  Skip reading in, serializing and sending around parts of the dataset that aren’t needed for our computations.  
  It is difficult to find the part of data which are not required inside the RDD,  
  because it is not structured but in structured we can easily remove columns which are not required.  

![image](https://user-images.githubusercontent.com/26399543/143828839-afb3ba15-4529-4bec-8703-68e140a77570.png)

![image](https://user-images.githubusercontent.com/26399543/143828874-3f15f0b1-36a0-4a5e-8107-f8e9c5935675.png)


It offers the 3 following optimization features:  

1. Off-Heap Memory Management
2. Cache Locality
3. Whole-Stage Code Generation

**Reference:**  
1. https://spoddutur.github.io/spark-notes/deep_dive_into_storage_formats.html
2. https://medium.com/@goyalsaurabh66/project-tungsten-and-catalyst-sql-optimizer-9d3c83806b63
3. https://jaceklaskowski.gitbooks.io/mastering-spark-sql/content/spark-sql-tungsten.html


