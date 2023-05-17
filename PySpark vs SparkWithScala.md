# When to use PySpark and When to use Spark with Scala - 

1. Spark is written in Scala, thus we get native support from Spark
2. Any new feature added into Spark is exposed usign Scala api  
  and slowly propagates across other languages like Python, Java or R    
3. UDFs written in Python are slow compared to writing them in Scala
4. Error messages from Spark (including PySpark) are in Java/Scala,  
  so they are hard for Python developers to understand  
5. PySpark code is converted into Java Objects to run on Executor process,  
  this conversion happens through Py4J channel  
6. Python, by nature, itself is a slow language compare to Scala/Java,  
  It is still popular because of its faster libraries like NumPy/Pandas etc.  

