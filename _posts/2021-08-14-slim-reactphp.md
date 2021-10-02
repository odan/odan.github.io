---
title: Slim 4 - ReactPHP
layout: post
comments: true
published: true
description:
keywords: php, slim, reactphp
image: https://odan.github.io/assets/images/slim-logo-600x330.png
---

## Table of contents

* [Introduction](#introduction)
* [Installation](#installation)
* [Minimal Slim-ReactPHP Example](#minimal-slim-reactphp-example)
* [Database Connection](#database-connection)  
* [Deploy ReactPHP on PHP host](#deploy-reactphp-on-php-host)
* [Conclusion](#conclusion)
* [Read more](#read-more)

## Introduction

Slim Framework is already an incredibly fast PHP microframework that brings out 
the best of PHP. But after trying 
[Slim in combination with ReactPHP](https://discourse.slimframework.com/t/slim4-micro-services-reactphp-html-server/4914),
I was just blown away by the performance boost.

## Installation

To install ReactPHP, just run:

```
composer require react/http
```

## Minimal Slim-ReactPHP Example

For demo purposes, let me show you how little code you need to implement a minimal Slim/ReactPHP application.

Create a file: `public/server.php`

```php
<?php

require_once __DIR__ . '/../vendor/autoload.php';

use Psr\Http\Message\ResponseInterface as Response;
use Psr\Http\Message\ServerRequestInterface as Request;
use React\EventLoop\Loop;
use React\Http\Server as HttpServer;
use React\Socket\SocketServer;
use Slim\Factory\AppFactory;

$app = AppFactory::create();

$app->get(
    '/',
    function (Request $request, Response $response) {
        $response->getBody()->write("Hello World!" . date('Y-m-d H:i:s'));
        return $response;
    }
);

$loop = Loop::get();

$server = new HttpServer(
    function (Request $request) use ($app) {
        return $app->handle($request);
    }
);

$socket = new SocketServer('127.0.0.1:8080');

$server->listen($socket);

$loop->run();
```

This simple web server, written in ReactPHP, responds with "Hello World!" for every request.

To start the Server, run:

```
php server.php
```

Now open the URL `http://127.0.0.1:8080` in your browser,
and you should see a Hello world message and the current time.

## Performance Testing

If you press F12 to open the Developer toolbar, 
you can see the response time is quite good, around 7 ms or less.

Now I want to see how it will perform under heavy load.
For this purpose I use the [ApacheBench](http://httpd.apache.org/docs/2.4/programs/ab.html)
tool to make HTTP performance tests. You can also test nginx or any 
other webserver with ApacheBench.

To simulate 1000 request with a concurrency level of 100 (number of requests at a time)
I use this command:

```
ab -n 1000 -c 100 -k http://127.0.0.1:8080/
```

*Justify the URL to your needs.*

Here are the results:

```
Server Software:        ReactPHP/1

Concurrency Level:      100
Time taken for tests:   0.245 seconds
Complete requests:      1000
Failed requests:        0
Keep-Alive requests:    1000
Total transferred:      151000 bytes
HTML transferred:       31000 bytes
Requests per second:    4081.65 [#/sec] (mean)
Time per request:       24.500 [ms] (mean)
Time per request:       0.245 [ms] (mean, across all concurrent requests)
Transfer rate:          601.88 [Kbytes/sec] received
```

Then I tested the same "Hello World" code without ReactPHP and Apache and `mod_php`. 
The result looks like this:

```
Server Software:        Apache/2.4.39

Concurrency Level:      100
Time taken for tests:   4.183 seconds
Complete requests:      1000
Failed requests:        0
Keep-Alive requests:    1000
Total transferred:      283100 bytes
HTML transferred:       31000 bytes
Requests per second:    239.07 [#/sec] (mean)
Time per request:       418.280 [ms] (mean)
Time per request:       4.183 [ms] (mean, across all concurrent requests)
Transfer rate:          66.10 [Kbytes/sec] received
```

**Update:** *The results has been fixed after disabling the XDebug extension.*

You can see that the difference is quite big.
Slim with ReactPHP performs about 17 times faster than with Apache.

Let's see how Nginx with PHP and FastCGI performs:

```
Server Software:        nginx/1.19.7

Concurrency Level:      100
Time taken for tests:   13.907 seconds
Complete requests:      1000
Failed requests:        0
Keep-Alive requests:    0
Total transferred:      84000 bytes
HTML transferred:       15500 bytes
Requests per second:    35.95 [#/sec] (mean)
Time per request:       2781.304 [ms] (mean)
Time per request:       27.813 [ms] (mean, across all concurrent requests)
Transfer rate:          5.90 [Kbytes/sec] received
```

The result was that Nginx with PHP (FastCGI) is about 56 times slower than native ReactPHP.
The problem with Nginx was that (on my machine) it works very unstable under
heavy load. Most of the time about 50% of all requests are failing
because the PHP FastCGI process just crashes under heavy load. Maybe my local 
setup was just not the best. I'm sorry, but for the moment, I could not 
invest more time to build a more stable Nginx+PHP server. 

```
502 Bad Gateway
nginx/1.19.7
```

```
Server Software:        nginx/1.19.7
...
Complete requests:      1000
Failed requests:        501
   (Connect: 0, Receive: 0, Length: 501, Exceptions: 0)
```

## Database Connection

To access MySQL asynchronously, you need an asynchronous MySQL database
client for ReactPHP like [react/mysql](https://github.com/friends-of-reactphp/mysql).
This implements the MySQL protocol and allows you to access your existing MySQL database.
It is written in pure PHP and does not require any extensions.

This example runs a simple `SELECT` query and dumps all the records from a `users` table:

```php
use React\MySQL\Factory;
use React\MySQL\QueryResult;

$factory = new Factory();

$uri = 'test:test@127.0.0.1/test';
$connection = $factory->createLazyConnection($uri);

$connection->query('SELECT * FROM users')->then(
    function (QueryResult $command) {
        print_r($command->resultFields);
        print_r($command->resultRows);
        echo count($command->resultRows) . ' row(s) in set' . PHP_EOL;
    },
    function (Exception $error) {
        echo 'Error: ' . $error->getMessage() . PHP_EOL;
    }
);

$connection->quit();
```

Please note, that special features, such as database [transactions](https://github.com/friends-of-reactphp/mysql/issues/111)
are not supported yet.

## Deploying ReactPHP

When you try to create a chat server in ReactPHP you have to start the listener
on a specific port:

```php
$socket = new React\Socket\Server(8080, $loop);
$server->listen($socket);
```

If you run this on a shared hosting you won't be able to host this. 
You need your own server, VPS, Docker container or bare metal, to run ReactPHP as a server because 
you're dealing with a daemon process. Shared hosting generally doesn't support that.

The "official" suggestion is to get a VPS / Docker container somewhere and 
look into Supervisor to keep your process running and restart it when something
happens to it. This also requires you to manage your own server with all the 
firewalling and networking knowledge that comes with it.

## Conclusion

The performance boost is just enormous which makes it very interesting
for high-performance real-time applications. 

However, when tackling heavy computing tasks, performance can decrease quite significantly.
The asynchronous programming model makes it difficult to maintain code.
You should be familiar with firewalls, networks and all other tasks related to IT operations (DevOps).
The lack of component / library support can endanger your code.

## Read more

* [ReactPHP website](https://reactphp.org/)
* [ReactPHP on Github](https://github.com/reactphp/reactphp)