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
* [The connection proxy](#the-connection-proxy)
* [Extending PDO](#extending-pdo)
* [Autowired objects](#autowired-objects)
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

## The connection proxy

**Use case:** Two or more database connections with fixed configuration

Example:

```php
<?php

namespace App\Database;

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

Add the container definition as follows:

```php
return [
    ConnectionProxy::class => function (ContainerInterface $container) {
        return new ConnectionProxy(
            $container->get('db1'),
            $container->get('db2')
        );
    },
    'db1' => function (ContainerInterface $container) {
        return new PDO(...);
    },
    'db2' => function (ContainerInterface $container) {
        return new PDO(...);
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

## Extending PDO

Instead of using another implicit "container" for multiple connections, you could
extend a class from PDO to give each database connection a unique and
fullly qualified name for the container definiton.

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

Please note: If you don't like the class name `PDO2`, just give it a more specific name.

## Autowired objects

See Matthieu Napoli's answer: <https://stackoverflow.com/a/57758106/1461181>

Note that the use of [DI/autowire()](https://stackoverflow.com/a/57758106/1461181), 
could cause too much effort in container configuration and maintenance can 
become a nightmare in bigger projects.

## Read more

* Stackoverflow: <https://stackoverflow.com/a/57769575/1461181>

