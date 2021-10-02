---
title: Slim 4 - Cycle Query Builder
layout: post
comments: true
published: true
description:
keywords: php, slim, cycle, sql, query-builder, slim-framework
image: https://odan.github.io/assets/images/slim-logo-600x330.png
---

## Table of contents

* [Requirements](#requirements)
* [Introduction](#introduction)
* [Installation](#installation)
* [Configuration](#configuration)
* [Accessing Query Builder](#accessing-query-builder)
* [Usage](#usage)
    * [Select](#select)
    * [Insert](#insert)
    * [Update](#update)
    * [Delete](#delete)
* [Handling relationships](#handling-relationships)
* [Transactions](#transactions)
* [Read more](#read-more)

## Requirements

* PHP 7.2+
* MySQL 5.7+
* Composer (for development)
* [A Slim 4 application](https://odan.github.io/2019/11/05/slim4-tutorial.html)

## Introduction

[Spiral Database](https://cycle-orm.dev/docs/query-builder-basic) provides 
query builders for Insert, Update, Delete and Select operations, 
and multiple shortcuts for common functionality.

For this reason, I want to take a closer look at this exciting library 
and show how you can integrate it into a Slim Framework project.

## Installation

To add the [cycle/database](https://github.com/cycle/database) package to your application, run:

```php
composer require cycle/database
```

## Configuration

In order to operate, the component requires a proper database connection to be set. 
All database connections are managed using the `DatabaseManager` service provided. 
You can configure your first database connection 
to be initiated on demand using the following configuration 
in your configuration file:

```php
// Database settings
$settings['db'] = [
    'default' => 'default',
    'databases' => [
        'default' => ['connection' => 'default']
    ],
    'connections' => [
        'default' => [
            'driver' => \Cycle\Database\Driver\MySQL\MySQLDriver::class,
            'connection' => 'mysql:host=127.0.0.1;dbname=test',
            'username' => '',
            'password' => '',
        ]
    ]
];
```

Next, add a DI container definitions for the `DatabaseManager` and `DatabaseInterface`:

```php
<?php

use Cycle\Database\Config\DatabaseConfig;
use Cycle\Database\DatabaseInterface;
use Cycle\Database\DatabaseManager;
use Psr\Container\ContainerInterface;
// ...

return [

    // ...
    
    DatabaseManager::class => function (ContainerInterface $container) {
        $dbConfig = $container->get('settings')['db'];

        return new DatabaseManager(new DatabaseConfig($dbConfig));
    },

    DatabaseInterface::class => function (ContainerInterface $container) {
        return $container->get(DatabaseManager::class)->database('default');
    },
];
```

The next step is optional. To fetch the `PDO` instance from the 
default connection you could add this container definition:

```php
<?php

use Cycle\Database\DatabaseManager;
use Psr\Container\ContainerInterface;
// ...

return [
    // ...
    PDO::class => function (ContainerInterface $container) {
        $driver = $container->get(DatabaseManager::class)->driver('default');
        $class = new ReflectionClass($driver);
        $method = $class->getMethod('getPDO');
        $method->setAccessible(true);

        return $method->invoke($driver);
    },
];
```

*Please note: I had to use [Reflection](https://www.php.net/manual/en/intro.reflection.php) 
to get access to the PDO object. If you know a better way, please let me know.*

## Accessing Query Builder

You can get access to the query builder by declaring the
`DatabaseInterface` in the class constructor. 

```php
<?php

namespace App\Domain\User\Repository;

use Cycle\Database\DatabaseInterface;

final class UserRepository
{
    private DatabaseInterface $database;

    public function __construct(DatabaseInterface $database)
    {
        $this->database = $database;
    }

    // ...
}
```

## Usage

### Select

Query all rows:

```php
$select = $this->database->select()->from('users');
$select->columns(['id', 'username']);

$rows = $select->fetchAll();

foreach ($rows as $row) {
    print_r($row);
}
```

Simple conditions

```php
$select->where('status', 'active');
```

Select a single row by id:

```php
$select = $this->database->select()
    ->from('users');
    ->columns(['id', 'username']);
    ->where('id', 1);

$row = $select->run()->fetch();
```

Read more: [SelectQuery Builder](https://cycle-orm.dev/docs/query-builder-extended#selectquery-builder)

### Insert

Insert a record

```php
$insert = $this->database->insert('users');

$values = [
    'first_name' => 'john',
    'last_name' => 'doe',
    'email' => 'john.doe@example.com',
];

$insert->values($values)->run();
```

Insert a record and get the last inserted id:

```php
$newId = (int)$insert->values($values)->run();
```

Read more: [Insert Builder](https://cycle-orm.dev/docs/query-builder-extended#insert-builder)

### Update

Update a record:

```php
$values = ['email' => 'new@example.com'];

$update = $this->database->table('users')->update($values);
$update->where('id', '=', 1);
$update->run();
```

Read more: [UpdateQuery Builder](https://cycle-orm.dev/docs/query-builder-extended#updatequery-builder)

### Delete

Delete a record:

```php
$this->database->table('users')->delete()
    ->where('id', '=', 1)
    ->run();
```

Read more: [DeleteQuery Builders](https://cycle-orm.dev/docs/query-builder-extended#deletequery-builders)

## Handling relationships

You can join any desired table to your query 
using `leftJoin`, `join`, `rightJoin`, `fullJoin` and `innerJoin` methods:

```php
$select = $this->database->select()->from('posts');

$select->columns(
    [
        'posts.id',
        'posts.title',
        'users.id as user_id'
    ]
);

$select->innerJoin('users')->on('users.id', 'posts.user_id');

$rows = $select->fetchAll();
```

Read more: [Joins](https://cycle-orm.dev/docs/query-builder-extended#selectquery-builder-joins)

## Transactions

Transactions should be handles in a server, but the implementation should 
happen in the infrastructure layer.

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

The `TransactionInterface` could be implemented as follows:

```php
<?php

namespace App\Database;

use Cycle\Database\DatabaseInterface;

final class Transaction implements TransactionInterface
{
    private $database;

    public function __construct(DatabaseInterface $database)
    {
        $this->database = $database;
    }

    public function begin(): void
    {
        $this->database->begin();
    }

    public function commit(): void
    {
        $this->database->commit();
    }

    public function rollback(): void
    {
        $this->database->rollback();
    }
}

```

Add a container definition for `App\Database\TransactionInterface::class`:

```php
<?php

use App\Database\Transaction;
use App\Database\TransactionInterface;
use Psr\Container\ContainerInterface;

return [
    TransactionInterface::class => function (ContainerInterface $container) {
        return $container->get(Transaction::class);
    },
];
```

#### Transaction handling example

You should orchestrate all transactions in a service class.
Please don't use the transaction handler directly within a repository.

**Example**

```php
<?php

namespace App\Domain\User\Service;

use App\Domain\User\Repository\UserCreatorRepository;
use App\Factory\LoggerFactory;
use App\Database\TransactionInterface;
use Exception;

final class UserCreator
{
    private UserCreatorRepository $repository;
    private TransactionInterface $transaction;

    public function __construct(
        UserCreatorRepository $repository,
        TransactionInterface $transaction
    ) {
        $this->repository = $repository;
        $this->transaction = $transaction;
    }

    public function createUser(array $formData): void
    {
        // Input validation
        // ...

        // Start database transaction
        $this->transaction->begin();

        try {
            // Execute multiple commands
            $userId = $this->repository->insertUser($userData);
            
            // Do more operations
            // ...    
    
            // Commit the active database transaction
            $this->transaction->commit();
        } catch (Exception $exception) {
            // Rollback the active database transaction
            $this->transaction->rollback();
        }
    }
}
```

## Read more

* <https://cycle-orm.dev/>
* <https://cycle-orm.dev/docs/query-builder-basic>
* <https://github.com/cycle/database>