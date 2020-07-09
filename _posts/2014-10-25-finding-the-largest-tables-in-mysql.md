---
title: Finding the largest tables in MySql
layout: post
comments: true
published: false
description: 
keywords: 
---

This SQL determines the largest tables of a MySQL server.

```sql
SELECT CONCAT(table_schema, '.', table_name) AS tablename,
       table_rows,
       CONCAT(ROUND(data_length / ( 1024 * 1024), 2), ' MB') AS data_size,
       CONCAT(ROUND(index_length / ( 1024 * 1024), 2), ' MB') AS index_size,
       CONCAT(ROUND(( data_length + index_length ) / ( 1024 * 1024 ), 2), ' MB') total_size,
       ROUND(index_length / data_length, 2) AS index_frac
FROM information_schema.TABLES
ORDER BY data_length + index_length DESC
LIMIT  100;
```

Source: 
* [Finding out largest tables on MySQL Server](http://www.mysqlperformanceblog.com/2008/02/04/finding-out-largest-tables-on-mysql-server/)