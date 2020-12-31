---
title: Slim 4 - Multiple PDO database connections
layout: post
comments: true
published: true
description: 
keywords: php slim pdo database connection container
---

## Table of contents

* [Requirements](#requirements)
* [Introduction](#introduction)
* [Extending PDO](#extending-pdo)
* [Autowired objects](#autowired-objects)
* [Multitenancy](#multitenancy)

## Requirements

* PHP 7.2+
* [A Slim 4 application](https://odan.github.io/2019/11/05/slim4-tutorial.html)

## Introduction

After reading the [Slim 4 tutorial](https://odan.github.io/2019/11/05/slim4-tutorial.html), people asked me how to add
multiple database connections to a Slim application.

The challenge here is to configure the container so that different instances of a PDO instance are configured and
injected into our repository classes.

The problem is that the container just maps a string
(e.g. a fully qualified class name) to an object. But since `PDO::class`
can only be mapped once, we have to think of something else.

Depending on the use case, I will present here several generic solutions, which can be further customized.

## Extending PDO

The dependency injection container requires a fully qualified name for each connection. For this we have to extend a
class from PDO to create a new container definition.

Create a file: src/Database/PDO2.php and copy/paste this content:

```php
<?php

namespace App\Database;

use PDO;

class PDO2 extends PDO
{
    // must be empty
}
```

Add the container definition for PDO2:

```php
use App\Database\PDO2;

// ...

return [
    //...

    PDO2::class => function (ContainerInterface $container) {
        return new PDO2(...);
    },
];
```

**Usage**

```php
namespace App\Domain\User\Repository;

use PDO;
use App\Database\PDO2; 

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

If you don't like the class name `PDO2`, just give it a more specific name.

## Autowired objects

See Matthieu Napoli's answer: <https://stackoverflow.com/a/57758106/1461181>

Note that the use of [DI/autowire()](https://stackoverflow.com/a/57758106/1461181), 
could cause too much effort in container configuration and maintenance can become 
a nightmare in bigger projects.

## Multitenancy

Multitenancy is when a single instance of a software application serves multiple customers 
within one architecture. Each customer is referred to as a client (tenant).

If desired, clients are given the ability to customize some parts of the application, 
such as the graphical user interface (GUI) design or business rules; however, 
they cannot customize the application's code.

In a multi-tenant architecture, multiple instances of an application operate 
in a shared environment. This works by keeping each client physically integrated 
but logically separate. Thus, a single instance of the software runs on one server, 
but serves multiple clients. In this way, an application in a multi-tenant architecture 
can use a dedicated instance of configurations, data, user management 
and other features for all clients.

The first approach from above makes sense when you have multiple database connections 
with a fix configuration. In another environments you may need to configure the 
second connection dynamically, for example by a JWT, or the session of the HTTP request.

Imagine that you need at least two database connection instances.

1. The default connection as the primary database.
2. The second connection for (the tenant) as dynamic database connection.

I assume that the parameters for the default connection is always the same. 
So there is nothing special to set up here.

For the second connection you have multiple other options depending on your use case.

**The simple approach:**
* The DI container creates an in-memory sqlite connection for the second database.
* Later connect the second PDO instance with the real tenant database when needed. 

Example:

```php
<?php

use App\Database\PDO2;
use Psr\Container\ContainerInterface;
// ...

return [

    // ...

    PDO2::class=> function (ContainerInterface $container) {
        // Container definition
        $options = [
                PDO::ATTR_PERSISTENT => false,
                PDO::ATTR_EMULATE_PREPARES => false,
                PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
                PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC
        ];

        // Initial connection, only in-memory
        return new PDO2('sqlite::memory:', null, null, $options);
    },

];
```

To change the connection of the second PDO instance, 
just invoke the constructor with the new connection parameters:

```php
// Connect to the real database, e.g. MySQL
$pdo2->__construct($dsn, $username, $password, $options);
```

**The complex approach:**

For the second connection you need a `ConnectionFactory` class, 
which creates all dynamic database connections on demand 
and a `ConnectionManager` that holds the connection instances.

First, create a `ConnectionManagerInterface` interface, e.g. in `src/Database/ConnectionManagerInterface`.

```php
<?php

namespace App\Database;

use PDO;

interface ConnectionManagerInterface
{
    public function setConnection(string $name, PDO $pdo): void;

    public function getConnection(string $name): PDO;
}
```

Then add the implementation of the interface, in `src/Database/ConnectionManager.php`:

```php
<?php

namespace App\Database;

use PDO;

final class ConnectionManager implements ConnectionManagerInterface
{
    /**
     * @var array<PDO>
     */
    private array $connections = [];

    public function setConnection(string $name, PDO $pdo): void
    {
        $this->connections[$name] = $pdo;
    }

    public function getConnection(string $name): PDO
    {
        return $this->connections[$name];
    }
}
```

Next we need a factory that creates the actual new PDO instance. 

Create a new class `ConnectionFactory`. 

File: `src/Database/ConnectionFactory.php`

```php
<?php

namespace App\Database;

use PDO;

final class ConnectionFactory
{
    /**
     * @var array
     */
    private $settings;

    /**
     * The constructor.
     *
     * @param array $settings The default connection settings
     */
    public function __construct(array $settings)
    {
        $this->settings = $settings;
    }

    public function createConnection(
        string $database,
        string $username = null,
        string $password = null
    ): PDO {
        // Add your custom logic here... 
        // The credentials could be fetched from another place
        // for example from the primary database connection
        $driver = $this->settings['driver'] ?? 'mysql';
        $host = $this->settings['host'] ?? '127.0.0.1';
        $username = $username ?? $this->settings['username'];
        $password = $password ?? $this->settings['password'];
        $charset = $this->settings['charset'];
        $flags = $this->settings['flags'];
        $dsn = "$driver:host=$host;dbname=$database;charset=$charset";

        return new PDO($dsn, $username, $password, $flags);
    }
}
```

The `ConnectionFactory` requires some default settings, and a custom container definition
to inject these default settings:

```
// The ConnectionFactory default settings
$settings['db2'] = [
    'driver' => 'mysql',
    'host' => 'localhost',
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

Insert a DI container definition for `ConnectionFactory:class` in `config/container.php`:

```php
<?php

use App\Database\ConnectionFactory;
use Psr\Container\ContainerInterface;
// ...

return [

    // ...

    ConnectionFactory::class=> function (ContainerInterface $container) {
        // Get default database settings for the second connection
        // The database name is not required here
        $db2Settings = $container->get('settings')['db2'];
        
        return new ConnectionFactory($db2Settings);
    },

];

```

Add a new connection using the ConnectionManager and ConnectionFactory: 

```php
$pdo = $connectionFactory->createConnection('customerdb', 'username', 'password');
$connectionManager->setConnection('customer', $pdo);
```

You could, for example add this connection within a middleware or where it makes sense.

To get the second connection you have to inject the ConnectionManager instance:

```php
public function __construct(ConnectionManager $connectionManager) {
    $this->pdo2 = $connectionManager->getConnection('customer');
}
```
