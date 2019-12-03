---
title: Slim 4 - CakePHP Query Builder Setup
layout: post
comments: true
published: true
description: 
keywords: php slim cakephp sql querybuilder
---

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

This tutorial demonstrates how to install [CakePHP Query Builder](https://book.cakephp.org/3/en/orm/query-builder.html)
and integrate it into a Slim 4 application.

## Installation

Add the database settings into your configuration file, e.g `config/settings.php`:

```php
// Database settings
$settings['db'] = [
    'driver' => \Cake\Database\Driver\Mysql::class,
    'host' => 'localhost',
    'encoding' => 'utf8mb4',
    'collation' => 'utf8mb4_unicode_ci',
    'flags' => [
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

As next we are installing [cakephp/database](https://github.com/cakephp/database) for 
building SQL queries.

To install the query builder, run:

```php
composer require cakephp/database
```

To configure the database connection we have to add a `\Cake\Database\Connection::class` 
container definition, e.g. into `config/container.php`:

```php
<?php

use Cake\Database\Connection;
use Psr\Container\ContainerInterface;
use Selective\Config\Configuration;
use Slim\App;
use Slim\Factory\AppFactory;

return [

    // ...
    
    // Database connection
    Connection::class => function (ContainerInterface $container) {
        return new Connection($container->get(Configuration::class)->getArray('db'));
    },

    PDO::class => function (ContainerInterface $container) {
        $db = $container->get(Connection::class);
        $db->getDriver()->connect();

        return $db->getDriver()->getConnection();
    },
];
```

Add the `QueryFactory` class file here: `src/Repository/QueryFactory.php` and copy / paste this content:

```php
<?php

namespace App\Repository;

use Cake\Database\Connection;
use Cake\Database\Query;
use RuntimeException;
use UnexpectedValueException;

/**
 * Factory.
 */
final class QueryFactory
{
    /**
     * @var Connection
     */
    private $connection;

    /**
     * @var callable
     */
    private $beforeUpdateCallback;

    /**
     * @var callable
     */
    private $beforeInsertCallback;

    /**
     * Constructor.
     *
     * @param Connection $connection The database connection
     */
    public function __construct(Connection $connection)
    {
        $this->connection = $connection;
    }

    /**
     * Create a new query.
     *
     * @return Query The query
     */
    public function newQuery(): Query
    {
        return $this->connection->newQuery();
    }

    /**
     * Create a new 'select' query for the given table.
     *
     * @param string $table The table name
     *
     * @throws RuntimeException
     *
     * @return Query A new select query
     */
    public function newSelect(string $table): Query
    {
        $query = $this->newQuery()->from($table);

        if (!$query instanceof Query) {
            throw new UnexpectedValueException('Failed to create query');
        }

        return $query;
    }

    /**
     * Create an 'update' statement for the given table.
     *
     * @param string $table The table to update rows from
     * @param array $data The values to be updated
     *
     * @return Query The new update query
     */
    public function newUpdate(string $table, array $data): Query
    {
        if (isset($this->beforeUpdateCallback)) {
            $data = (array)call_user_func($this->beforeUpdateCallback, $data, $table);
        }

        return $this->newQuery()->update($table)->set($data);
    }

    /**
     * Create an 'update' statement for the given table.
     *
     * @param string $table The table to update rows from
     * @param array $data The values to be updated
     *
     * @return Query The new insert query
     */
    public function newInsert(string $table, array $data): Query
    {
        if (isset($this->beforeInsertCallback)) {
            $data = (array)call_user_func($this->beforeInsertCallback, $data, $table);
        }

        $columns = array_keys($data);

        return $this->newQuery()->insert($columns)
            ->into($table)
            ->values($data);
    }

    /**
     * Create a 'delete' query for the given table.
     *
     * @param string $table The table to delete from
     *
     * @return Query A new delete query
     */
    public function newDelete(string $table): Query
    {
        return $this->newQuery()->delete($table);
    }

    /**
     * Before update event.
     *
     * @param callable $callback The callback (string $row, string $table)
     *
     * @return void
     */
    public function beforeUpdate(callable $callback): void
    {
        $this->beforeUpdateCallback = $callback;
    }

    /**
     * Before insert event.
     *
     * @param callable $callback The callback (string $row, string $table)
     *
     * @return void
     */
    public function beforeInsert(callable $callback): void
    {
        $this->beforeInsertCallback = $callback;
    }
}

```

## Usage

You can use the query builder to create `SELECT`, `UPDATE`, `INSERT` and `DELETE` statements.

Here are some examples.

Select all rows:

```php
$query = $this->queryFactory->newSelect('users')->select('*');
$rows = $query->execute()->fetchAll('assoc');

foreach ($rows as $row) {
    print_r($row);
}
```

Select all rows (for complex queries):

```php
$query = $this->queryFactory->newSelect('users');

$query->select(['id', 'username']);
$select->andWhere(['id' => 1]);

$rows = $query->execute()->fetchAll('assoc');

foreach ($rows as $row) {
    print_r($row);
}
```

Select only the first row:

```php
$query = $this->queryFactory->newSelect('users')->andWhere(['id' => 1]);

$row = $query->execute()->fetch('assoc');
```

Insert a record:

```php
$row = [
    'first_name' => 'john',
    'last_name' => 'doe',
    'email' => 'john.doe@example.com',
];

$newId = (int)$this->queryFactory->newInsert('users', $row)->execute()->lastInsertId();
```

Update a record:

```php
$data = ['email' => 'new@example.com'];

$this->queryFactory->newUpdate('users', $data)->andWhere(['id' => 1])->execute();
```

Delete a record:

```php
$this->queryFactory->newUpdate('users')->andWhere(['id' => 1])->execute();
```

## Repository

Create a new directory: `src/Domain/User/Repository`

Create the file: `src/Domain/User/Repository/UserCreatorRepository.php` and insert this content:

```php
<?php

namespace App\Domain\User\Repository;

use App\Domain\User\Data\UserData;
use App\Repository\QueryFactory;
use App\Repository\RepositoryInterface;
use App\Repository\TableName;
use Cake\Database\StatementInterface;

/**
 * Repository.
 */
final class UserRepository implements RepositoryInterface
{
    /**
     * @var QueryFactory The query factory
     */
    private $queryFactory;

    /**
     * Constructor.
     *
     * @param QueryFactory $queryFactory The query factory
     */
    public function __construct(QueryFactory $queryFactory)
    {
        $this->queryFactory = $queryFactory;
    }

    /**
     * Find all users.
     *
     * @return UserCreatorData[] A list of users
     */
    public function findAllUsers(): array
    {
        $query = $this->queryFactory->newSelect(TableName::USERS)->select('*');

        $rows = $query->execute()->fetchAll(StatementInterface::FETCH_TYPE_ASSOC);

        $result = [];
        foreach ($rows as $row) {
            $result[] = new UserData($row);
        }

        return $result;
    }
}
```

Note that we have declared `QueryFactory` as a dependency, because the repository requires a database connection.
