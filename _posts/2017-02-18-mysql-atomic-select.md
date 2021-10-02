---
title: MySQL Atomic SELECT
layout: post
comments: true
published: false
description: 
keywords: 
---

Sometimes you need to select rows exclusively for only one single process. 

In this case you have to make sure that only the current and no other process (e.g. other cronjobs) fetches the same rows in the same (milli)second. 

## MySQL "[Locking Reads](https://dev.mysql.com/doc/refman/5.7/en/innodb-locking-reads.html)" 

If you query data and then insert or update related data within the same transaction, the regular SELECT statement does not give enough protection. Other transactions can update or delete the same rows you just queried. InnoDB supports two types of locking reads that offer extra safety:

```sql
SELECT ... FOR UPDATE
```

It has to do with locking the table in transactions. Let's say you have the following:

```sql
START TRANSACTION;
SELECT .. FOR UPDATE;
UPDATE .... ;
COMMIT;
```

After the `SELECT` statement runs, if you have another `SELECT` from a different user, it won't run until your first transaction hits the COMMIT line.

Also note that `FOR UPDATE` outside of a transaction is meaningless.

This behavior has some disadvantages and offers not what I'm looking for. 
* <http://stackoverflow.com/questions/8849518/mysql-select-for-update-behaviour>
* <http://stackoverflow.com/questions/27865992/why-use-select-for-update>

My concern (and experience) was that those records will be modified between the SELECT and UPDATE.
A row lock does not prevent read access (e.g. a normal select without for update). Other process still could read and process this rows.

A possible solution is to first UPDATE the record(s) with a UUID, then read the records with the same UUID.

Each process will have a unique ID (UUID), and the table will have a new column named 'process_uuid' for that ID. Then the process will run something like:

* Generate a UUID with: <https://github.com/ramsey/uuid>

* Update the desired records with the generated UUID.

```sql
UPDATE table SET status='processing', process_uuid='69c6a30e-bc77-498f-954d-1640bc394f74' WHERE status='new';
```

* SELECT only the process specific rows by UUID

While the update runs the row is locked, so no other process can read it. 
After the update this row is owned by a specific process, 
then it can run a SELECT to fetch all necessary data from that row(s).

```sql
SELECT * FROM table WHERE status='processing' AND process_uuid='69c6a30e-bc77-498f-954d-1640bc394f74';
```

Finally, the last UPDATE will set the row to 'processed', 
and may (or may not) remove the owner field (process_uuid). 
Keeping it there will also enable some statistics about the process work.

```sql
UPDATE table SET status='processed' WHERE id=x;
```