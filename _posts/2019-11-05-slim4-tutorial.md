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
* [Directory structure](#directory-structure)
* [Apache URL rewriting](#apache-url-rewriting)
* [Your first route](#your-first-route)
* [Actions](#actions)
* [Domain](#domain)
* [Repositories](#repositories)
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

As next we need a PSR-11 container implementtion for **dependency injection** and **autowiring**.

Run this command to install [PHP-DI](http://php-di.org/):

```
composer require php-di/php-di
```

For testing purpose we are installing [phpunit](https://phpunit.de/) as development dependency with the `--dev` option:

```
composer require phpunit/phpunit --dev
```

Ok nice, now we have installed the most basic dependencies for our project. Later we will add more.

**Note:** Please don't commit the `vendor/` to your git repository. To set up the git repository correctly, create a file called `.gitignore` in the project root folder and add the following lines to this file:

```
vendor/
.idea/
```

## Directory structure

A good directory structure helps you organize your code, simplifies setup on the web server and increases the security of the entire application.

Create the following directory structure in the root directory of your project:

```
.
├── config/             Configuration files
├── public/             Web server files (DocumentRoot)
│   └── .htaccess       Apache redirect rules for the front controller
│   └── index.php       The front controller
├── templates/          Twig templates
├── src/                PHP source code (The App namespace)
├── tmp/                Temporary files (cache and logfiles)
├── vendor/             Reserved for composer
├── .htaccess           Internal redirect to the public/ directory
└── .gitignore          Git ignore rules
```

In a web application, it is important to distinguish between the public and 
non-public areas.

The `public/` directory serves your application and will therefore also be 
directly accessible by all browsers, search engines and API clients. 
All other folders are not public and must not be accessible online. 
This can be done by defining the `public` folder in Apache as `DocumentRoot` 
of your website. But more about that later.


## Apache URL rewriting

To run a Slim app with apache we have to add url rewrite rules to redirect the web traffic to a so called [front controller](https://en.wikipedia.org/wiki/Front_controller).

The front controller is just a `index.php` file and the entry point to the application.

* Create a directory: `public/`
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

The [front controller](https://en.wikipedia.org/wiki/Front_controller) is the entry point 
to your slim application and handles all requests by channeling requests through a single handler object.

### Configuration

The entire configuration of your application is stored and managed in 
the `config/` directory.

* Create a directory: `config/`
* Create a configuration file `config/settings.php` and copy/paste this content:

```php
<?php

// Error reporting
error_reporting(0);
ini_set('display_errors', '0');

// Timezone
date_default_timezone_set('Europe/Berlin');

// Settings
$settings = [];

// Path settings
$settings['root'] = dirname(__DIR__);
$settings['temp'] = $settings['root'] . '/tmp';
$settings['public'] = $settings['root'] . '/public';

return $settings;
```

#### Boostrapping

Boostrapping is the first code that is executed when the application (request) is started. 

The bootstrap procedure starts with the composer autoloader and then continues to
build the container, create the app and register the routes + middleware entries.

Create the bootstrap file `config/bootstrap.php` and copy/paste this content:

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

#### Routing setup

Create a file for all routes `config/routes.php` and copy/paste this content:

```php
<?php

use Slim\App;

return static function (App $app) {
    // empty
};

```

#### Middeware setup

Create a file to load global middleware handler `config/middleware.php` and copy/paste this content:

```php
<?php

use Slim\App;

return static function (App $app) {
    // Parse json, form data and xml
    $app->addBodyParsingMiddleware();

    // Add global middleware to app
    $app->addRoutingMiddleware();

    // Error handler
    $displayErrorDetails = true;
    $logErrors = true;
    $logErrorDetails = true;

    $app->addErrorMiddleware($displayErrorDetails, $logErrors, $logErrorDetails);
};
```

#### Container setup

Slim 4 uses a dependency injection container to prepare, manage and inject application dependencies. 

You can add any container library that implements the [PSR-11](https://www.php-fig.org/psr/psr-11/) interface.

Create a new file for the container entries `config/container.php` and copy/paste this content:

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

### A quick guide to the container

**[Dependency injection](https://en.wikipedia.org/wiki/Dependency_injection)** is passing dependency to other objects.
Dependency injection makes testing easier. The injection can be done through a constructor.

A **dependencies injection container** (DIC) is a tool to help injecting dependencies.

**A general rule:** The application itself should not use the container.
Injecting the container into a class is an **anti-pattern**. Please declare all class dependencies in your 
constructor explicitly instead. 

Why is injecting the container (in the most cases) an anti-pattern?

In Slim 3 the [Service Locator](https://blog.ploeh.dk/2010/02/03/ServiceLocatorisanAnti-Pattern/) (anti-pattern) was the
default "style" to inject the whole ([Pimple](https://pimple.symfony.com/)) container and fetch the dependencies from it. 
However, there are the following disadvantages:

* The Service Locator (anti-pattern) **hides the actual dependencies** of your class. 
* The Service Locator (anti-pattern) also violates the [Inversion of Control](https://en.wikipedia.org/wiki/Inversion_of_control) (IoC) principle of [SOLID](https://en.wikipedia.org/wiki/SOLID).

Q: How can I make it better? 

A: Use [composition over inheritance](https://en.wikipedia.org/wiki/Composition_over_inheritance) 
and (explicit) **constructor dependency injection**. 

Dependency injection is a programming practice of passing into an object it’s collaborators, 
rather the object itself creating them. 

Since **Slim 4** you can use modern DIC like `PHP-DI` and `league/container` with the awesome 
[autowire](http://php-di.org/doc/autowiring.html) feature. 
This means: Now you can declare all dependencies explicitly in your constructor and let the DIC inject these 
dependencies for you. 

To be more clear: "Composition" has nothing to do with the "Autowire" feature of the DIC. You can use composition 
with pure classes and without a container or anything else. The autowire feature just uses the 
[PHP Reflection](https://www.php.net/manual/en/book.reflection.php) classes to resolve and inject the 
dependencies automatically for you.

## Your first route

To add the first route we have to open the file `config/routes.php` and insert this example route for the home path `/`:

```php
<?php

use Psr\Http\Message\ResponseInterface as Response;
use Psr\Http\Message\ServerRequestInterface as Request;
use Slim\App;

return static function (App $app) {
    $app->get('/', function (Request $request, Response $response) {
        $response->getBody()->write('Hello, World!');

        return $response;
    });
};

```

Now open your website, e.g. http://localhost and you should see a message `Hello, World!`.

To add a second route for the path `/hello/{name}` insert this [closure](https://www.php.net/manual/en/class.closure.php) function into `config/routes.php`:

```php
$app->get('/hello/{name}', function (Request $request, Response $response, $args) {
    $name = $args['name'];

    $response->getBody()->write("Hello, $name");

    return $response;
});
```

Opening the url, e.g. http://localhost/hello/daniel, should display the message `Hello, daniel`.

## PSR-4 autoloading

For the next steps we have to setup the `\App` namespace for the PSR-4 autoloader.

Add this autoloading settings into `composer.json`:

```json
"autoload": {
    "psr-4": {
        "App\\": "src"
    }
},
"autoload-dev": {
    "psr-4": {
        "App\\Test\\": "tests"
    }
}
```

The complete `composer.json` file should look like this:

```json
{
    "require": {
        "php-di/php-di": "^6.0",
        "slim/psr7": "^0.6.0",
        "slim/slim": "^4.3"
    },
    "autoload": {
        "psr-4": {
            "App\\": "src"
        }
    },
    "autoload-dev": {
        "psr-4": {
            "App\\Test\\": "tests"
        }
    },
    "config": {
        "sort-packages": true
    }
}
```

Run `composer update` for the changes to take effect.

## Actions

### Writing JSON to the response

## Domain

## Repositories

## Validation

## Conclusion
