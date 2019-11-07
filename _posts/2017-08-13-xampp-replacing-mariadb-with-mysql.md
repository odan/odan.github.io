---
title: XAMPP - Replacing MariaDB with MySQL
layout: post
comments: true
published: true
description: 
keywords: 
---

The latest version of XAMPP contains [MariaDB](https://mariadb.org/) instead of [MySQL](https://www.mysql.com/).

But [MariaDB is not 100% compatible with MySQL](https://mariadb.com/kb/en/mariadb/mariadb-vs-mysql-compatibility/) 
and can be replaced with the "orginal" MySQL server.

## Requirements

* Windows
* [XAMPP](https://www.apachefriends.org) for Windows
* Administrator privileges to restart Windows services

## Backup

* Backup the old database into a SQL dump file
* Stop the MariaDB service
* Rename the folder: `c:\xampp\mysql` to `c:\xampp\mariadb`

## Installation

* Download MySQL Community Server: <https://dev.mysql.com/downloads/mysql/>
* Click: `Looking for the latest GA version?`
* Select Version: 5.7.25
* Select Operating System: Microsoft Windows
* Select OS Version: Windows (x86, 32-bit)
* Scroll down to `ZIP Archive` and click [Download](https://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-5.7.25-win32.zip).
* Create a new and empty folder: `c:\xampp\mysql`
* Extract `mysql-5.7.25-win32.zipp` to: `c:\xampp\mysql`
* Create a new file: `c:\xampp\mysql\bin\my.ini` and copy this content:

```ini
[mysqld]
# Set basedir to your installation path
basedir=c:/xampp/mysql

# Set datadir to the location of your data directory
datadir=c:/xampp/mysql/data

# Default: 128 MB
# New: 1024 MB
innodb_buffer_pool_size = 1024M
```

### Initializing the data directory

* Copy the old `data` directory from `c:\xampp\mariadb\data` to  `c:\xampp\mysql\data`

* Start the MySQL server. You can use the XAMPP Control Panel (MySQL > Start) to start the MySQL service.

* Repair all corrupted tables in the `c:\xampp\mysql\data` directory. Press ENTER if your password is empty.

```cmd
cd c:\xampp\mysql\bin
```

```cmd
mysqlcheck.exe -u root -p --auto-repair --all-databases
```

Update structure to latest version:

```cmd
mysql_upgrade.exe -u root -p --force
```

Check the tables for errors:
  
```cmd
mysqlcheck.exe -u root -p --check --all-databases
```

**Notice:** If you don't want to copy and migrate the old `data` directory, you can create a fresh directory 
with this command:

```cmd
c:\xampp\mysql\bin>mysqld.exe --initialize-insecure --basedir=c:\xampp\mysql --datadir=c:\xampp\mysql\data
```

Finished

## Known issues

### Questions

* I can't start or stop mysql using the XAMPP control panel button
* The XAMPP control panel is crashing while shutting down

### Answers

* Make sure you have installed the 32-bit version of MySQL. 
The MySQL 64-bit version is not compatible with the XAMPP Control Panel (32-bit).
* This setup is not tested with MySQL 8.x.
* MySQL 8.x is only available as 64-bit version
* Try to fix the directory permissions with this [batch script](https://odan.github.io/2017/01/13/reset-windows-folder-permissions.html)
