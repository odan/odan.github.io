---
title: Slim 4 - Whoops
layout: post
comments: true
published: true
description:
keywords: php, slim, whoops, error
image: https://odan.github.io/assets/images/slim-logo-600x330.png
---

## Table of contents

* [Requirements](#requirements)
* [Introduction](#introduction)
* [Installation](#installation)
* [Usage](#usage)
* [Security note](#security-note)
* [Conclusion](#conclusion)
* [Read more](#read-more)

## Requirements

* PHP 7.4+
* [A Slim 4 application](https://odan.github.io/2019/11/05/slim4-tutorial.html)

## Introduction

> whoops is an error handler framework for PHP. Out-of-the-box, 
> it provides a pretty error interface that helps you debug your web projects, 
> but at heart it's a simple yet powerful stacked error handling system.

## Installation

Use Composer to install Whoops into your project:

```
composer require filp/whoops
```

We also need a [middleware](https://github.com/middlewares/whoops) to use Whoops as error handler.

```
composer require middlewares/whoops
```

Now add the `Middlewares\Whoops` middleware, before the `ErrorMiddleware`, 
into your Slim middleware stack.

```php
<?php

use Middlewares\Whoops;
use Slim\App;
use Slim\Middleware\ErrorMiddleware;

return function (App $app) {
    // ...
    $app->add(Whoops::class);
    $app->add(ErrorMiddleware::class);
};
```

If you are not using the DI container [autowire](https://php-di.org/doc/autowiring.html)
feature, you can also add the middleware manually as follows:

```php
<?php

use Middlewares\Whoops;
use Slim\App;
use Slim\Middleware\ErrorMiddleware;

return function (App $app) {
    // ...
    $app->add(new Whoops());
    $app->addErrorMiddleware(true, true, true);
};
```

## Usage

To see Whoops in action just open a non-existing URL or throw a new Exception.

```
throw new \RuntimeException('This is a test');
```

Now you should get a pretty HTML page with the error message, the details of the HTTP 
request, and the stack trace. When the HTTP client sends an Ajax request, 
a JSON response should be returned.

![image](https://user-images.githubusercontent.com/781074/132087873-0819bcf3-1a3f-4328-9047-251e63085640.png)

## Security note

Whoops displays very detailed data of the web server 
and parts of the source code of your application. 
You should therefore use Whoops only for debugging, but not when running in production.

By using Whoops, you change the behavior of how your Slim app handles exceptions.
This could affect other logging middleware handlers.

## Conclusion

On the one hand it's nice to have such a pretty HTML error page, 
on the other hand there are a lot of other aspects like security, logging, testing,
configuration etc. that will become more complicated to set up and handle.
I would still prefer (and recommend) to use Xdebug for debugging and add a 
separate error page renderer for all other cases.

## Read more

* <https://github.com/filp/whoops>
* <https://github.com/middlewares/whoops>
* [Ask a question or report an issue](https://github.com/odan/slim4-tutorial/issues)
 