---
title: MySQL backup and restore
layout: post
comments: true
published: false
description: 
keywords: 
---


Backup all databases:

```bat
@echo off

mysqldump --user=root --password= --all-databases --comments --quote-names --allow-keywords --complete-insert --skip-extended-insert --create-options --add-drop-table --hex-blob --default-character-set=utf8 -r C:\mysqlbackup.sql

echo finished
```

Backup with bulk insert for more performance: `--extended-insert=true`:

```bat
@echo off

mysqldump dbname --user=root --password=  --comments --quote-names --allow-keywords --extended-insert=true --create-options --add-drop-table --hex-blob --default-character-set=utf8 -r C:\mysqlbackup.sql

echo finished
```

Restore from a sql dump file (for small sql files):

```bat
@echo off

mysql -h localhost -u root -p secret dbname --default-character-set=utf8 < mysqlbackup.sql

echo finished
```

Restore from a sql dump file (for big sql files (> 1 GB)):

```bat
@echo off

mysql -h localhost -u root -p secret dbname --default-character-set=utf8 --init-command="SET GLOBAL max_allowed_packet=1073741824;SET NAMES utf8;SET FOREIGN_KEY_CHECKS=0;SET UNIQUE_CHECKS=0;" < mysqlbackup.sql

echo finished
```

