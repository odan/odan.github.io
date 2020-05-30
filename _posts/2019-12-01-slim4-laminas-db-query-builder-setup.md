---
title: Slim 4 - Laminas Query Builder Setup
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

In your `config/container.php` or wherever you add your container definitions:

```php
<?php

use Psr\Container\ContainerInterface;
use Selective\Config\Configuration;
use Slim\App;
use Slim\Factory\AppFactory;
use Laminas\Db\Adapter\Adapter;
use Laminas\Db\Adapter\AdapterInterface;

return [

    // ...
    
    AdapterInterface::class => function (ContainerInterface $container) {
        return new Adapter($container->get(Configuration::class)->getArray('db'));
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

Once the query factory instance has been injected, you may use it like so:

### Query all rows

A simple query:

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

### Query the table with where

Select only the first row:

```php
$row = $this->queryFactory->table('users')
    ->select(['id' => 1])->current();
```

Read more: [Select](https://docs.laminas.dev/laminas-db/sql/#select)

### Insert a record

```php
$table = $this->queryFactory->table('users');
$table->insert($values);

$newId = (int)$table->getLastInsertValue();
```

Read more: [Insert](https://docs.laminas.dev/laminas-db/sql/#insert)

### Update a record

```php
$values = ['email' => 'new@example.com'];

$this->queryFactory->table('users')->update($values, ['id' => 1]);
```

Read more: [Update](https://docs.laminas.dev/laminas-db/sql/#update)

### Delete a record

```php
$this->queryFactory->table('users')->delete(['id' => 1]);
```

Read more: [Delete](https://docs.laminas.dev/laminas-db/sql/#delete)
