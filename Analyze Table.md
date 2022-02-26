# ANALYZE TABLE – 
The `ANALYZE TABLE` statement collects statistics about one specific table or all the tables in one specified database,  
that are to be used by the query optimizer to find a better query execution plan.  

**Syntax:**  
```sql
ANALYZE TABLE table_identifier [ partition_spec ]
    COMPUTE STATISTICS [ NOSCAN | FOR COLUMNS col [ , ... ] | FOR ALL COLUMNS ]
```
```sql
ANALYZE TABLES [ { FROM | IN } database_name ] COMPUTE STATISTICS [ NOSCAN ]
```

**Examples:**  

```
CREATE DATABASE school_db;
USE school_db;

CREATE TABLE teachers (name STRING, teacher_id INT);
INSERT INTO teachers VALUES ('Tom', 1), ('Jerry', 2);

CREATE TABLE students (name STRING, student_id INT) PARTITIONED BY (student_id);
INSERT INTO students VALUES ('Mark', 111111), ('John', 222222);

ANALYZE TABLE students COMPUTE STATISTICS NOSCAN;

DESC EXTENDED students;
+--------------------+--------------------+-------+
|            col_name|           data_type|comment|
+--------------------+--------------------+-------+
|                name|              string|   null|
|          student_id|                 int|   null|
|                 ...|                 ...|    ...|
|          Statistics|           864 bytes|       |
|                 ...|                 ...|    ...|
+--------------------+--------------------+-------+

ANALYZE TABLE students COMPUTE STATISTICS;

DESC EXTENDED students;
+--------------------+--------------------+-------+
|            col_name|           data_type|comment|
+--------------------+--------------------+-------+
|                name|              string|   null|
|          student_id|                 int|   null|
|                 ...|                 ...|    ...|
|          Statistics|   864 bytes, 2 rows|       |
|                 ...|                 ...|    ...|
+--------------------+--------------------+-------+

ANALYZE TABLE students PARTITION (student_id = 111111) COMPUTE STATISTICS;

DESC EXTENDED students PARTITION (student_id = 111111);
+--------------------+--------------------+-------+
|            col_name|           data_type|comment|
+--------------------+--------------------+-------+
|                name|              string|   null|
|          student_id|                 int|   null|
|                 ...|                 ...|    ...|
|Partition Statistics|   432 bytes, 1 rows|       |
|                 ...|                 ...|    ...|
+--------------------+--------------------+-------+

ANALYZE TABLE students COMPUTE STATISTICS FOR COLUMNS name;

DESC EXTENDED students name;
+--------------+----------+
|     info_name|info_value|
+--------------+----------+
|      col_name|      name|
|     data_type|    string|
|       comment|      NULL|
|           min|      NULL|
|           max|      NULL|
|     num_nulls|         0|
|distinct_count|         2|
|   avg_col_len|         4|
|   max_col_len|         4|
|     histogram|      NULL|
+--------------+----------+

ANALYZE TABLES IN school_db COMPUTE STATISTICS NOSCAN;

DESC EXTENDED teachers;
+--------------------+--------------------+-------+
|            col_name|           data_type|comment|
+--------------------+--------------------+-------+
|                name|              string|   null|
|          teacher_id|                 int|   null|
|                 ...|                 ...|    ...|
|          Statistics|          1382 bytes|       |
|                 ...|                 ...|    ...|
+--------------------+--------------------+-------+

DESC EXTENDED students;
+--------------------+--------------------+-------+
|            col_name|           data_type|comment|
+--------------------+--------------------+-------+
|                name|              string|   null|
|          student_id|                 int|   null|
|                 ...|                 ...|    ...|
|          Statistics|           864 bytes|       |
|                 ...|                 ...|    ...|
+--------------------+--------------------+-------+

ANALYZE TABLES COMPUTE STATISTICS;

DESC EXTENDED teachers;
+--------------------+--------------------+-------+
|            col_name|           data_type|comment|
+--------------------+--------------------+-------+
|                name|              string|   null|
|          teacher_id|                 int|   null|
|                 ...|                 ...|    ...|
|          Statistics|  1382 bytes, 2 rows|       |
|                 ...|                 ...|    ...|
+--------------------+--------------------+-------+

DESC EXTENDED students;
+--------------------+--------------------+-------+
|            col_name|           data_type|comment|
+--------------------+--------------------+-------+
|                name|              string|   null|
|          student_id|                 int|   null|
|                 ...|                 ...|    ...|
|          Statistics|   864 bytes, 2 rows|       |
|                 ...|                 ...|    ...|
+--------------------+--------------------+-------+
```

**Reference:**  
1. Apache Spark SQL Documentation

