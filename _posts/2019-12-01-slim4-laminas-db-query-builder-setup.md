---
title: Slim 4 - Laminas Query Builder
layout: post
comments: true
published: true
description: 
keywords: php slim sql laminas zend querybuilder
---

## Table of contents

* [Requirements](#requirements)
* [Introduction](#introduction)
* [Installation](#installation)
* [Configuration](#configuration)
* [Repository](#repository)
* [Usage](#usage)
  * [Select](#select)  
  * [Insert](#insert)
  * [Update](#update)
  * [Delete](#delete)
* [Handling relationships](#handling-relationships)
* [Transactions](#transactions)
* [Usage](#usage)

## Requirements

* PHP 7.1+
* MySQL 5.7+
* Composer
* [A Slim 4 application](https://odan.github.io/2019/11/05/slim4-tutorial.html)

## Introduction

You can use the [Laminas SQL Query Builder](https://docs.laminas.dev/laminas-db/)
to connect your Slim 4 application to a database.

## Installation

To install the [laminas/laminas-db](https://docs.laminas.dev/laminas-db/) package, run:

```php
composer require laminas/laminas-db
```

## Configuration

Add the database settings into your configuration file, e.g `config/settings.php`:

```php
// Database settings
$settings['db'] = [
    'driver' => 'Pdo_Mysql',
    'hostname' => 'localhost',
    'username' => 'root',
    'database' => 'test',
    'password' => '',
    'charset' => 'utf8mb4',
    'collation' => 'utf8mb4_unicode_ci',
    'driver_options' => [
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

Add the following container definitions, e.g. in `config/container.php`:

```php
<?php

use Psr\Container\ContainerInterface;
use Slim\App;
use Slim\Factory\AppFactory;
use Laminas\Db\Adapter\Adapter;
use Laminas\Db\Adapter\AdapterInterface;

return [

    // ...
    
    AdapterInterface::class => function (ContainerInterface $container) {
        return new Adapter($container->get('settings')['db']);
    },

    PDO::class => function (ContainerInterface $container) {
        $connection = $container->get(AdapterInterface::class)
            ->getDriver()
            ->getConnection();

        $connection->connect();

        return $connection->getResource();
    },
];
```

Add the `QueryFactory` class into: `src/Factory/QueryFactory.php` and copy / paste this content:

```php
<?php

namespace App\Factory;

use Laminas\Db\Adapter\AdapterInterface;
use Laminas\Db\ResultSet\ResultSet;
use Laminas\Db\TableGateway\TableGateway;

final class QueryFactory
{
    /**
     * @var AdapterInterface The database connection
     */
    private $adapter;

    public function __construct(AdapterInterface $adapter)
    {
        $this->adapter = $adapter;
    }

    public function table(string $table): TableGateway
    {
        return new TableGateway(
            $table, 
            $this->adapter,
            null, 
            new ResultSet(ResultSet::TYPE_ARRAY)
        );
    }
}
```

## Repository

You can inject the query factory instance into your repository like this:

```php
<?php

namespace App\Domain\User\Repository;

use App\Factory\QueryFactory;

/**
 * Repository.
 */
class UserCreatorRepository
{
    /**
     * @var QueryFactory The query builder factory
     */
    private $queryFactory;

    /**
     * Constructor.
     *
     * @param QueryFactory $queryFactory The query builder factory
     */
    public function __construct(QueryFactory $queryFactory)
    {
        $this->queryFactory = $queryFactory;
    }

    // ...

}
```

## Usage

### Select 

Fetch all rows:

```php
$rows = $this->queryFactory->table('users')->select( /* where */ );

foreach ($rows as $row) {
    print_r($row);
}
```

A complex query:

```php
$table = $this->queryFactory->table('users');

$select = $table->getSql()->select();
$select->columns(['id']);
$select->where(['id' => 1]);

$rows = $table->selectWith($select);

foreach ($rows as $row) {
    print_r($row);
}
```

Note that the result of a query is an instance of 
[ResultSet](https://docs.laminas.dev/laminas-db/result-set/#laminasdbresultsetresultset-and-laminasdbresultsetabstractresultset)
that will expose each row as either an ArrayObject-like object or an array of row data.
 
Select only the first row:

```php
$row = $this->queryFactory->table('users')
    ->select(['id' => 1])->current();
```

Read more: [Select](https://docs.laminas.dev/laminas-db/sql/#select)

### Insert 

Insert a record:

```php
$table = $this->queryFactory->table('users');
$table->insert($values);

$newId = (int)$table->getLastInsertValue();
```

Read more: [Insert](https://docs.laminas.dev/laminas-db/sql/#insert)

### Update
 
Update a record

```php
$values = ['email' => 'new@example.com'];

$this->queryFactory->table('users')->update($values, ['id' => 1]);
```

Read more: [Update](https://docs.laminas.dev/laminas-db/sql/#update)

### Delete

Delete a record:

```php
$this->queryFactory->table('users')->delete(['id' => 1]);
```

Read more: [Delete](https://docs.laminas.dev/laminas-db/sql/#delete)

## Handling relationships

You can define relationships directly with a [join clause](https://docs.laminas.dev/laminas-db/sql/#join).

```php
use Laminas\Db\Sql\Join;
// ...

$table = $this->queryFactory->table('users');

$select = $table->getSql()->select();

$select->columns(['id']);
$select->where(['id' => 1]);

$select->join('contacts', 'users.id = contacts.user_id');
$select->join('orders', 'users.id = orders.user_id', Join::JOIN_LEFT);

$rows = $table->selectWith($select);
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
