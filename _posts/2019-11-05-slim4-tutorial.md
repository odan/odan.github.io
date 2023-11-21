---
title: Slim 4 - Tutorial
layout: post
comments: true
published: true
description:
keywords: php, slim, tutorial, slim-framework, slim-4, getting started
image: https://odan.github.io/assets/images/slim-logo-600x330.png
---

This tutorial shows you how to work with the powerful and lightweight Slim 4 framework.

![slim](https://user-images.githubusercontent.com/781074/82730649-87608800-9d01-11ea-83ea-6112f973b051.png)

**[You can buy all Slim articles as eBook](../../../donate.html).**

## Table of contents

* [Requirements](#requirements)
* [Introduction](#introduction)
* [Installation](#installation)
* [Directory Structure](#directory-structure)
* [Autoloader](#autoloader)
* [Configuration](#configuration)
* [DI Container](#di-container)
* [Bootstrap](#bootstrap)
* [Front Controller](#front-controller)
* [Middleware](#middleware)
* [Routes](#routes)
* [Single Action Controller](#single-action-controller)
* [Creating a JSON Response](#creating-a-json-response)
* [Conclusion](#conclusion)
* [Support](#support)

## Requirements

* PHP 8
* Composer

## Introduction

Welcome to this Slim 4 Tutorial!

In this guide, we will explore the fundamentals of Slim 4, 
a lightweight and powerful PHP micro-framework for building
efficient and scalable web applications and RESTful APIs.

## Installation

To set up your Slim 4 project, follow these steps:

Create a new project directory.

Install the Slim main package using Composer:

```
composer require slim/slim
```

Slim 4 has decoupled most implementations from the App core. 

Next, we need packages to manage HTTP requests and responses within the application. 
The Nyholm PSR-7 packages offer the fastest and strictest implementation of the PSR-7 
standard for HTTP messages.

Install the packages, run:

```
composer require nyholm/psr7
composer require nyholm/psr7-server
```

Ok nice, now we have installed the most basic dependencies for our project.
Later we will add more dependencies to the project.

Please keep in mind not to include the `vendor/` directory in your Git repository. 
To avoid this, create a `.gitignore` file in your project root 
directory and add the following lines:

```
vendor/
.idea/
.env
```

## Directory Structure

A well-organized directory structure is essential for code organization,
and enhances the overall application security.

In a web application, it is important to differentiate between the **public** and
**non-public** areas.

The `public/` is directly accessible from the web by all browsers and API clients.

This means, that the `public/` folder, should be defined the 
[DocumentRoot](https://httpd.apache.org/docs/2.4/en/mod/core.html#documentroot)
for your site.

All other directories (within the project root-directory)
should remain **non-public** and must not be accessible from the web.

Create a new directory: `public/`

In the next steps of this tutorial we will create a project directory structure
that will look like this:

```
.
├── config/             Configuration files
├── public/             Web server files (DocumentRoot)
│   └── index.php       The front controller
├── src/                PHP source code (The App namespace)
├── vendor/             Reserved for composer
├── .gitignore          Git ignore rules
└── composer.json       Project dependencies
```

## Autoloader

To manage the loading of PHP classes in our project, 
we need a [PSR-4 Autoloader](https://www.php-fig.org/psr/psr-4/).

PSR-4 defines a standard for class naming and directory structure, 
making it easier to organize and autoload classes without the need for manual `require` statements.

Create a new directory: `src/`

Add the following `autoload` section to your `composer.json` file:

```json
"autoload": {
  "psr-4": {
    "App\\": "src/"
  }
}
```

With this configuration, your application will efficiently
autoload all classes within the `\App` namespace from the `src/` directory.

Here's how your complete `composer.json` file should look now:

```json
{
  "require": {
    "nyholm/psr7": "^1",
    "nyholm/psr7-server": "^1",
    "slim/slim": "^4"
  },
  "autoload": {
    "psr-4": {
      "App\\": "src/"
    }
  }
}

```

Run `composer update` for the changes to take effect.

## Configuration

Let's set up the configuration files.

The directory for all configuration files is: `config/`

Create a directory `config/` in your project.

Inside the `config/` directory, create a file named `settings.php`.

Copy and paste the following content into `config/settings.php`:

```php
<?php

// Should be set to 0 in production
error_reporting(E_ALL);

// Should be set to '0' in production
ini_set('display_errors', '1');

// Settings
$settings = [];

// ...

return $settings;
```

This `settings.php` file serves as the main configuration file, 
merging default settings with environment-specific ones. 

You can further populate the `$settings` array with your project's 
specific configuration options.

## DI Container

[Dependency injection](https://en.wikipedia.org/wiki/Dependency_injection) is a programming 
technique that separates object creation from their use, 
aiming to create loosely coupled programs.

Without dependency injection, classes often hard-code their dependencies.

Dependency injection separates concerns by ensuring that objects receive their 
required dependencies from "outside". Constructor injection is the most common form, 
where dependencies are injected through a constructor.

**Example**

```php

final class Logger
{
    public function log(string $message): void
    {
        echo "Logging: " . $message . "\n";
    }
}

final class UserCreator
{
    private Logger $logger;

    public function __construct(Logger $logger)
    {
        $this->logger = $logger;
    }
    
    public function createUser(string $username): void
    {
        // ...
        $this->logger->log("User created: " . $username);
    }
}

// Usage
$logger = new Logger();
$userCreator = new UserCreator($logger);

$userCreator->createUser('john.doe');
```

Manual construction of dependencies can become complex, 
while a Dependency Injection (DI) container like [PHP-DI](http://php-di.org/) 
can automate this process, see [Autowiring](https://php-di.org/doc/autowiring.html).

To install PHP-DI, run:

```
composer require php-di/php-di
```

### DI Container Definitions

Create a new file for the DI container entries `config/container.php`
and copy/paste this content:

```php
<?php

use Psr\Container\ContainerInterface;
use Slim\App;
use Slim\Factory\AppFactory;

return [
    'settings' => function () {
        return require __DIR__ . '/settings.php';
    },

    App::class => function (ContainerInterface $container) {
        $app = AppFactory::createFromContainer($container);

        // Register routes
        (require __DIR__ . '/routes.php')($app);

        // Register middleware
        (require __DIR__ . '/middleware.php')($app);

        return $app;
    },
];
```

Note that we use the DI container to build the Slim App instance and to load the
application settings. This allows us to configure the infrastructure services,
like the database connection, mailer etc. within the DI container.
All settings are just passed as a simple array, so we have no special class identifier (FQCN) here.

The `App::class` identifier is needed to ensure that we use the same App object 
across the application and to ensure that the App object uses the same DI container object.

Make sure you add the required `use` statements at the top of the PHP 
file to ensure that the correct namespaces are defined.

## Bootstrap

The app startup process contains the code that is executed when the
application (request) is started.

The bootstrap procedure includes the composer autoloader and then continues to
build the DI container, creates the app and registers the routes + middleware entries.

Create the bootstrap file `config/bootstrap.php` and copy/paste this content:

```php
<?php

use DI\ContainerBuilder;
use Slim\App;

require_once __DIR__ . '/../vendor/autoload.php';

// Build DI container instance
$container = (new ContainerBuilder())
    ->addDefinitions(__DIR__ . '/container.php')
    ->build();

// Create App instance
return $container->get(App::class);

```

## Front Controller

The [front controller](https://en.wikipedia.org/wiki/Front_controller) is the entry point
to your slim application and handles all requests by channeling
requests through a single handler object.

For security reasons you should always place your front-controller (index.php) into the `public/`
directory. You should never place the front controller directly into the project root directory.

Create the front-controller file `public/index.php` and copy/paste this content:

```php
<?php

(require __DIR__ . '/../config/bootstrap.php')->run();
```

**Note:** The `public/` directory is only the `DoumentRoot` of your webserver,
but it's never part of your base path and the official url.

<span style="color:green">Good URLs:</span>

* `https://www.example.com`
* `https://www.example.com/users`
* `https://www.example.com/my-app`
* `https://www.example.com/my-app/users`

<span style="color:red">Bad URLs:</span>

* `https://www.example.com/public`
* `https://www.example.com/public/users`
* `https://www.example.com/public/index.php`
* `https://www.example.com/my-app/public`
* `https://www.example.com/my-app/public/users`

## Middleware

A [middleware](https://www.slimframework.com/docs/v4/concepts/middleware.html) 
can be executed before and after your Slim application
to read or manipulate the request and response object.

To get Slim running we need to add the Slim `RoutingMiddleware` and `ErrorMiddleware`.

The `RoutingMiddleware` will route and dispatch the incoming requests
and the `ErrorMiddleware` is able to catch all exceptions to render a pretty error page.

The `BodyParsingMiddleware` is optional, but recommend if you work with JSON or form data.

Create a file `config/middleware.php` to set up the global middlewares
and copy/paste this content:

```php
<?php

use Slim\App;

return function (App $app) {
    // Parse json, form data and xml
    $app->addBodyParsingMiddleware();

    // Add the Slim built-in routing middleware
    $app->addRoutingMiddleware();

    // Handle exceptions
    $app->addErrorMiddleware(true, true, true);
};

```

## Routes

A "route" is a URL path that can be mapped to a specific handler.
Such a handler can be a simple function or an invokable class.
Under the hood Slim uses the `nikic/FastRoute` package, but it
also adds some nice features for routing groups, names and middlewares etc.

The application routes will be defined in plain PHP files.

Create a file `config/routes.php` and copy/paste this content:

```php
<?php

use Slim\App;

return function (App $app) {
    // empty
};
```

**The first route**

Open the file `config/routes.php` and insert the code for the first route:

```php
<?php

use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface;
use Slim\App;

return function (App $app) {
    $app->get('/', function (ServerRequestInterface $request, ResponseInterface $response) {
        $response->getBody()->write('Hello, World!');

        return $response;
    });
};

```

Open a new console and start the [built-in development server](https://www.php.net/manual/en/features.commandline.webserver.php)
using this command:

```
php -S localhost:8080 -t public/
```

The terminal will show:

```
[Sat Aug  6 10:38:50 2022] PHP 8.x Development Server (http://localhost:8080) started
```

**Note:** This web server is designed to aid application development. 
It may also be useful for testing purposes or for application 
demonstrations that are run in controlled environments. 
It is not intended to be a full-featured web server, and 
it should not be used in a public network.

The parameter `-S localhost:8080` defines the server hostname and port.
The parameter `-t public/` specifies the document root directory with the index.php file.

Now open your website, e.g. <http://localhost:8080> and you should see the message: `Hello, World!`

To simplify this step you may add a script command `start` 
and the config key `process-timeout` into your composer.json file:

```json
"config": {
  "process-timeout": 0
},
"scripts": {
  "start": "php -S localhost:8080 -t public/",
}
```

To stop the built-in web server, press: `Ctrl+C`.

To start the built-in web server, you can now use this command:

```
composer start
```

## Single Action Controller

Slim provides some methods for adding controller logic directly in a route callback.
The request/response object (and optionally the route arguments) are passed by Slim 
to a callback function as follows:

```php
<?php

use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface;

// ...

$app->get('/', function (ServerRequestInterface $request, ResponseInterface $response) {
    $response->getBody()->write('Hello, World!');
    
    return $response;
});
```

Such interface may look intuitive, but it is not suitable for complexer scenarios.

Anonymous functions as routing handlers are quite "expensive", 
because PHP has to create all functions for each request.
The use of class names is more lightweight, faster and scales better for larger applications.
Unless your logic is very simple, I don't recommend using callback functions.
This is the moment where a **Single Action Controller** come into play.

Each Single Action Controller is represented by its own class and has only **one public method**.

The *Action* does only these things:

* Collects input from the HTTP request (if needed)
* Invokes the **Domain** with those inputs (if required) and retains the result
* Builds an HTTP response (typically with the Domain invocation results).

All other logic, including all forms of input validation, error handling, and so on,
are therefore pushed out of the Action and into the **Domain**
(for domain logic concerns) or the response renderer (for presentation logic concerns).

A response could be rendered to HTML for a standard web request; or
it might be something like JSON for a RESTful API.

Create a sub-directory: `src/Action`

Create this action class in: `src/Action/HomeAction.php`

```php
<?php

namespace App\Action;

use Psr\Http\Message\ResponseInterface as Response;
use Psr\Http\Message\ServerRequestInterface as Request;

final class HomeAction
{
    public function __invoke(Request $request, Response $response): Response
    {
        $response->getBody()->write('Hello, World!');

        return $response;
    }
}
```

Then open `config/routes.php` and replace the route closure for `/` with this line:

```php
$app->get('/', \App\Action\HomeAction::class);
```

The complete `config/routes.php` should look like this now:

```php
<?php

use Slim\App;

return function (App $app) {
    $app->get('/', \App\Action\HomeAction::class);
};
```

Now open the URL, e.g. <http://localhost:8080> and you should see the message `Hello, World!`.

### Creating a JSON Response

To create a valid JSON response you can write the json encoded string to the response body
and set the `Content-Type` header to `application/json`:

```php
<?php

namespace App\Action;

use Psr\Http\Message\ResponseInterface as Response;
use Psr\Http\Message\ServerRequestInterface as Request;

final class HomeAction
{
    public function __invoke(Request $request, Response $response): Response
    {
        $response->getBody()->write(json_encode(['hello' => 'world']));

        return $response->withHeader('Content-Type', 'application/json');
    }
}
```

Open your website, e.g. <http://localhost:8080> and you should see the JSON response `{"hello":"world"}`.

## Conclusion

Remember the relationships:

* Slim - To handle routing and dispatching
* Single Action Controllers - Request and response handling.

Slim follows the Unix philosophy in regard to software
selection and programming as it applies to the modern era,
select and write programs that do one thing and do it well.

Slim provides a solid foundation for packages
that can be installed as needed.
Thanks to the PSR interfaces, composer and the DI container,
packages can be replaced quite easily. Updates can
be done in smaller steps without risking too much.

Slim does not force you into a corset like other major frameworks do.
You are also not forced to use any anti-patterns (e.g. facade or active-record),
and you can build modern, clean code.

How to add a database connection and much more can 
be found in my [Slim 4 eBook](https://ko-fi.com/s/5f182b4b22). 

## Support

The source code with for this tutorial can be found here:

* <https://github.com/odan/slim4-tutorial>

You can post your questions, comments or technical issue here:

* <https://github.com/odan/slim4-tutorial/issues/new>

If you have Slim-Framework specific questions, visit:

* <https://discourse.slimframework.com/>
* <https://www.slimframework.com/docs/v4/>
