---
title: Slim 4 - Doctrine DBAL
layout: post
comments: true
published: true
description: 
keywords: php slim doctrine sql query builder
---

## Table of contents

* [Requirements](#requirements)
* [Introduction](#introduction)
* [Installation](#installation)
* [Repository](#repository)
* [Usage](#usage)
  * [Select](#select)  
  * [Insert](#insert)
  * [Update](#update)
  * [Delete](#delete)
* [Handling relationships](#handling-relationships)
* [Transactions](#transactions)
* [Read more](#read-more)

## Requirements

* PHP 7.1+
* MySQL 5.7+
* Composer
* [A Slim 4 application](https://odan.github.io/2019/11/05/slim4-tutorial.html)

## Introduction

You can use a query builder such as [Doctrine DBAL](https://www.doctrine-project.org/projects/dbal.html) to connect 
your Slim 4 application to a database.

## Installation

To add Doctrine DBAL to your application, run:

```
composer require doctrine/dbal
```

## Configuration

Add the database settings to Slim’s settings array, e.g `config/settings.php`:

```php
// Database settings
$settings['db'] = [
    'driver' => 'pdo_mysql',
    'host' => 'localhost',
    'dbname' => 'test',
    'user' => 'root',
    'password' => '',
    'charset' => 'utf8mb4',
    'collation' => 'utf8mb4_unicode_ci',
    'driverOptions' => [
        // Turn off persistent connections
        PDO::ATTR_PERSISTENT => false,
        // Enable exceptions
        PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
        // Emulate prepared statements
        PDO::ATTR_EMULATE_PREPARES => true,
        // Set default fetch mode to array
        PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
        // Set character set
        PDO::MYSQL_ATTR_INIT_COMMAND => 'SET NAMES utf8mb4 COLLATE utf8mb4_unicode_ci'
    ],
];
```

In your `config/container.php` or wherever you add your container definitions:

```php
<?php

use Doctrine\DBAL\Configuration as DoctrineConfiguration;
use Doctrine\DBAL\Connection;
use Doctrine\DBAL\DriverManager;
use Psr\Container\ContainerInterface;
use Slim\App;
use Slim\Factory\AppFactory;

return [

    // ...
    
    // Database connection
    Connection::class => function (ContainerInterface $container) {
        $config = new DoctrineConfiguration();
        $connectionParams = $container->get('settings')['db'];

        return DriverManager::getConnection($connectionParams, $config);
    },

    PDO::class => function (ContainerInterface $container) {
        return $container->get(Connection::class)->getWrappedConnection();
    },
];
```

## Repository

You can inject the connection instance into your repository like this:

```php
<?php

namespace App\Domain\User\Repository;

use Doctrine\DBAL\Connection;

class UserRepository
{
    /**
     * @var Connection The database connection
     */
    private $connection;

    /**
     * The constructor.
     *
     * @param Connection $connection The database connection
     */
    public function __construct(Connection $connection)
    {
        $this->connection = $connection;
    }

    // ...
}
```

## Usage

Once the connection instance has been injected, you may use it like so:

### Select
 
Query all rows:

```php
$query = $this->connection->createQueryBuilder();

$rows = $query
    ->select('id', 'username')
    ->from('users')
    ->execute()
    ->fetchAll();
```

Query the table with where:

```php
$userInputEmail = 'mail@example.com';

$query = $this->connection->createQueryBuilder();

$rows = $query
    ->select('id', 'username')
    ->from('users')
    ->where('email = :email')
    ->setParameter('email', $userInputEmail)
    ->execute()
    ->fetchAll();
```

Query the table by id:

```php
$query = $this->connection->createQueryBuilder();

$row = $query->select('id', 'username')
    ->from('users')
    ->where('id = :id')
    ->setParameter('id', 1)
    ->execute()
    ->fetch();
```

### Insert

Insert a record:

```php
$values = [
    'first_name' => 'john',
    'last_name' => 'doe',
    'email' => 'john.doe@example.com',
];

$this->connection->insert('users', $values);
```

Retrieve the last inserted id:

```php
$newId = $this->connection->lastInsertId();
```

### Update

Update a record:

```php
$values = ['email' => 'new@example.com'];

$this->connection->update('users', $values, ['id' => 1]);
```

### Delete

Delete a record

```php
$this->connection->delete('users', ['id' => 1]);
```

## Handling relationships

You can define relationships directly with a 
[join clause](https://www.doctrine-project.org/projects/doctrine-dbal/en/latest/reference/query-builder.html#join-clauses).

```php
$query = $this->connection->createQueryBuilder();

$rows = $query
    ->select('id', 'username')
    ->from('users')
    ->innerJoin('users', 'contacts', 'contacts', 'users.id = contacts.user_id')
    ->leftJoin('users', 'orders', 'orders', 'users.id = orders.user_id')
    ->execute()
    ->fetchAll();
```

## Transactions

You should orchestrate all transactions in a service class.
Please don't use the transaction handler directly within a repository.

The transaction handling can be abstracted away with this interface:

```php
<?php

namespace App\Database;

interface TransactionInterface
{
    public function begin(): void;
    public function commit(): void;
    public function rollback(): void;
}
```

## Read more

* [Doctrine DBAL website](https://www.doctrine-project.org/projects/dbal.html)
* [Doctrine DBAL documentation ](https://www.doctrine-project.org/projects/doctrine-dbal/en/2.10/reference/query-builder.html#sql-query-builder)
* [Doctrine DBAL on Github](https://github.com/doctrine/dbal)
* [Simplifying database interactions with Doctrine DBAL](https://www.thedevfiles.com/2014/08/simplifying-database-interactions-with-doctrine-dbal)
