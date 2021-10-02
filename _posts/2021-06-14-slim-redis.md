---
title: Slim 4 - Redis
layout: post
comments: true
published: true 
description:
keywords: php, slim, redis
image: https://odan.github.io/assets/images/slim-logo-600x330.png
---

## Table of contents

* [Introduction](#introduction)
* [Requirements](#requirements)
* [Redis Setup on Linux](#redis-setup-on-linux)
* [Redis Setup on Windows](#redis-setup-on-windows)
* [Installation](#installation)
* [Configuration](#configuration)
* [DI Container Setup](#di-container-setup)
* [Cache Keys](#cache-keys)  
* [Usage](#usage)
* [Cache Invalidation](#cache-invalidation)
* [Stampede prevention](#stampede-prevention)
* [Conclusion](#conclusion)
* [Read more](#read-more)

## Requirements

* PHP 7.4+ or PHP 8.0+
* [A Slim 4 application](https://odan.github.io/2019/11/05/slim4-tutorial.html)
* At least one Redis server must be installed and running.

## Introduction

Redis is an in-memory database with a simple key-value data structure and belongs to the family of NoSQL databases.
Redis is open source and, according to a 
[survey by db-engines.com](https://db-engines.com/en/ranking/key-value+store),
the most popular key-value store.

Redis is very often used for as a **session cache**.
The advantages of using Redis over other session stores, such as Memcached,
is that Redis offers persistence.

Outside of basic session tokens, Redis can be used as a **Full Page Cache (FPC)**
to make your websites even faster.

Please note that Redis is primarily a key-value store and
[not a task queue](https://redislabs.com/ebook/part-2-core-concepts/chapter-6-application-components-in-redis/6-4-task-queues/).
So if you are looking for a proper queuing system, you should take a look at more
specific solutions like RabbitMQ, Amazon SQS, and others.

## Redis Setup on Linux

For this demo you need at least a running Redis server.

Please follow the official installation guide of Redis:

* [Redis Quick Start](https://redis.io/topics/quickstart)
* [How To Install and Secure Redis on Ubuntu 18.04](https://www.digitalocean.com/community/tutorials/how-to-install-and-secure-redis-on-ubuntu-18-04)

## Redis Setup on Windows

For people like me who develop on Windows, there is a special setup file for the Redis server.

* Download Redis for Windows: <https://github.com/microsoftarchive/redis/releases>
* Extract the ZIP file
* Open `cmd` and `cd` into the `Redis-x64-3.2.100` directory.
* The Redis servers starts automatically as a Windows service. To start the Redis server manually, run: `C:\Program Files\Redis\redis-server.exe`
* The server is now ready to accept connections on port `6379`

## Installation

The `predis/predis` component is a flexible and feature-complete Redis client for PHP 7.2 and newer.

To install predis, run:

```
composer require predis/predis
```

## Configuration

Insert the settings into your configuration file, e.g. `config/settings.php`;

```php
$settings['redis'] = [
    'server' => 'tcp://127.0.0.1:6379',
    'options' => null,
];
```

For performance reason you may better use the IP address of the redis server.

## DI Container Setup

Add a DI container definition for `Predis\ClientInterface:class` in `config/container.php`.

```php
<?php

use Predis\Client;
use Predis\ClientInterface;
use Psr\Container\ContainerInterface;
// ...

return [
    // ...
    
    ClientInterface::class => function (ContainerInterface $container) {
        $settings = $container->get('settings')['redis'];

        return new Client($settings['server'], $settings['options']);
    },
];
```

## Cache keys

According to PSR-6 the following characters could be used in a valid cache key; 
`A-Z`, `a-z`, `0-9`, `_`, and `.` Some characters are forbidden like: `{}()/\@:`. 

If you use any of the forbidden characters you will get an exception. 
Other characters (like `-`) are not forbidden nor valid. 
It is up to the implementation if they support that character or not.

**I recommend to always use valid characters in the cache key.**

To make sure you do not use an invalid character by mistake, you should hash your keys.

## Usage

To access the `Predis\Client` object, we must first declare the
`Predis\ClientInterface` in the constructor, so that it can be automatically 
injected by the DI Container.

Example:

```php
<?php

namespace App\Action\Example;

use Predis\ClientInterface;
use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface;

final class ExampleAction
{
    private ClientInterface $cache;

    public function __construct(ClientInterface $cache)
    {
        $this->cache = $cache;
    }

    public function __invoke(
        ServerRequestInterface $request, 
        ResponseInterface $response
    ): ResponseInterface {
        $value = $this->cache->get('my_cache_key');

        if ($value === null) {
            // ... do some HTTP request or heavy computations
            $value = 'foobar-' . date('H:i:s');

            // Expire after x seconds
            $expire = 3;

            // Save computed value
            $this->cache->setex('my_cache_key', $expire, $value);
        }

        $response->getBody()->write($value);

        return $response;
    }
}

```

### Cache Invalidation

Removing the cache key:

```php
$this->cache->del('my_cache_key');
```

Deleting more than one key at the same time:

```php
$this->cache->del([
    'my_cache_key',
    'my_cache_key2',
    'my_cache_key3',
]);
```

Deleting all keys from the connection's current database:

```php
$this->cache->flushdb();
```

Deleting all keys from all databases.

```php
$this->cache->flushall();
```

## Stampede prevention

Cache is usually used to reduce performing a complex operation. In case of a
cache miss, that operation is executed & the result is stored.

A [cache stampede](https://en.wikipedia.org/wiki/Cache_stampede) happens when 
there are a lot of requests for data that is not currently in cache.

Examples could be:

* cache expires for something that is often under very heavy load
* sudden unexpected high load on something that is likely to not be in cache

In those cases, this huge amount of requests for data that is not at that
time in the cache, causes that expensive operation to be executed a lot of times,
all at once.

The first solution is to use **locking**: only allow one PHP process (on a per-host basis) 
to compute a specific key at a time. Locking a key is [not built-in](https://redis.io/topics/distlock) 
by default, but can be added for example with the [Symfony Lock](https://symfony.com/doc/current/components/lock.html)
component. In reality, you don't need to reinvent to this algorithm, because
the [Symfony Cache](https://symfony.com/doc/current/components/cache.html) component
comes with a built-in Stampede prevention.

## Conclusion

As you can see, Redis can be integrated very quickly and easily into a Slim project. 
However, it must be mentioned that this solution is not well testable with phpunit.
For this reason, I will explain a more flexible and testable solution using 
the Symfony Cache component in the next article.

## Read more

* [The Predis Github Repository](https://github.com/predis/predis)
* [Ask a question or report an issue](https://github.com/odan/slim4-tutorial/issues)