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
* [Usage](#usage)
* [Repository](#repository)

## Requirements

* PHP 7.1+
* MySQL 5.7+
* Composer
* [A Slim 4 application](https://odan.github.io/2019/11/05/slim4-tutorial.html)

## Introduction

You can use a database ORM such as [Eloquent](https://laravel.com/docs/eloquent) to connect 
your SlimPHP application to a database.

## Installation

To add Eloquent to your application, run:

```
composer require illuminate/database
```

## Configuration

Add the database settings to Slim’s settings array.

, e.g `config/settings.php`:

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
use Selective\Config\Configuration;
use Slim\App;
use Slim\Factory\AppFactory;

return [

    // ...
    
    // Database connection
    Manager::class => function (ContainerInterface $container) {
        $capsule = new \Illuminate\Database\Capsule\Manager;
        $capsule->addConnection($container->get(Configuration::class)->getArray('db'));

        $capsule->setAsGlobal();
        $capsule->bootEloquent();

        return $capsule;
    },

    PDO::class => function (ContainerInterface $container) {
        $db = $container->get(Connection::class);
        $db->getDriver()->connect();

        return $db->getDriver()->getConnection();
    },
];
```

Note: `composer require illuminate/events` is required when you need to use observers with Eloquent.


## Repository

You can inject the capsule instance into your repository like this:

```php
<?php

namespace App\Domain\User\Repository;

use App\Domain\User\Data\UserCreateData;
use Illuminate\Database\Capsule\Manager;

class UserRepository
{
    /**
     * @var Manager The query builder factory
     */
    private $database;

    /**
     * The constructor.
     *
     * @param Manager $database The query builder
     */
    public function __construct(Manager $database)
    {
        $this->database = $database;
    }

    // ...
}
```

## Usage

Once the Capsule instance has been injected, you may use it like so:

#### Query all rows

```php
$rows = $this->database->table('users')->get();
```

#### Query the table with where

Query searching for names matching foo:

```php
$rows = $this->database->table('users')->where('username', 'like', '%root%')->get();
```

#### Query the table by id

Selecting a row based on id:

```php
$row = $this->database->table('users')->find(1);
```

#### Insert a record

```php
$values = [
    'first_name' => 'john',
    'last_name' => 'doe',
    'email' => 'john.doe@example.com',
];

$this->database->table('users')->insert($values);
```

Insert a record and get the last inserted id:

```php
$newId = $this->database->table('users')->insertGetId($values);
```

#### Update a record

```php
$this->database->table('users')
    ->where(['id' => 1])
    ->update(['email' => 'new@example.com']);
```

#### Delete a record

```php
$this->database->table('users')->delete(1);
```

or

```php
$this->database->table('users')
    ->where(['id' => 1])
    ->delete();
```

**Read more:**

* [Eloquent documentation](https://laravel.com/docs/master/eloquent)
* [Illuminate Database on Github](https://github.com/illuminate/database)