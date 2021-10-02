---
title: Slim 4 - Sentry
layout: post
comments: true
published: true
description: 
keywords: php, sentry, error, logging, slim-framework
---

## Table of contents

* [Requirements](#requirements)
* [Introduction](#introduction)
* [Installation](#installation)
* [Configuration](#configuration)
* [Usage](#usage)

## Requirements

* PHP 7.2+
* [A Slim 4 application](https://odan.github.io/2019/11/05/slim4-tutorial.html)
* A [Sentry account](https://sentry.io/signup/)

## Introduction

What is [Sentry](https://sentry.io/)? Sentry is an error tracking and monitoring tool that 
aggregates errors across your stack in real time. 

## Installation

Install the Sentry PHP SDK:

```
composer require sentry/sdk:2.1.0
```

## Configuration

Add the following settings to your Slim settings array, e.g `config/settings.php`:

```php
$settings['sentry'] = [
    'dsn' => 'https://<key>@<organization>.ingest.sentry.io/<project>',
];
```

**Please note:** For security reasons you should keep the secret DSN out of version control.

## Middleware

Create a new `SentryMiddleware` in `src/Middleware/SentryMiddleware.php`:

```php
<?php

namespace App\Middleware;

use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface;
use Psr\Http\Server\MiddlewareInterface;
use Psr\Http\Server\RequestHandlerInterface;
use Throwable;

final class SentryMiddleware implements MiddlewareInterface
{
    /**
     * @var array
     */
    private $options;

    /**
     * The constructor.
     *
     * @param array $options The sentry options
     */
    public function __construct(array $options)
    {
        $this->options = $options;
    }

    /**
     * Invoke middleware.
     *
     * @param ServerRequestInterface $request The request
     * @param RequestHandlerInterface $handler The handler
     *
     * @throws Throwable
     *
     * @return ResponseInterface The response
     */
    public function process(
        ServerRequestInterface $request,
        RequestHandlerInterface $handler
    ): ResponseInterface {
        try {
            \Sentry\init($this->options);

            return $handler->handle($request);
        } catch (Throwable $exception) {
            \Sentry\captureException($exception);

            throw $exception;
        }
    }
}
```

To capture all errors, even the one during the startup of your application, 
you should initialize the Sentry PHP SDK as soon as possible.

Add the `SentryMiddleware` before the Slim error middleware, e.g. in `config/middleware.php`:

```php
<?php

use App\Middleware\SentryMiddleware;
use Slim\App;
use Slim\Middleware\ErrorMiddleware;

return function (App $app) {
    // ...

    $app->add(SentryMiddleware::class); // <-- here

    $app->add(ErrorMiddleware::class);
};

```

However, of course any errors in bootstrap etc aren't handled, so for completion one needs 
to also add the `Sentry\init()` call just after the container build if you wanted to 
monitor that part of the app.

```php
// Build PHP-DI Container instance
$container = $containerBuilder->build();

Sentry\init($container->get('settings')['sentry']);
```

### Container setup

Add a container definition for `\App\Middleware\SentryMiddleware:class` in `config/container.php`:

```php
<?php

use App\Middleware\SentryMiddleware;
use Psr\Container\ContainerInterface;
// ...

return [

    // ...

    SentryMiddleware::class => function (ContainerInterface $container) {
        return new SentryMiddleware($container->get('settings')['sentry']);
    },
];

```

## Usage

Once you have Sentry integrated into your project, you probably want to verify that 
everything is working as expected before deploying it.

One way to verify your setup is by intentionally sending an event that breaks your application.

You can throw an exception in your PHP application:

```php
throw new Exception('My first Sentry error!');
```

Then login to your Sentry account and you should see all details of the error:

![image](https://user-images.githubusercontent.com/781074/86631970-159d7e80-bfcf-11ea-8eed-5b294b9760c4.png)

Now that you’ve got basic reporting set up, you’ll want to explore adding additional context to your data.

**Read more:** [Next Steps](https://docs.sentry.io/error-reporting/quickstart/?platform=php#next-steps)

## Read more

* [Ask a question or report an issue](https://github.com/odan/slim4-tutorial/issues)
