---
title: Slim 4 - Database testing
layout: post
comments: true
published: false
description: 
keywords: php, test, testing, database, phpunit, dbunit
---

## Table of contents

* [Requirements](#requirements)
* [Introduction](#introduction)
* [Configuration](#configuration)

## Requirements

* PHP 7.2+
* [A Slim 4 application](https://odan.github.io/2019/11/05/slim4-tutorial.html)

## Introduction

I would like to share some tips I have accumulated working with Slim writing 
functional tests.

**Testing with the persistence layer**

Probably the first thing you want to achieve while running your functional tests, 
is to separate your testing database from your development database. 
This is done to create a clean state for your test suite to run. 
It will allow you to control and create a desired state of the app for your test cases. 
Also, having a test suite write random data to your development copy of the database is not ideal.

This generally is achieved by making your Slim app connect to a different 
database while running the test suite. 

Using Symfony 4 or 5, you can define 
environment variables that are going to be used while testing in a ‘.env.test’ file. 


## Configuration

Edit the `config\settings.php` file and add a `db_testing` array:

```php
$settings['db_testing'] = array_merge($settings['db'], [
    'username' => 'root',
    'database' => 'test',
    'password' => '',
]);
```

You should also configure PHPUnit to change the environment variable APP_ENV to test.

Open `phpunit.xml` and add this configuration to define the `APP_ENV` constant as `test`:

```xml
<php>
    <const name="APP_ENV" value="test"/>
</php>
```

## Dumping the schema

The following script dumps the local development database into a set 
of SQL statements that can be executed to reproduce the original database object 
definitions and table data. 

Create the following directories in your project root: `bin/` and `resources/migrations/`.

Create a new file in `bin/dump.php` and copy-paste this content:

```php
<?php

use Psr\Container\ContainerInterface;

$filename = __DIR__ . '/../resources/migrations/schema.sql';

/** @var ContainerInterface $container */
$container = (require __DIR__ . '/../config/bootstrap.php')->getContainer();

$pdo = $container->get(PDO::class);

echo sprintf("Dump database: %s\n", (string)$pdo->query('select database()')->fetchColumn());

$statement = $pdo->query('SELECT TABLE_NAME FROM information_schema.tables WHERE table_schema = database()');

$sql = [];

while ($row = $statement->fetch(PDO::FETCH_ASSOC)) {
    $statement2 = $pdo->query(sprintf('SHOW CREATE TABLE `%s`;', (string)$row['TABLE_NAME']));
    $createTableSql = $statement2->fetch()['Create Table'];
    $sql[] = preg_replace('/AUTO_INCREMENT=\d+/', '', $createTableSql) . ';';
}

$sql = implode("\n\n", $sql);

file_put_contents($filename, $sql);

echo sprintf("Schema dumped to: %s\n", realpath($filename));

```

The dump the current table structure, run:

```
php bin/dump.php
```

Now open the generated file: `resources/migrations/schema.sql` and check the result.


## Resetting The Database After Each Test

* <https://phpunit.de/manual/6.5/en/database.html>
* <https://blog.cemunalan.com.tr/2020/02/02/10-symfony-testing-tips/>
* <https://beberlei.de/2015/02/14/phpunit_before_annotations_and_traits_for_code_reuse.html>
* <https://github.com/sebastianbergmann/dbunit>