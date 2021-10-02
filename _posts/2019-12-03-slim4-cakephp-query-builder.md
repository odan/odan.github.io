---
title: Slim 4 - CakePHP Query Builder
layout: post
comments: true
published: true
description: 
keywords: php, slim, cakephp, sql, query-builder, slim-framework
image: https://odan.github.io/assets/images/slim-logo-600x330.png
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

## Requirements

* PHP 7.4+
* MySQL 5.7+
* Composer (for development)
* [A Slim 4 application](https://odan.github.io/2019/11/05/slim4-tutorial.html)

## Introduction

The [CakePHP query builder](https://book.cakephp.org/4/en/orm/query-builder.html) 
provides a simple-to-use fluent interface 
for creating and running database queries. By composing queries together, 
you can create advanced queries using unions and subqueries with ease.

Underneath the covers, the query builder uses PDO prepared statements 
which protect against SQL injection attacks.

## Installation

To add the [cakephp/database](https://github.com/cakephp/database) package to your application, run:

```php
composer require cakephp/database
```

## Configuration

Add the database settings into your configuration file, e.g `config/settings.php`:

```php
// Database settings
$settings['db'] = [
    'driver' => \Cake\Database\Driver\Mysql::class,
    'host' => 'localhost',
    'database' => 'database',
    'username' => 'root',
    'password' => '',
    'encoding' => 'utf8mb4',
    'collation' => 'utf8mb4_unicode_ci',
    // Enable identifier quoting
    'quoteIdentifiers' => true,
    // Set to null to use MySQL servers timezone
    'timezone' => null,
    // PDO options
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

In your `config/container.php` or wherever you add your container definitions:

```php
<?php

use Cake\Database\Connection;
use Psr\Container\ContainerInterface;
use Slim\App;
use Slim\Factory\AppFactory;

return [

    // ...
    
    // Database connection
    Connection::class => function (ContainerInterface $container) {
        return new Connection($container->get('settings')['db']);
    },

    PDO::class => function (ContainerInterface $container) {
        $db = $container->get(Connection::class);
        $driver = $db->getDriver();
        $driver->connect();

        return $driver->getConnection();
    },
];
```

Create a new PHP file: `src/Factory/QueryFactory.php` and copy / paste this content:

```php
<?php

namespace App\Factory;

use Cake\Database\Connection;
use Cake\Database\Query;
use UnexpectedValueException;

final class QueryFactory
{
    private Connection $connection;

    /**
     * The constructor.
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
     * @throws UnexpectedValueException
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
     *
     * @return Query The new update query
     */
    public function newUpdate(string $table): Query
    {
        return $this->newQuery()->update($table);
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
        return $this->newQuery()
            ->insert(array_keys($data))
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
}

```

## Repository

You can inject the query factory instance into your repository like this:

```php
<?php

namespace App\Domain\User\Repository;

use App\Factory\QueryFactory;
use Cake\Database\StatementInterface;

final class UserRepository
{
    private QueryFactory $queryFactory;

    public function __construct(QueryFactory $queryFactory)
    {
        $this->queryFactory = $queryFactory;
    }

    // ...
}
```

## Usage

Once the query factory instance has been injected, you may use it like so:

### Select

Query all rows:

```php
$query = $this->queryFactory->newSelect('users')->select('*');
$rows = $query->execute()->fetchAll('assoc');

foreach ($rows as $row) {
    print_r($row);
}
```

Query the table with where:

```php
$query = $this->queryFactory->newSelect('users');

$query->select(['id', 'username']);
$select->andWhere(['id' => 1]);

$rows = $query->execute()->fetchAll('assoc');

foreach ($rows as $row) {
    print_r($row);
}
```

Query the table by id:

```php
$query = $this->queryFactory->newSelect('users')->andWhere(['id' => 1]);

$row = $query->execute()->fetch('assoc');
```

**Aggregate functions**

The [func()](https://api.cakephp.org/4.0/class-Cake.Database.Query.html#func) method 
returns an instance of a function builder object that can be used for 
generating arbitrary SQL functions.

```php
$query->select(['counter' => $query->func()->count('invoices.id')]);

$query->select(['amount_sum' => $query->func()->sum('payments.amount')]);

$query->select(['user_score_max' => $query->func()->max('users.score')]);
```

**Custom functions**

The [newExpr()](https://api.cakephp.org/4.0/class-Cake.Database.Query.html#newExpr) method returns a new QueryExpression object. 
This is a handy function when building complex queries using a fluent interface. 
You can also override this function in subclasses to use a more specialized
QueryExpression class if required.

```php
$query->select(
    [
        'date_of_birth' => $query->newExpr("DATE_FORMAT(users.date_of_birth,'%d.%m.%Y')"),
    ]
);
```

Read more: [Selecting data](https://book.cakephp.org/4/en/orm/query-builder.html#selecting-data)

### Insert

Insert a record

```php
$values = [
    'first_name' => 'john',
    'last_name' => 'doe',
    'email' => 'john.doe@example.com',
];

$this->queryFactory->newInsert('users', $values)->execute();
```

Insert a record and get the last inserted id:

```php
$newId = (int)$this->queryFactory->newInsert('users', $values)
    ->execute()
    ->lastInsertId();
```

Read more: [Inserting data](https://book.cakephp.org/4/en/orm/query-builder.html#inserting-data)

### Update

Update a record:

```php
$values = ['email' => 'new@example.com'];

$this->queryFactory->newUpdate('users')
    ->set($values)
    ->andWhere(['id' => 1])
    ->execute();
```

Read more: [Updating data](https://book.cakephp.org/4/en/orm/query-builder.html#updating-data)

### Delete 

Delete a record:

```php
$this->queryFactory->newDelete('users')
    ->andWhere(['id' => 1])
    ->execute();
```

Read more: [Deleting data](https://book.cakephp.org/4/en/orm/query-builder.html#deleting-data)

## Handling relationships

You can define relationships directly with a join clause.

In addition to `join()` you can use `rightJoin()`, `leftJoin()` 
and `innerJoin()` to create joins:

```php
$query = $this->queryFactory->newSelect('users');

$query->select(['users.*']);

$query->innerJoin('contacts', 'contacts.user_id = users.id');
$query->leftJoin('orders', 'orders.user_id = users.id');

$rows = $query->execute()->fetchAll('assoc');
```

## Transactions

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

use Cake\Database\Connection;

/**
 * Transaction handler.
 */
final class Transaction implements TransactionInterface
{
    /**
     * @var Connection The database connection
     */
    private $connection;

    public function __construct(Connection $connection)
    {
        $this->connection = $connection;
    }

    public function begin(): void
    {
        $this->connection->begin();
    }

    public function commit(): void
    {
        $this->connection->commit();
    }

    public function rollback(): void
    {
        $this->connection->rollback();
    }
}

```

Add a container definition for `TransactionInterface::class`:

```php
<?php

use App\Database\Transaction;
use App\Database\TransactionInterface;
use Cake\Database\Connection;
use Psr\Container\ContainerInterface;

return [

    // ...
    
    TransactionInterface::class => function (ContainerInterface $container) {
        return new Transaction($container->get(Connection::class);
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

        $this->transaction->begin();

        try {
            // ...

            $userId = $this->repository->insertUser($userData);
            
            // Do more database operations
            // ...    
    
            // Commit all changes
            $this->transaction->commit();
        } catch (Exception $exception) {
            // Revert all changes
            $this->transaction->rollback();
        }
    }
}
```