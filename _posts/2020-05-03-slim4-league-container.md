---
title: Slim 4 - League Container
layout: post
comments: true
published: false
description:
keywords: php, slim, container, psr11, psr-11, slim-framework
---

## Table of contents

* [Requirements](#requirements)
* [Introduction](#introduction)
* [Installation](#installation)
* [Configuration](#configuration)
* [Bootstrapping](#bootstrapping)

## Requirements

* PHP 7.2+
* Composer
* [A Slim 4 application](https://odan.github.io/2019/11/05/slim4-tutorial.html)

## Introduction

The League Container is a small but powerful dependency injection container <http://container.thephpleague.com>

## Installation

To install League Container, run:

```
composer require league/container
```

## Configuration

Add the database settings to Slim’s settings array, e.g `config/container.php`:

```php
<?php

use Monolog\Logger;
use Psr\Container\ContainerInterface;
use Psr\Log\LoggerInterface;

return [
    'settings' => function () {
        return require __DIR__ . '/settings.php';
    },

    LoggerInterface::class => function (ContainerInterface $container) {
        $logger = new Logger('name');
        
        // ...
        
        return $logger;
    },
    
    // Add more definitions here...
}
```

### Bootstrapping

In your  `config/bootstrap.php` or wherever you have your boostrap code:

```php
<?php

use League\Container\Container;
use League\Container\ReflectionContainer;
use Slim\App;
use Symfony\Component\Translation\Translator;

require_once __DIR__ . '/../vendor/autoload.php';

$container = new Container();
$container->defaultToShared(true);

// Register the reflection container as a delegate to enable auto wiring
$container->delegate(new ReflectionContainer());

// Add container definitions (closures)
foreach (require __DIR__ . '/container.php' as $key => $factory) {
    $container->add($key, $factory)->addArgument($container);
}

// Create slim app instance
AppFactory::setContainer($container);
$app = AppFactory::create();

// Add routes, middleware etc...

$app->run();
```

## Known issues

* A closure factory doesn't get the container automatically. You have to pass it manually with `addArgument`.
* Constructor parameters with optional / nullable values are not supported.
* Compared to PHP-DI, the performance of League Container can be up to 12 slower.
