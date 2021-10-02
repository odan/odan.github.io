---
title: MySQL - Identify Differences Among Databases
layout: post
comments: true
published: false
description: 
keywords: 
---

## Setup

* Visual C++ Redistributable Packages for Visual Studio 2013
 - <https://www.microsoft.com/en-US/download/details.aspx?id=40784>
* MySQL Utilities - http://dev.mysql.com/downloads/utilities/

## mysqldiff

mysqldiff all tables in database:
<https://dev.mysql.com/doc/mysql-utilities/1.5/en/mysqldiff.html>

Example:
```
mysqldiff --server1=root@localhost --server2=root@localhost --difftype=sql --changes-for=server2 --force test:test2
```

## mysqldbcompare

<https://dev.mysql.com/doc/mysql-utilities/1.5/en/mysqldbcompare.html>

Example:
```
mysqldbcompare --server1=root@localhost --server2=root@localhost --difftype=sql 
 --changes-for=server2 --skip-data-check --run-all-tests 
 --skip-checksum-table --skip-data-check --skip-row-count test:test2
```

## Issue

There is a "Bug" or missing feature to disable the export of AUTO_INCREMENT.

* <https://bugs.mysql.com/bug.php?id=79296>
* <https://bugs.mysql.com/bug.php?id=69669>