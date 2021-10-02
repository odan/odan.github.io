---
title: Slim 4 - Laravel Query Builder
layout: post
comments: true
published: true
description: 
keywords: php slim laravel eloquent orm sql querybuilder
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
* [Fetching arrays](#fetching-arrays)
* [Multiple database connections](#multiple-database-connections)

## Requirements

* PHP 7.2+
* MySQL 5.7+
* Composer (for development)
* [A Slim 4 application](https://odan.github.io/2019/11/05/slim4-tutorial.html)

## Introduction

You can use the [Illuminate Query Builder](https://laravel.com/docs/master/queries) from Laravel
to connect your Slim 4 application to a database.

Please note: This concept requires no Eloquent ORM.

## Installation

To add the Query Builder to your application, run:

```
composer require illuminate/database
```

## Configuration

Add the database configuration to the settings array, e.g in `config/settings.php`:

```php
// Database settings
$settings['db'] = [
    'driver' => 'mysql',
    'host' => 'localhost',
    'database' => 'test',
    'username' => 'root',
    'password' => '',
    'charset' => 'utf8mb4',
    'collation' => 'utf8mb4_unicode_ci',
    'prefix' => '',
    'options' => [
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

Add a new container definition for `Connection::class` and `PDO::class`, e.g. in `config/container.php`:

```php
<?php

use Illuminate\Container\Container as IlluminateContainer;
use Illuminate\Database\Connection;
use Illuminate\Database\Connectors\ConnectionFactory;
use Psr\Container\ContainerInterface;
// ...

return [

    // ...
    
    // Database connection
    Connection::class => function (ContainerInterface $container) {
        $factory = new ConnectionFactory(new IlluminateContainer());

        $connection = $factory->make($container->get('settings')['db']);

        // Disable the query log to prevent memory issues
        $connection->disableQueryLog();

        return $connection;
    },

    PDO::class => function (ContainerInterface $container) {
        return $container->get(Connection::class)->getPdo();
    },
];
```

## Repository

The best place to use the query builder is within a repository.

To get the query build into the repository instance, 
it must be declared in the constructor as follows:

```php
<?php

namespace App\Domain\User\Repository;

use Illuminate\Database\Connection;

class UserRepository
{
    /**
     * @var Connection
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

    // Add your custom query methods here...
}
```

## Usage

Once the connection instance has been injected, you may use it as follows. 
Add a new method per query you want to provide for the service class. Example:

```php
public function getUserById(int $userId): array
{
    $row = $this->connection->table('users')->find($userId);

    if(!$row) {
        throw new \DomainException(sprintf('User not found: %s', $userId));
    }       

    return $row;
}
```

### Select

Query all rows:

```php
$rows = $this->connection->table('users')->get();
```

You can use the [where]((https://laravel.com/docs/master/queries#where-clauses)) 
method on a query builder instance to add where clauses to the query. 

```php
$userRows = $this->connection->table('users')->where('username', '=', 'admin')->get();
```

Query searching for names matching foo:

```php
$userRows = $this->connection->table('users')->where('username', 'like', '%root%')->get();
```

Query the table by id:

```php
$userRow = $this->connection->table('users')->find(1);
```

### Insert

The query builder also provides an [insert](https://laravel.com/docs/master/queries#inserts)
method for inserting records into the database table. 
The insert method accepts an array of column names and values:

```php
$values = [
    'first_name' => 'john',
    'last_name' => 'doe',
    'email' => 'john.doe@example.com',
];

$this->connection->table('users')->insert($values);
```

Insert a record and get the last inserted id:

```php
$newId = $this->connection->table('users')->insertGetId($values);
```

### Update

In addition to inserting records into the database, 
the query builder can also update existing records using 
the [update](https://laravel.com/docs/master/queries#updates) method.

```php
$values = ['email' => 'new@example.com'];

$this->connection->table('users')
    ->where(['id' => 1])
    ->update($values);
```

### Delete

The query builder can also be used to [delete](https://laravel.com/docs/master/queries#deletes) 
records from the table via the delete method.

```php
$this->connection->table('users')->delete(1);
```

or

```php
$this->connection->table('users')
    ->where(['id' => 1])
    ->delete();
```

## Handling relationships

You can define relationships directly with a [join clause](https://laravel.com/docs/master/queries#joins).

```php
$userRows = $this->connection->table('users')
    ->select('users.*', 'contacts.phone', 'orders.price')
    ->join('contacts', 'users.id', '=', 'contacts.user_id')
    ->join('orders', 'users.id', '=', 'orders.user_id')
    ->get();
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

## Fetching arrays

Since Laravel 5.4, the default (and only) fetch mode is `PDO::FETCH_OBJ`. 

But you still have at least two options to change the fetch mode to `PDO::FETCH_ASSOC`.

### Option 1: Extending MySqlConnection

You can extend from `\Illuminate\Database\MySqlConnection` and define a resolver for it.

```php
class MySqlConnectionAssocArray extends \Illuminate\Database\MySqlConnection
{
    public function __construct($connection, $database, $prefix, $config)
    {
        parent::__construct($connection, $database, $prefix, $config);

        $this->fetchMode = \PDO::FETCH_ASSOC;
    }
}
```

The resolver:

```php
\Illuminate\Database\Connection::resolverFor('mysql', function($connection, $database, $prefix, $config) {
    return new \App\Database\MySqlConnectionAssocArray($connection, $database, $prefix, $config);
});
```

*Thanks to `devinim` for this tip.*

### Option 2: Using events

Installation: `composer require illuminate/events`

Register this event handler in your container definition for `Connection::class`:

```php
// ...

return [
    // ...
    
    // Database connection
    Connection::class => function (ContainerInterface $container) {
        $factory = new ConnectionFactory(new IlluminateContainer());

        $connection = $factory->make($container->get('settings')['db']);

        // Disable the query log to prevent memory issues
        $connection->disableQueryLog();

        $dispatcher = new \Illuminate\Events\Dispatcher();
        $connection->setEventDispatcher($dispatcher);

        $dispatcher->listen(\Illuminate\Database\Events\StatementPrepared::class, function ($event) {
            $event->statement->setFetchMode(PDO::FETCH_ASSOC);
        });

        return $connection;
    },
    
    // ...
];
```

**Usage:**

```php
$rows = $this->connection->table('users')->get()->toArray();
```

## Multiple database connections

The following example shows how you can use the Eloquent Query Builder to set up 
multiple database connections and access them in a repository.

Extend your second connection from `Illuminate\Database\Connection`:

Create a new file: `src/App/Database/SecondConnection.php`

```php
<?php

namespace App\Database;

use Illuminate\Database\MySqlConnection;

class SecondConnection extends MySqlConnection
{

}
```

Register a new container definition for the second database connection.

Please note: You also need new connection configuration (e.g. db2) and a new driver name 
(e.g. mysql2) for the second connection parameters.

```php
// Database settings
$settings['db2'] = [
    'driver' => 'mysql2',
    // ...
```

```php

// ...
use App\Database\SecondConnection;
use Illuminate\Database\Connection;
// ...

return [

    // ...
    
    // Database connection
    SecondConnection::class => function (ContainerInterface $container) {
        $factory = new ConnectionFactory(new IlluminateContainer());

        // Resolve the driver
        Connection::resolverFor('mysql2', function ($connection, $database, $prefix, $config) {
            return new SecondConnection($connection, $database, $prefix, $config);
        });
        
        $connection = $factory->make($container->get('settings')['db2']);

        // Disable the query log to prevent memory issues
        $connection->disableQueryLog();

        return $connection;
    },
];
```

Inject the second database connection into the repository.

If you need multiple connections, define them in the constructor parameter list as follows:

```php
<?php

namespace App\Domain\User\Repository;

use Illuminate\Database\Connection;
use App\Database\SecondConnection;

class UserRepository
{
    /**
     * @var Connection
     */
    private $connection;

    /**
     * @var SecondConnection
     */
    private $connection2;

    /**
     * The constructor.
     *
     * @param Connection $connection The database connection
     * @param SecondConnection $connection2 The second database connection
     */
    public function __construct(Connection $connection, SecondConnection $connection2)
    {
        $this->connection = $connection;
        $this->connection2 = $connection2;
    }

    // ...
}
```

## Read more

* [Laravel Database Query Builder](https://laravel.com/docs/master/queries)
* [Illuminate Database on Github](https://github.com/illuminate/database)
