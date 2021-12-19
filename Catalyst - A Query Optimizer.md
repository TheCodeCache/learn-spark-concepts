# TODO

The final phase:  
**Code Generation:**  
Catalyst relies on a special feature of the Scala language, quasiquotes, to make code generation simpler.  
`Quasiquotes allow the programmatic construction of abstract syntax trees (ASTs) in the Scala language,`  
which can then be fed to the Scala compiler at runtime to generate bytecode.  
We use Catalyst to transform a `tree` representing an expression in SQL to an `AST` for Scala code to evaluate that expression,  
and then `compile` and `run` the generated code.  

**Reference:**  
1. https://blog.bi-geek.com/en/spark-sql-optimizador-catalyst/
2. https://people.csail.mit.edu/matei/papers/2015/sigmod_spark_sql.pdf
3. https://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.308.4668&rep=rep1&type=pdf

