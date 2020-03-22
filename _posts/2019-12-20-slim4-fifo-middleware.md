---
title: Slim 4 - FIFO Middleware Setup
layout: post
comments: true
published: false
description: 
keywords: php slim fifo middleware
---

## Table of contents

* [Requirements](#requirements)
* [Introduction](#introduction)
* [Installation](#installation)
* [Dispatcher](#dispatcher)
* [Container](#container)
* [Middleware](#middleware)

## Requirements

* PHP 7.1+
* MySQL 5.7+
* Composer
* [A Slim 4 application](https://odan.github.io/2019/11/05/slim4-tutorial.html)

## Introduction

You can change the default middleware order from [LIFO](https://github.com/slimphp/Slim/issues/2408)
to FIFO by replacing the default `MiddlewareDispatcherInterface` with [Relay](https://relayphp.com/).

> ⚠️ Warning: This approach is experimental 🧪 and should not be used in production.

## Installation

To add Relay to your application, run:

```
composer require relay/relay
```

## Dispatcher

Add the following dispatcher to: `src/Middleware/FifoMiddlewareDispatcher.php`

```php
<?php

namespace App\Middleware;

use Closure;
use Psr\Container\ContainerInterface;
use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface;
use Psr\Http\Server\MiddlewareInterface;
use Psr\Http\Server\RequestHandlerInterface;
use Relay\Relay;
use Slim\Interfaces\MiddlewareDispatcherInterface;
use UnexpectedValueException;

/**
 * FIFO Middleware Dispatcher
 */
class FifoMiddlewareDispatcher implements MiddlewareDispatcherInterface
{

    /**
     * @var array
     */
    private $queue = [];

    /**
     * @var RequestHandlerInterface
     */
    private $kernel;

    /**
     * @var Closure
     */
    private $resolver;

    /**
     * The constructor.
     *
     * @param ContainerInterface $container The container
     */
    public function __construct(ContainerInterface $container)
    {
        $this->resolver = function ($resolve) use ($container) {
            return is_string($resolve) ? $container->get($resolve) : $resolve;
        };
    }

    /**
     * @inheritDoc
     */
    public function add($middleware): MiddlewareDispatcherInterface
    {
        if ($middleware instanceof MiddlewareInterface) {
            return $this->addMiddleware($middleware);
        }

        if (is_string($middleware) || is_callable($middleware)) {
            $this->queue[] = $middleware;

            return $this;
        }

        throw new UnexpectedValueException(
            'A middleware must be an object/class name referencing an implementation of ' .
            'MiddlewareInterface or a callable with a matching signature.'
        );
    }

    /**
     * @inheritDoc
     */
    public function addMiddleware(MiddlewareInterface $middleware): MiddlewareDispatcherInterface
    {
        $this->queue[] = $middleware;

        return $this;
    }

    /**
     * @inheritDoc
     */
    public function seedMiddlewareStack(RequestHandlerInterface $kernel): void
    {
        $this->kernel = $kernel;
    }

    /**
     * @inheritDoc
     */
    public function handle(ServerRequestInterface $request): ResponseInterface
    {
        // Add the route runner as last entry
        $this->queue[] = function ($request) {
            return $this->kernel->handle($request);
        };

        $relay = new Relay($this->queue, $this->resolver);

        return $relay->handle($request);
    }
}
```

## Container

In your `config/container.php` or wherever you add your container definitions:

```php
<?php

use App\Middleware\FifoMiddlewareDispatcher;
use Slim\App;
use Slim\Factory\AppFactory;
use Slim\Interfaces\MiddlewareDispatcherInterface;

return [
    // ...
    App::class => function (ContainerInterface $container) {
        AppFactory::setContainer($container);
        $app = AppFactory::create(
            null,
            null,
            null,
            null,
            null,
            $container->get(MiddlewareDispatcherInterface::class)
        );
    
        return $app;
    },
    
    MiddlewareDispatcherInterface::class => function (ContainerInterface $container) {
        return new FifoMiddlewareDispatcher($container);
    },
];
```

## Middleware

In your `config/middleware.php` file you have to "reorder" the middleware entries from top to down:

Example:

```php
<?php

use Selective\BasePath\BasePathMiddleware;
use Slim\App;
use Slim\Views\TwigMiddleware;

return function (App $app) {
    $app->addErrorMiddleware(true, true, true);

    //$app->add(BasePathMiddleware::class);

    $app->addRoutingMiddleware();
    $app->addBodyParsingMiddleware();

    // Add more...
    // $app->add(TwigMiddleware::class);
};
```
