---
title: Slim 4 - Performance Testing using Apache Bench
layout: post
comments: true
published: true
description:
keywords: php slim apache performance benchmark
---

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

## Read more

* <https://stackoverflow.com/questions/9356462/apachebench-is-very-slow>
* <https://serverfault.com/questions/254765/apache-benchmark-keep-alive>
* <http://help.slimframework.com/discussions/problems/24-high-loading-times-in-apache-bench/page/1>

