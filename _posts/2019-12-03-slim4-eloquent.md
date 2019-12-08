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

## Requirements

* PHP 7.1+
* MySQL 5.7+
* Composer
* [A Slim 4 application](https://odan.github.io/2019/11/05/slim4-tutorial.html)

## Introduction

You can use a database ORM such as [Eloquent](https://laravel.com/docs/eloquent) to connect 
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

You can inject the capsule instance into your repository like this:

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

## Read more

* [Eloquent documentation](https://laravel.com/docs/master/eloquent)
* [Illuminate Database on Github](https://github.com/illuminate/database)
