---
title: Slim 4 - Performance
layout: post
comments: true
published: true
description:
keywords: php slim apache performance benchmark
---

## Table of contents

* [Benchmarking](#benchmarking)
* [Performance Optimization](#performance-optimization)

## Benchmarking

[Apache Bench](https://httpd.apache.org/docs/2.4/programs/ab.html) (ab) is a great tool 
to test the performance of your API.

The problem is that your first test might give this strange result:

```
$ ab -n 1 'http://localhost/my-slim-app'
> 5.008 seconds
```

Of course the loading time in the browser is extremely fast, but when testing the loading 
time with AB you should not get a loading time < 5 seconds.

Using the switch `-k` (Use Keep-Alive) helps to solve the problem.

```
$ ab -n 1 -k 'http://localhost/my-slim-app'
> 0.100 seconds
```

There are two other options:

Add `Connection: close` to the response, e.g.

```php
$response = $response->withHeader('Connection', 'close');
```

Change the HTTP version to `1.0`, e.g.

```php
$response = $response->withProtocolVersion('1.0');
```

**Read more**

* <https://stackoverflow.com/questions/9356462/apachebench-is-very-slow>
* <https://serverfault.com/questions/254765/apache-benchmark-keep-alive>
* <http://help.slimframework.com/discussions/problems/24-high-loading-times-in-apache-bench/page/1>


## Performance Optimization

One important aspect is to fine-tune the performance of slim apps.
In this article, I have collected some important tips for optimizing
the application performance.

Please note: Most of these tips are only useful for the production environment.

* Use PHP configuration files to benefit from the OPCache.
* Run the latest version of PHP. The latest version of PHP has brought significant improvements in its performance.  
* Run `install --no-dev --optimize-autoloader` to remove all dev dependencies and generate a faster autoloader.
* Disable XDebug. Xdebug can slow everything down to one second per request, or even more.
* Use Caching where it makes sense
  * [HTTP Caching](https://github.com/slimphp/Slim-HttpCache)
  * Storage cache (PSR-6, PSR-16)
  * [Route Cache](https://www.slimframework.com/docs/v4/objects/routing.html#route-expressions-caching)
  * Cache query results
* Load only middleware that is really needed.
* Minify and bundle your assets, e.g. JavaScript, CSS and images.
* Make proper use of database indexes.
* Docker containers are slow on Windows and Mac. Consider a local webserver.
* Run Lighthouse in Chrome DevTools.
