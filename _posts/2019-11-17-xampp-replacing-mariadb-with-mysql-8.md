---
title: XAMPP - Replacing MariaDB with MySQL 8
layout: post
comments: true
published: true
description: 
keywords: mysql
---

The latest version of [XAMPP](https://www.apachefriends.org/) contains [MariaDB](https://mariadb.org/) instead of [MySQL](https://www.mysql.com/).

But [MariaDB is not 100% compatible with MySQL](https://mariadb.com/kb/en/mariadb/mariadb-vs-mysql-compatibility/) 
and can be replaced with the "original" MySQL server.

## Requirements

* Windows
* The latest [XAMPP](https://www.apachefriends.org) for Windows v.7.1+ (**64-Bit**)
* [The latest Visual C++ Redistributable Packages](https://support.microsoft.com/en-us/help/2977003/the-latest-supported-visual-c-downloads), 
  * [How to install the Visual C++ Redistributable](https://www.groovypost.com/howto/fix-visual-c-plus-plus-redistributable-windows-10/)
* Administrator privileges to restart Windows services

## Backup

* Backup the old databases into a SQL dump file (without the system databases)
* Stop the MariaDB service
* Rename the folder: `c:\xampp\mysql` to `c:\xampp\mariadb`

## Installation

* Download MySQL Community Server 8.x (**64-Bit**) from: <https://dev.mysql.com/downloads/mysql/>
* Scroll down to `Other Downloads:` and click **Download**.
* Click the link: **No thanks, just start my download**
* Create a new and empty folder: `c:\xampp\mysql`
* Extract ZIP archive to: `c:\xampp\mysql`
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

# Default since MySQL 8: caching_sha2_password
default_authentication_plugin=mysql_native_password

[client]
ssl-mode=DISABLED
```

### Initializing the data directory

**<span style="color:red">Important: Don't skip this step!</span>**

Note: You need to open the console (cmd) as Administrator.

Change into the `mysql\bin` directory.

```cmd
cd c:\xampp\mysql\bin
```

Create a "data" directory.

```cmd
mkdir c:\xampp\mysql\bin\data
```

Initialize the data directory.

```cmd
mysqld.exe --default-authentication-plugin=mysql_native_password --initialize-insecure --basedir=c:\xampp\mysql --datadir=c:\xampp\mysql\data
```

Start the MySQL service in your XAMPP Control Panel or run `C:\xampp\mysql\bin\mysqld.exe`.

### Data restore

Import the SQL dump file into the new database.

Done.

## Known issues

* Error: `Authentication plugin 'caching_sha2_password' cannot be loaded`
  * MySQL 8 uses `caching_sha2_password` as default authentication plugin.
  * Please follow the instructions (from above) and it should not happen.
  * Add the command line option: [--default-auth=mysql_native_password](https://dev.mysql.com/doc/refman/8.0/en/mysqldump.html#option_mysqldump_default-auth)
  * Read more: [Connecting MySQL - 8.0 with MySQL Workbench](https://stackoverflow.com/questions/49194719/authentication-plugin-caching-sha2-password-cannot-be-loaded)

* Error: `mysqldump: Got error: 2026: SSL connection error: error:00000000:lib(0):func(0):reason(0) when trying to connect`
  * Add the `[client] ssl-mode=DISABLED` option to your `my.ini` file

* XAMPP control panel error: `MySQL shutdown unexpectedly.`
  * Make sure you have installed the **64-bit** version of XAMPP and MySQL.
  * Make sure that no other MySQL server is already running on the same port (3306).
  * Try to run `xampp-control.exe` as administrator
  * You might forgot to [initialize the data directory](#initializing-the-data-directory). 
  * Some MySQL versions are not compatible with the XAMPP control panel.
  * Try to start `mysqld.exe` from the console: `c:\xampp\mysql\bin>mysqld.exe`
  * Try to [delete the ibdata1 file](https://stackoverflow.com/a/18162264/1461181)
  
* I can't start or stop MySQL using the XAMPP control panel button.
  * Make sure you have installed the **64-bit** version of XAMPP and MySQL.
