---
title: Slim 4 Tutorial
layout: post
comments: true
published: false
description: 
keywords: php slim tutorial
---

This Tutorial shows everything you need to learn about working with a powerful, lightweight Slim 4 framework.

## Table of contents

* [Requirements](#requirements)
* [Introduction](#introduction)
* [Installation](#installation)
* [Action handlers](#single-action-controllers)
* [Repositories](#repositories)
* [Domain](#domain)
* [Validation](#validation)
* [Conclusion](#conclusion)

## Requirements

* PHP 7.1+
* MySQL 5.7+
* Apache webserver
* Slim Framework 4
* Composer

## Introduction

Slim Framework is a great micro framework for web application, RESTful API's and websites.

Our aim is to create a RESTful API with routing, business logic and database operations.

Standards like [PSR](https://www.php-fig.org/psr/) and best practices are very imported and integrated part of this tutorial.

## Installation

Create a new project directory and run this command to install the Slim 4 core components:

```
composer require slim/slim
```

In Slim 4 the PSR-7 implementation is decoupled from the App core. 
This means you can also install other PSR-7 implementations like [nyholm/psr7](https://github.com/Nyholm/psr7).

In our case we are installing the Slim PSR-7 implementations using this command:

```
composer require slim/psr7
```

As next we need a PSR-11 container implementtion for dependency injection and autowiring.

Run this command to install [PHP-DI](http://php-di.org/):

```
composer require php-di/php-di
```

For testing purpose we are installing [phpunit](https://phpunit.de/) as development dependency with the `--dev` option:

```
composer require phpunit/phpunit --dev
```

Ok nice, now we have installed the most basic dependencies for our project. Later we will add more.

## Apache URL rewriting

To run a Slim app with apache we have to add url rewrite rules to redirect the web traffic to a so called [front controller](https://en.wikipedia.org/wiki/Front_controller).

The front controller is just a `index.php` file and the entry point to the application.

* Create a new directory: `public/`
* Create a `.htaccess` file in your `public/` directory and copy/paste this content:

```htaccess
# Redirect to front controller
RewriteEngine On
# RewriteBase /
RewriteCond %{REQUEST_FILENAME} !-d
RewriteCond %{REQUEST_FILENAME} !-f
RewriteRule ^ index.php [QSA,L]
```

* Create a second `.htaccess` file in your project root-directory and copy/paste this content:

```htaccess
RewriteEngine on
RewriteRule ^$ public/ [L]
RewriteRule (.*) public/$1 [L]
```

* Create the front-controller file `public/index.php` and copy/paste this content:

```php
<?php
(require __DIR__ . '/../config/bootstrap.php')->run();
```

* Create a directory: `config/`

The config directory is the place for the application configuration files.

* Create the bootstrap file `config/bootstrap.php` and copy/paste this content:

```php
<?php

use DI\ContainerBuilder;
use Slim\App;
use Symfony\Component\Translation\Translator;

require_once __DIR__ . '/../vendor/autoload.php';

$containerBuilder = new ContainerBuilder();

// Set up settings
$containerBuilder->addDefinitions(__DIR__ . '/container.php');

// Build PHP-DI Container instance
$container = $containerBuilder->build();

// Create App instance
$app = $container->get(App::class);

// Register routes
(require __DIR__ . '/routes.php')($app);

// Register middleware
(require __DIR__ . '/middleware.php')($app);

return $app;
```

* Create a file for all routes `config/routes.php` and copy/paste this content:

```php
<?php

// empty

```

* Create a file to load global middleware handler `config/middleware.php` and copy/paste this content:

```php
<?php

// empty

```

* Create a file to configure the container entries `config/container.php` and copy/paste this content:

```php
<?php

use Psr\Container\ContainerInterface;
use Slim\App;
use Slim\Factory\AppFactory;

return [
    App::class => static function (ContainerInterface $container) {
        AppFactory::setContainer($container);
        $app = AppFactory::create();

        // Optional: Set the base path to run the app in a subdirectory.
        //$app->setBasePath('/slim4-tutorial');

        return $app;
    },
];

```

## Your first route

To add the first route we have to open the file `config/routes.php` and insert this example route for the home path `/`:

```php
<?php

use Psr\Http\Message\ResponseInterface as Response;
use Psr\Http\Message\ServerRequestInterface as Request;

$app->get('/', static function (Request $request, Response $response) {
    $response->getBody()->write('Hello, World!');

    return $response;
});

$app->get('/hello/{name}', static function (Request $request, Response $response, $args) {
    $name = $args['name'];
    
    $response->getBody()->write("Hello, $name");

    return $response;
});
```

Now open your website, e.g. http://localhost and you should see a message "Hello, World!".

To add a second route add the route to `config/routes.php`:

```php
<?php

$app->get('/hello/{name}', static function (Request $request, Response $response, $args) {
    $name = $args['name'];
    
    $response->getBody()->write("Hello, $name");

    return $response;
});
```

Opening the url, e.g. http://localhost/hello/daniel, should display the message "Hello, daniel".



