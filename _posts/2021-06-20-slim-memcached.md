---
title: Slim 4 - Memcached
layout: post
comments: true
published: true
description:
keywords: php, slim, memcached
image: https://odan.github.io/assets/images/slim-logo-600x330.png
---

## Table of contents

* [Introduction](#introduction)
* [Requirements](#requirements)
* [Memcached Setup on Linux](#memcached-setup-on-linux)
* [Memcached Setup on Windows](#memcached-setup-on-windows)
* [Configuration](#configuration)
* [DI Container Setup](#di-container-setup)
* [Cache Keys](#cache-keys)
* [Usage](#usage)
* [Cache Invalidation](#cache-invalidation)
* [Conclusion](#conclusion)
* [Read more](#read-more)

## Requirements

* PHP 7.4+ or PHP 8.0+
* [A Slim 4 application](https://odan.github.io/2019/11/05/slim4-tutorial.html)
* At least one Memcached server must be installed and running.

## Introduction

[Memcached](http://memcached.org/) is a free & open source, high-performance, distributed memory object caching system, 
generic in nature, but intended for use in speeding up dynamic web applications by alleviating database load.

Memcached is an in-memory key-value store for small chunks of arbitrary data (strings, objects) 
from results of database calls, API calls, or page rendering.

Please note that it doesn't always make sense to add memcached to your application.
To guarantee latency and speed memcached stores everything in RAM.
Even if the memcache instance appears to have free space, it may need to remove data due to memory limitations.

For this reason memcached is
[not recommended for sessions](https://github.com/memcached/memcached/wiki/ProgrammingFAQ#why-is-memcached-not-recommended-for-sessions-everyone-does-it),
as a database, and a (reliable) queue server, because the data would be lost if you restart the server
or when Memcached flushes its memory.

If you need **persistence**, you might want to look at other solutions like
[Redis](https://odan.github.io/2021/06/14/slim-redis.html).

Please note that there are two PHP extensions,
`php-memcached` and `php-memcache`, which are not identical.

PHP-Memcache is older, very stable but has a few limitations.
The PHP Memcache module utilizes the daemon directly, 
while the PHP-Memcached module uses the libMemcached client 
library and contains some added features.

You can compare features and differences between them 
[here](https://github.com/memcached/old-wiki/blob/master/PHPClientComparison.wiki).

## Memcached Setup on Linux

For this demo you need at least a running Memcached instance.

Please follow the official installation guide: [Memcached Installation](https://github.com/memcached/memcached/wiki/Install)

Install memcached and libmemcached:

```
sudo apt install memcached libmemcached-tools
```

Check service status:

```
sudo systemctl status memcached
```

If the service is not startet, restart with:

```
sudo service memcached restart
```

Install the php-memcache extension:

```
sudo apt install php-memcache
sudo apt install php8.0-memcache
```

Optional, install the php-memcached extension:

```
sudo apt install php-memcached
sudo apt install php8.0-memcached
```

Restart Apache:

`sudo service apache2 restart`

## Memcached Setup on Windows

Memcached works on most Linux and BSD like systems. There is no official support for windows builds.
For people like me who develop on Windows, there is a free binary version.

* Download Memcached for Windows: <https://github.com/nono303/memcached/releases>
* Extract the ZIP file
* Open `cmd` and `cd` into the `memcached-1.6.9/cygwin/x64` directory.
* To start the Memcached server, run: `memcached.exe`
* The server is now ready to accept connections on port `11211`

The Memcached API is available as a PHP [PECL extension](https://www.php.net/manual/en/book.memcache.php). 
Information for installing this PECL extension may be found
on [PHP.net](https://www.php.net/manual/en/memcache.installation.php)

* Open: <http://pecl.php.net/package/memcache/4.0.5.2/windows>
* Download the `7.x Thread Safe (TS) x64` version, e.g. [7.4 Thread Safe (TS) x64](https://windows.php.net/downloads/pecl/releases/memcache/4.0.5.2/php_memcache-4.0.5.2-7.4-ts-vc15-x64.zip)
* Extract the ZIP file
* Copy the file `php_memcache.dll` to `c:\xampp\php\ext`
* Open the file `c:\xampp\php\php.ini` with Notepad++ and add this line: `extension=memcache`. [Full example](https://github.com/nono303/PHP-memcache-dll/blob/master/memcache.ini)
* Restart Apache

## Configuration

Insert the settings into your configuration file, e.g. `config/settings.php`;

```php
$settings['memcache'] = [
    'host' => '127.0.0.1',
    'port' => '11211',
];
```

## DI Container Setup

Add a DI container definition for `Memcache:class` in `config/container.php`.

```php
<?php

use Memcache;
use Psr\Container\ContainerInterface;
use RuntimeException;
// ...

return [
    // ...

    Memcache::class => function (ContainerInterface $container) {
        $settings = $container->get('settings')['memcache'];

        $memcache = new Memcache();
        
        if (!$memcache->connect($settings['host'], $settings['port'])) {
            throw new RuntimeException('Could not connect to Memcache server');
        }

        return $memcache;
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

The first example shows how to cache data that expires after 10 seconds:

```php
$object = new stdClass();
$object->message = 'test';
$object->number = 123;

$memcache->set('key', $object, false, 10);
```

To access the `Memcache` object, we must first declare it in the class constructor so that it can 
be automatically injected by the DI container.

The following example shows a very simple page cache (without [cache stampede](https://en.wikipedia.org/wiki/Cache_stampede) prevention).

```php
<?php

namespace App\Action\Example;

use Memcache;
use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface;

final class ExampleAction
{
    private Memcache $cache;

    public function __construct(Memcache $cache)
    {
        $this->cache = $cache;
    }

    public function __invoke(
        ServerRequestInterface $request,
        ResponseInterface $response
    ): ResponseInterface {
        $value = $this->cache->get('my_cache_key');

        if (!$value) {
            // ... do some heavy computations
            $value = 'foobar-' . date('H:i:s');

            // Save computed value, expire after 10 seconds
            $this->cache->set('my_cache_key', $value, false, 10);
        }

        $response->getBody()->write($value);

        return $response;
    }
}

```

### Cache Invalidation

Delete item from the server:

```php
$this->cache->delete('key_to_delete');
```

Flush all existing items at the server:

```php
$this->cache->flush();
```

Please note that after flushing, you have to wait a certain amount of time (in my case < 1s)
to be able to write to Memcached again. If you don't, `Memcached::set()` will return 1,
although your data is in fact not saved.

## Conclusion

Getting Memcached and the PECL extension up and running requires quite some time.
In my opinion, the setup and the many technical limitations are not really worth the effort.
Anyway, Memcached is still very fast and works well for storing results of heavy database queries, 
API calls or as a page cache.

## Read more

* [memcached.org](http://memcached.org/)
* [Memcache packages](https://packagist.org/?query=memcache)
* [Memcache on php.net](https://www.php.net/manual/en/book.memcache.php)