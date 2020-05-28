---
title: Slim 4 - Eloquent Setup
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
* [Fetching arrays](#fetching-arrays)
* [Setup multiple connections](#setup-multiple-connections)

## Requirements

* PHP 7.1+
* MySQL 5.7+
* Composer
* [A Slim 4 application](https://odan.github.io/2019/11/05/slim4-tutorial.html)

## Introduction

You can use a database query builder such as [Eloquent](https://laravel.com/docs/eloquent) to connect 
your Slim 4 application to a database.

## Installation

To add Eloquent to your application, run:

```
composer require illuminate/database
```

## Configuration

Add the database settings to Slim’s settings array, e.g `config/settings.php`:

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

In your `config/container.php` or wherever you add your container definitions:

```php
<?php

use Illuminate\Container\Container as IlluminateContainer;
use Illuminate\Database\Connection;
use Illuminate\Database\Connectors\ConnectionFactory;
use Psr\Container\ContainerInterface;
use Selective\Config\Configuration;
use Slim\App;
use Slim\Factory\AppFactory;

return [

    // ...
    
    // Database connection
    Connection::class => function (ContainerInterface $container) {
        $factory = new ConnectionFactory(new IlluminateContainer());

        $connection = $factory->make($container->get(Configuration::class)->getArray('db'));

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

You can inject the connection instance into your repository like this:

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

    // ...
}
```

## Usage

Once the connection instance has been injected, you may use it as follows:

### Query all rows

```php
$rows = $this->connection->table('users')->get();
```

### Query the table with where

*Query searching for names matching foo*

```php
$rows = $this->connection->table('users')->where('username', 'like', '%root%')->get();
```

### Query the table by id

```php
$row = $this->connection->table('users')->find(1);
```

### Insert a record

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

### Update a record

```php
$values = ['email' => 'new@example.com'];

$this->connection->table('users')
    ->where(['id' => 1])
    ->update($values);
```

### Delete a record

```php
$this->connection->table('users')->delete(1);
```

or

```php
$this->connection->table('users')
    ->where(['id' => 1])
    ->delete();
```

### Fetching arrays

In Laravel 5.4, the default (and only) fetch mode is `PDO::FETCH_OBJ`. The only global way to change this is to now rely on events.

You still have at least two options to change the fetch mode to array.

### Option 1: Extending MySqlConnection

You can extend from `\Illuminate\Database\MySqlConnection` and define a resolver for it.

```php
\Illuminate\Database\Connection::resolverFor('mysql', function($connection, $database, $prefix, $config) {
    return new \App\Database\MySqlConnectionAssocArray($connection, $database, $prefix, $config);
});
```

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

*Thanks to devinim for this tip.*

### Option 2: Using events

Installation: `composer require illuminate/events`

Register this event handler in your container definiton for `Connection::class`:

```php
$dispatcher = new \Illuminate\Events\Dispatcher();
$db->setEventDispatcher($dispatcher);

$dispatcher->listen(\Illuminate\Database\Events\StatementPrepared::class, function ($event) {
    $event->statement->setFetchMode(PDO::FETCH_ASSOC);
});
```

**Usage:**

```php
$db = $this->get('db');
$rows = $db->table('information_schema.schemata')->get()->toArray();
```

Downside: Events are a poor workaround for this, as mocking an event for testing will 
then prevent the fetch mode event from firing, and if your app relies on a different 
fetch mode you now may get issues with your tests.

### Setup multiple connections

The following example shows how you can use the Eloquent Query Builder to set up 
multiple database connections and access them in a repository.

1. Extend your second connection from `Illuminate\Database\Connection`:

Create a new file: `src/App/Database/SecondConnection.php`

```php
<?php

namespace App\Database;

use Illuminate\Database\MySqlConnection;

class SecondConnection extends MySqlConnection
{

}
```

2. Register a new container definition for the second database connection.

Please note: You also need new connection configuration (e.g. db2) and a new driver name (e.g. mysql2) for the second connection parameters.

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
        
        $connection = $factory->make($container->get(Configuration::class)->getArray('db2'));

        // Disable the query log to prevent memory issues
        $connection->disableQueryLog();

        return $connection;
    },
];
```

3. Inject the database connections into the repository

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

* [Eloquent documentation](https://laravel.com/docs/master/eloquent)
* [Illuminate Database on Github](https://github.com/illuminate/database)
