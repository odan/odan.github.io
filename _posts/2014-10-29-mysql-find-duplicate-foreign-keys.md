---
title: MySQL - Find duplicate foreign keys
layout: post
comments: true
published: false
description: 
keywords: mysql
---

A foreign key is a field (or collection of fields) in one table that uniquely 
identifies a row of another table. As a database designer you have to ensure 
that all foreign keys are unique. In MySQL you can find all foreign keys in 
the information_schema.key_column_usage table.

Find all duplicate foreign keys:

```sql
SELECT 
    ANY_VALUE(constraint_name) AS 'constraint_name',
    CONCAT(ANY_VALUE(table_name), '.', ANY_VALUE(column_name)) AS 'foreign_key',
    CONCAT(
        ANY_VALUE(referenced_table_name),
        '.',
        ANY_VALUE(referenced_column_name)
    ) AS 'reference'
    #, COUNT(*) AS c
FROM
    information_schema.key_column_usage 
WHERE 
    referenced_table_name IS NOT NULL 
    AND table_schema = DATABASE()
GROUP BY ANY_VALUE(column_name)
# HAVING c > 1 ;
HAVING COUNT(*) > 1;
```

Find all foreign keys:

```sql
SELECT 
  constraint_name AS 'name',
  CONCAT(table_name, '.', column_name) AS 'foreign_key',
  CONCAT(
    referenced_table_name,
    '.',
    referenced_column_name
  ) AS 'reference' 
FROM
  information_schema.key_column_usage 
WHERE referenced_table_name IS NOT NULL 
  AND table_schema = DATABASE() 
ORDER BY NAME;

```

### Source

* <https://stackoverflow.com/questions/688549/finding-duplicate-values-in-mysql>
* <https://lists.mysql.com/mysql/204859>
* <https://stackoverflow.com/questions/854128/find-duplicate-records-in-mysql>
