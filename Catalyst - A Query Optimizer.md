# Catalyst Optimizer - 

This link has good content -  

https://www.unraveldata.com/resources/catalyst-analyst-a-deep-dive-into-sparks-optimizer/  

The final phase:  
**Code Generation:**  
Catalyst relies on a special feature of the Scala language, quasiquotes, to make code generation simpler.  
`Quasiquotes allow the programmatic construction of abstract syntax trees (ASTs) in the Scala language,`  
`which can then be fed to the Scala compiler at runtime to generate bytecode.`  
We use `Catalyst` to transform a `tree` representing an expression in SQL to an `AST` for Scala code to evaluate that expression,  
and then `compile` and `run` the generated code.  

As a simple example, consider the Add, Attribute and Literal tree nodes,  
which allowed us to write expressions such as (x+y)+1.  
>Without code generation,  
>such expressions would have to be interpreted for each row of data, by walking down a tree of Add, Attribute and Literal nodes.  
>This introduces large amounts of branches and virtual function calls that slow down execution.  

>With code generation, we can write a function to translate a specific expression tree to a Scala AST as follows:  
```scala
def compile(node: Node ): AST = node match {
  case Literal(value) => 
    q"$value"
  case Attribute (name) => 
    q"row.get($name )"
  case Add(left , right) =>
    q"${compile(left )} + ${compile(right )}"
}
```

![image](https://user-images.githubusercontent.com/26399543/146682492-b13a84a4-df70-4cf2-908a-858a6abe68a5.png)


**Reference:**  
1. https://blog.bi-geek.com/en/spark-sql-optimizador-catalyst/
2. https://people.csail.mit.edu/matei/papers/2015/sigmod_spark_sql.pdf
3. https://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.308.4668&rep=rep1&type=pdf

