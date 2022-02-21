---
title: Designing a database
layout: post
comments: true
published: false
description: 
keywords: 
---

## Schema Naming Conventions 

* Tables: english, lower_snake_case and plural.
* Columns: english, lower_snake_case and singular.
* Don't use [reserved words](http://dev.mysql.com/doc/refman/5.6/en/keywords.html)
* Every table must contain an auto increment primary key 'id' int(11)
* Foreign key names must be unique per database. Format: sourcetable_sourcefield_destinationtable_destinationfield
* The name of an index should match the field name. Multiple fields can be combined with _ (underscore).
* Views, Stored Procedures, Enums should be avoided. Use only tables.
* All tables should be normalized to the third normal form ([3NF](https://en.wikipedia.org/wiki/Database_normalization))
* Follow the [SQL Style Guide by Simon Holywell](http://www.sqlstyle.guide/)

## Best practice, performance tips

* Use LIMIT to preserve memory and speed up queries.
* Try to prevent sub-queries. MySQL sub-queries are very slow.
* JOINS with big tables are very slow. Maybe move more logic to PHP.
* Split big queries into small queries.
* Use an PHP array as buffer and optimize the query to a maximum.
* A query should not take longer then one second.

## Default Engine/Encoding

* Engine: InnoDB
* CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci

To make sure your strings go from PHP to MySQL as UTF-8, make sure your database and tables are all set to the utf8 character set and collation, and that you use the utf8 character set in the PDO connection string.

```php
$pdo = new PDO('mysql:host=127.0.0.1;dbname=test;charset=utf8', 'root', '',
    array(
        PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
        PDO::ATTR_PERSISTENT => false,
        PDO::MYSQL_ATTR_INIT_COMMAND => "SET NAMES utf8mb4 COLLATE utf8mb4_unicode_ci"
    )
);
```

Read more: <https://stackoverflow.com/a/766996/1461181>

## Data types

* Primary and foreign keys: INT(11) signed (4 bytes, max id: 2'147'483'647)
* Strings: VARCHAR(255)
* Medium Text: TEXT (max. 64 KB)
* Text/Blob: LONGTEXT (max. 4 GB)
* Date-Time: DATETIME (Format: YYYY-MM-DD HH:II:SS)
* Boolean: TINYINT(1) NOT NULL DEFAULT '0' (0 = false, 1 = true)
* Float: DOUBLE
* Exact monetary data: DECIMAL(15, 2)

## Basic table

```sql
CREATE TABLE `tablename` (
    `id` INT(11) NOT NULL AUTO_INCREMENT,
    `created_at` DATETIME DEFAULT NULL,
    `created_user_id` INT(11) DEFAULT NULL,
    `updated_at` DATETIME DEFAULT NULL,
    `updated_user_id` INT(11) DEFAULT NULL,
    PRIMARY KEY (`id`),
    KEY `created_user_id` (`created_user_id`),
    KEY `updated_user_id` (`updated_user_id`)
) ENGINE = INNODB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci ;
```

## Basic codelist table (type)

```sql
CREATE TABLE `tablename_types` (
    `id` INT(11) NOT NULL AUTO_INCREMENT,
    `title` VARCHAR (255) COLLATE utf8mb4_unicode_ci DEFAULT NULL,
    `created_at` DATETIME DEFAULT NULL,
    `created_user_id` INT(11) DEFAULT NULL,
    `updated_at` DATETIME DEFAULT NULL,
    `updated_user_id` INT(11) DEFAULT NULL,
    PRIMARY KEY (`id`),
    KEY `created_user_id` (`created_user_id`),
    KEY `updated_user_id` (`updated_user_id`)
) ENGINE = INNODB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci ;
```
