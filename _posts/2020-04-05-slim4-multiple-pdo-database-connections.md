---
title: Slim 4 - Multiple PDO database connections
layout: post
comments: true
published: true
description: 
keywords: php slim pdo database connection container phpdi
---

## Table of contents

* [Requirements](#requirements)
* [Introduction](#introduction)
* [Two database connections with fixed configuration](#two-database-connections-with-fixed-configuration)
* [First database fixed, second database with dynamic configuration](#first-database-fixed-second-database-with-dynamic-configuration)
* [Read more](#read-more)

## Requirements

* PHP 7.2+
* [A Slim 4 application](https://odan.github.io/2019/11/05/slim4-tutorial.html)

## Introduction

After reading the [Slim 4 tutorial](https://odan.github.io/2019/11/05/slim4-tutorial.html), 
people asked me how to add multiple database connections to a Slim application.

The challenge here is to configure the container so that different instances of a 
PDO instance are configured and injected into our repository classes. 

The problem is that the container just maps a string 
(e.g. a fully qualified class name) to an object. But since `PDO::class` 
can only be mapped once, we have to think of something else. 

Depending on the use case, I will present here several generic solutions, 
which can be further customized. 

## Two database connections with fixed configuration

You have multiple options:

1. The connection proxy
2. Extending PDO
3. Autowired objects

### The  connection proxy

Example:

```php
use PDO;

class ConnectionProxy
{
    private $pdo;

    private $pdo2;

    public function __construct(PDO $pdo, PDO $pdo2)
    {
        $this->pdo = $pdo;
        $this->pdo2 = $pdo2;
    }

    public function getPdo(): PDO
    {
        return $this->pdo;
    }

    public function getPdo2(): PDO
    {
        return $this->pdo2;
    }
}
```

The container definition:

```php
return [
    ConnectionProxy::class => function (ContainerInterface $c) {
        return new ConnectionProxy(
            $c->get('db1'),
            $c->get('db2')
        );
    },
    'db1' => function (ContainerInterface $c) {
        return new PDO();
    },
    'db2' => function (ContainerInterface $c) {
        return new PDO();
    },
];
```

**Usage**

```php
class MyRepository
{
    private $pdo;

    private $pdo2;
    
    public function __construct(ConnectionProxy $connectionProxy)
    {
        $this->pdo = $connectionProxy->getPdo();
        $this->pdo2 = $connectionProxy->getPdo2();
    }
}
```

### 2. Extending PDO

Create a file: src/Database/PDO2.php and copy/paste this content:

```php

namespace App\Database;

use PDO;

class PDO2 extends PDO
{

}
```

Add the container definition for PDO2:

```
use App\Database\PDO2;

// ...

return [
    PDO2::class => function (ContainerInterface $c) {
        return new PDO2();
    },
];
```

**Usage**

```php
namespace App\Domain\User\Repository;

use PDO;
use PDO2; 

class UserRepository
{
    private $pdo;

    private $pdo2;
    
    public function __construct(PDO $pdo, PDO2 $pdo2)
    {
        $this->pdo = $pdo;
        $this->pdo2 = $pdo2;
    }
}
```

### 3. Autowired objects

See Matthieu Napoli's answer: https://stackoverflow.com/a/57758106/1461181

Please note that I am not a fan of [DI\autowire()](https://stackoverflow.com/a/57758106/1461181), as this would require 
too much effort in container configuration and maintenance is a nightmare in larger projects.
I think that the DIC should do this job for us as automatically as possible.

## First database fixed, second database with dynamic configuration

Imagine that you need at least two database connection instances.

1. The default connection for the primary database.
2. The second connection for a dynamic database connection.

I asume that the parameters for the default connection is always the same. 
So there is nothing special to setup here.

For the second connection you may create a ConnectionManager class, which creates a
dynamic database connection on demand for you.

### Settings

Add the default settings for the second connection. The database is not defined here.

File: config/defaults.php

```php
// Database settings for the second connection
$settings['db2'] = [
    'driver' => 'mysql',
    'host' => 'localhost',
    'username' => 'root',
    'password' => '',
    'charset' => 'utf8mb4',
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

Add the ConnectionManager class.

File: src/Database/ConnectionManager.php

```php
<?php

namespace App\Database;

use PDO;
use UnexpectedValueException;

final class ConnectionManager
{
    /**
     * @var array
     */
    private $settings;

    /**
     * @var PDO|null
     */
    private $connection;

    public function __construct(array $settings)
    {
        $this->settings = $settings;
    }

    public function createConnection(
        string $database,
        string $username = null,
        string $password = null
    ): PDO {
        $host = $this->settings['host'];
        $username = $username ?? $this->settings['username'];
        $password = $password ?? $this->settings['password'];
        $charset = $this->settings['charset'];
        $flags = $this->settings['flags'];
        $dsn = "mysql:host=$host;dbname=$database;charset=$charset";

        $this->connection = new PDO($dsn, $username, $password, $flags);

        return $this->connection;
    }

    public function getConnection(): PDO
    {
        if (!isset($this->connection)) {
            throw new UnexpectedValueException('Database connection not found');
        }

        return $this->connection;
    }
}
```

Add the DatabaseMiddleware.

File: src/Middleware/DatabaseMiddleware.php

```php
<?php

namespace App\Middleware;

use App\Database\ConnectionManager;
use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface;
use Psr\Http\Server\MiddlewareInterface;
use Psr\Http\Server\RequestHandlerInterface;

final class DatabaseMiddleware implements MiddlewareInterface
{
    /**
     * @var ConnectionManager
     */
    private $connectionManager;

    public function __construct(ConnectionManager $connectionFactory)
    {
        $this->connectionManager = $connectionFactory;
    }

    public function process(ServerRequestInterface $request, RequestHandlerInterface $handler): ResponseInterface
    {
        // fetch the database name from the JWT
        $databaseName = $request->getAttribute('database');

        if ($databaseName) {
            $this->connectionManager->createConnection($databaseName);
        }

        return $handler->handle($request);
    }
}
```

Add the DatabaseMiddleware to the stack:

File: config/middleware.php

```php
use App\Middleware\DatabaseMiddleware;

// ...

$app->add(DatabaseMiddleware::class);

```

Add this container definiton in config/container.php

```php
use App\Database\ConnectionManager;

// ...
ConnectionManager::class=> function (ContainerInterface $container) {
    // Get default database settings for the second connection
    // The database name is not required here
    return new ConnectionManager($container->get(Configuration::class)->getArray('db2'));
},
```

### Usage

Now you have two options to use the second connection within your repositories.

1. Option: Let the dependecy injection container inject the ConnectionManager instance, then fetch the connection from it.

```php
<?php

namespace App\Domain\User\Repository;

use App\Database\ConnectionManager;
use PDO;

class UserRepository
{
    /**
     * @var PDO The second database connection
     */
    private $pdo;

    public function __construct(ConnectionManager $connectionManager)
    {
        $this->pdo = $connectionManager->getConnection();
    }
    
    public function demo()
    {
         $row = $this->pdo->query('SELECT database();')->fetch();
         // ...
    }
    
    // ...
}
```

## Read more

* Stackoverflow: <https://stackoverflow.com/a/57769575/1461181>

[Comments](https://gist.github.com/odan/5843e4e69141501e4d3ba8ff99c0b1d9) | 
[Donate](../../../donate.md)
