---
title: Slim 4 - Zend Query Builder Setup
layout: post
comments: true
published: true
description: 
keywords: php slim sql querybuilder
---

This tutorial shows you how to work with the powerful and lightweight Slim 4 framework.

## Table of contents

* [Requirements](#requirements)
* [Introduction](#introduction)
* [Installation](#installation)
* [Usage](#usage)
* [Repository](#repository)

## Requirements

* PHP 7.1+
* MySQL 5.7+
* Composer
* [A Slim 4 application](https://odan.github.io/2019/11/05/slim4-tutorial.html)

## Introduction

This tutorial demonstrates how to install [Zend SQL Query Builder](https://docs.zendframework.com/zend-db/)
and integrate it into a Slim 4 application.

## Installation

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

As next we are installing [zendframework/zend-db](https://docs.zendframework.com/zend-db/) for 
building SQL queries.

To install the query builder, run:

```php
composer require zendframework/zend-db
```

To configure the database connection we have to add an `\Zend\Db\Adapter\AdapterInterface` 
container definition. 

Add the following container definitions, e.g. into `config/container.php`:

```php
<?php

use Psr\Container\ContainerInterface;
use Selective\Config\Configuration;
use Slim\App;
use Slim\Factory\AppFactory;
use Zend\Db\Adapter\Adapter;
use Zend\Db\Adapter\AdapterInterface;

return [

    // ...
    
    AdapterInterface::class => function (ContainerInterface $container) {
        return new Adapter($container->get(Configuration::class)->getArray('db'));
    },

    PDO::class => function (ContainerInterface $container) {
        $connection = $container->get(AdapterInterface::class)->getDriver()->getConnection();
        $connection->connect();

        return $connection->getResource();
    },
];
```

Add the `QueryFactory` class into: `src/Factory/QueryFactory.php` and copy / paste this content:

```php
<?php

namespace App\Factory;

use Zend\Db\Adapter\AdapterInterface;
use Zend\Db\ResultSet\ResultSet;
use Zend\Db\TableGateway\TableGateway;

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
        return new TableGateway($table, $this->adapter, null, new ResultSet(ResultSet::TYPE_ARRAY));
    }
}
```

## Usage

You can use the query builder to create `SELECT`, `UPDATE`, `INSERT` and `DELETE` statements.

Here are some examples.

Select all rows (for simple queries):

```php
$rows = $this->queryFactory->table('users')->select( /* where */ );

foreach ($rows as $row) {
    print_r($row);
}
```

Select all rows (for complex queries):

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

Select only the first row:

```php
$row = $this->queryFactory->table('users')
    ->select(['id' => 1])->current();
```

Insert a record:

```php
$table = $this->queryFactory->table('users');
$table->insert($values);

$newId = (int)$table->getLastInsertValue();
```

Update a record:

```php
$this->queryFactory->table('users')
    ->update(['email' => 'new@example.com'], ['id' => 1]);
```

Delete a record:

```php
$this->queryFactory->table('users')
    ->delete(['id' => 1]);
```

## Repository

Create a new directory: `src/Domain/User/Repository`

Create the file: `src/Domain/User/Repository/UserCreatorRepository.php` and insert this content:

```php
<?php

namespace App\Domain\User\Repository;

use App\Domain\User\Data\UserCreateData;
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

    /**
     * Insert user row.
     *
     * @param UserCreateData $user The user
     *
     * @return int The new ID
     */
    public function insertUser(UserCreateData $user): int
    {
        $values = [
            'username' => $user->username,
            'first_name' => $user->firstName,
            'last_name' => $user->lastName,
            'email' => $user->email,
        ];

        // Insert record
        $table = $this->queryFactory->table('users');
        $table->insert($values);

        // Return new primary key value (id)
        return (int)$table->getLastInsertValue();
    }
}
```

Note that we have declared `QueryFactory` as a dependency, because the repository requires a database connection.
