# Using Spark's "Hadoop Free" Build — 

Spark uses Hadoop client libraries for HDFS and YARN.  
Starting in version Spark 1.4, the project packages "`Hadoop free`" builds  
that lets us more easily connect a single Spark binary to any Hadoop version.  
To use these builds, we need to modify `SPARK_DIST_CLASSPATH` to include Hadoop's package jars.  
The most convenient place to do this is by adding an entry in `conf/spark-env.sh`.  

let's check how to connect Spark to Hadoop for different types of distributions -  

For Apache distributions, we can use Hadoop's `classpath` command. For example:  

```shell
### in conf/spark-env.sh ###

# If 'hadoop' binary is on your PATH
export SPARK_DIST_CLASSPATH=$(hadoop classpath)

# With explicit path to 'hadoop' binary
export SPARK_DIST_CLASSPATH=$(/path/to/hadoop/bin/hadoop classpath)

# Passing a Hadoop configuration directory
export SPARK_DIST_CLASSPATH=$(hadoop --config /path/to/configs classpath)
```

**Reference:**  
1. https://spark.apache.org/docs/2.4.4/hadoop-provided.html

