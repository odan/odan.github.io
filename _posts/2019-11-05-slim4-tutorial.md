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
* [Apache URL Rewrite Rules](#apache-url-rewrite-rules)
* [Configuration](#configuration)
* [DI Container](#di-container)
* [Bootstrap](#bootstrap)
* [Middleware](#middleware)
* [Routes](#routes)
* [Base Path](#base-path)
* [Actions](#actions)
* [Creating a JSON Response](#creating-a-json-response)
* [Conclusion](#conclusion)

## Requirements

* PHP 7.4+
* MySQL 5.7+ or MariaDB
* Apache webserver with [mod_rewrite](https://httpd.apache.org/docs/2.4/mod/mod_rewrite.html)
  and [.htaccess](https://httpd.apache.org/docs/2.4/howto/htaccess.html)
* Composer (only for development)

## Introduction

Slim Framework is a great microframework for web applications, RESTful API's and websites.

Our aim is to create a RESTful API with routing, business logic and database operations.

Standards like [PSR](https://www.php-fig.org/psr/) and best practices
are very important and integrated part of this tutorial.

## Installation

Create a new project directory and run this command to install the Slim 4 core components:

```
composer require slim/slim:"4.*"
```

In Slim 4 the PSR-7 implementation is decoupled from the App core.
This means you can also install other PSR-7 implementations like [nyholm/psr7](https://github.com/Nyholm/psr7).

In our case we are installing the Slim PSR-7 implementations using this command:

```
composer require slim/psr7
```

Ok nice, now we have installed the most basic dependencies for our project.
Later we will add more dependencies to the project.

**Note:** Please don't commit the `vendor/` to your git repository.
To set up the git repository correctly,
create a file called `.gitignore` in the project root folder and
add the following lines to this file:

```
vendor/
.idea/
```

## Directory Structure

A good directory structure helps you organize your code,
simplifies setup on the webserver and increases the security of the entire application.

In the next steps of this tutorial we will create a project directory structure
that will look like this:

```
.
├── config/             Configuration files
├── public/             Web server files (DocumentRoot)
│   └── .htaccess       Apache redirect rules for the front controller
│   └── index.php       The front controller
├── src/                PHP source code (The App namespace)
├── vendor/             Reserved for composer
├── .htaccess           Internal redirect to the public/ directory
├── .gitignore          Git ignore rules
└── composer.json       Project dependencies and autoloader
```

In a web application, it is important to distinguish between the public and
non-public areas.

Create a new directory: `public/`

The `public/` directory serves your application and will therefore also be
directly accessible by all browsers, search engines and API clients.
All other folders are not public and must not be accessible online.
This can be done by defining the `public` folder in Apache as `DocumentRoot`
of your website. But more about that later.

Create a new directory: `src/`

The `src/` directory contains all the project specific PHP classes and
acts as the root for the `\App` namespace.

## Autoloader

One of the most fundamental and important thing is to have a
working [PSR-4 autoloader](https://www.php-fig.org/psr/psr-4/).
For the next steps we have to define the `src/` directory as root for the `\App` namespace.

Add this autoloader settings into `composer.json`:

```json
"autoload": {
"psr-4": {
"App\\": "src/"
}
},
```

The complete `composer.json` file should look like this:

```json
{
  "require": {
    "php-di/php-di": "^6.0",
    "slim/psr7": "^1",
    "slim/slim": "^4.4"
  },
  "autoload": {
    "psr-4": {
      "App\\": "src/"
    }
  }
}
```

Run `composer update` for the changes to take effect.

## Apache URL Rewrite Rules

This step is optional, and only needed if you plan to develop or host your Slim app on Apache.

To run a Slim app with apache we have to add url rewrite rules to redirect
the web traffic to a so-called [front controller](https://en.wikipedia.org/wiki/Front_controller).

The front controller is just a `index.php` file and the entry point to the application.

Create a `.htaccess` file in your `public/` directory and copy/paste this content:

```htaccess
# Redirect to front controller
RewriteEngine On
# RewriteBase /
RewriteCond %{REQUEST_FILENAME} !-d
RewriteCond %{REQUEST_FILENAME} !-f
RewriteRule ^ index.php [QSA,L]
```

Please **do not** change the `RewriteRule` directive. It must be exactly as shown above.

Note that only one some webhosts, you may need to uncomment the line `# RewriteBase /` to make it work.

Create a second `.htaccess` file in your project root-directory and copy/paste this content:

```htaccess
RewriteEngine on
RewriteRule ^$ public/ [L]
RewriteRule (.*) public/$1 [L]
```

Do not skip this step. This second `.htaccess` file is important to run your Slim
app in a sub-directory and within your development environment.

## Configuration

The directory for all configuration files is: `config/`

Create a directory: `config/`

The file `config/settings.php` is the main configuration file and combines
the default settings with environment specific settings.

Create a configuration file `config/settings.php` and copy/paste this content:

```php
<?php

// Should be set to 0 in production
error_reporting(E_ALL);

// Should be set to '0' in production
ini_set('display_errors', '1');

// Timezone
date_default_timezone_set('Europe/Berlin');

// Settings
$settings = [];

// ...

return $settings;
```

## DI Container

As next, we need a [PSR-11](https://www.php-fig.org/psr/psr-11/)
**dependencies injection container** (DI Container)
implementation for **dependency injection** and **autowiring**.

[Dependency injection](https://en.wikipedia.org/wiki/Dependency_injection) is passing
dependency to other objects. Dependency injection makes testing easier.

[Autowiring](http://php-di.org/doc/autowiring.html) means,
that you can declare all dependencies explicitly
in your class constructor and let the DI Container inject these
dependencies automatically.

My favorite DI Container implementation is [PHP-DI](http://php-di.org/).

To install the PHP-DI package, run:

```
composer require php-di/php-di
```

### DI Container Definitions

Create a new file for the container entries `config/container.php`
and copy/paste this content:

```php
<?php

use Nyholm\Psr7\Factory\Psr17Factory;
use Psr\Container\ContainerInterface;
use Psr\Http\Message\ResponseFactoryInterface;
use Slim\App;
use Slim\Factory\AppFactory;
use Slim\Middleware\ErrorMiddleware;

return [
    'settings' => function () {
        return require __DIR__ . '/settings.php';
    },

    App::class => function (ContainerInterface $container) {
        AppFactory::setContainer($container);

        return AppFactory::create();
    },

    ResponseFactoryInterface::class => function (ContainerInterface $container) {
        return $container->get(Psr17Factory::class);
    },
];
```

Note that we use the DI container to build the Slim App instance and to load the
application settings. This allows us to configure the infrastructure services,
like the database connection mailer etc. within the DI container.

### Bootstrap

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

The [front controller](https://en.wikipedia.org/wiki/Front_controller) is the entry point
to your slim application and handles all requests by channeling
requests through a single handler object.

Create the front-controller file `public/index.php` and copy/paste this content:

```php
<?php

(require __DIR__ . '/../config/bootstrap.php')->run();
```

## Middleware

A middleware can be executed before and after your Slim application
to manipulate the request and response object according to your requirements.

[Read more](https://www.slimframework.com/docs/v4/concepts/middleware.html)

In the first step we need to add the Slim `RoutingMiddleware`, `ErrorMiddleware`
and the `BodyParsingMiddleware` the get the most basic features for a running application.

Create a file to load global middleware handler `config/middleware.php`
and copy/paste this content:

```php
<?php

use Slim\App;
use Slim\Middleware\ErrorMiddleware;

return function (App $app) {
    // Parse json, form data and xml
    $app->addBodyParsingMiddleware();

    // Add the Slim built-in routing middleware
    $app->addRoutingMiddleware();

    // Catch exceptions and errors
    $app->addErrorMiddleware(true, true, true);
};
```

### Routes

Create a file for all routes `config/routes.php` and copy/paste this content:

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
    $app->get('/', function (
        ServerRequestInterface $request,
        ResponseInterface $response
    ) {
        $response->getBody()->write('Hello, World!');

        return $response;
    });
};

```

Now open your website, e.g. `http://localhost` and you should see the message `Hello, World!`.

## Base Path

When you set up your first Slim project, you may get an **404 error (not found)**.

If you run your Slim app in a sub-directory, resp. not directly within the
[DocumentRoot](https://httpd.apache.org/docs/2.4/en/mod/core.html#documentroot)
of your webserver, you must set the "correct" base path.

Ideally the `DoumentRoot` of your production server points directly to the `public/` directory.

In all other cases you have to make sure, that your base path is correct. For example,
the DocumentRoot directory is `/var/www/domain.com/htdocs/`, but the application
is stored under `/var/www/domain.com/htdocs/my-app/`, then you have to set `/my-app` as base path.

To be more precise: In this context "sub-directory" means a sub-directory of the project,
and **not** the `public/` directory. For example when you place your app not directly
under the webservers `DocumentRoot`.

For security reasons you should always place your front-controller (index.php) into the `public/`
directory. Don't place your front controller directly into the project root directory.

You can manually set the base path in Slim using the `setBasePath` method:

```php
$app->setBasePath('/slim4-tutorial');
```

But the problem is, that the basePath can be different for each host (dev, testing, staging, prod etc...).

The [BasePathMiddleware](https://github.com/selective-php/basepath) detects and sets
the base path into the Slim app instance.

To install the BasePathMiddleware, run:

```
composer require selective/basepath
```

Add the following container definition into `config/container.php`:

```php
use Selective\BasePath\BasePathMiddleware;
// ...

return [
    // ...

    BasePathMiddleware::class => function (ContainerInterface $container) {
        return new BasePathMiddleware($container->get(App::class));
    },
];
``` 

Then add the `BasePathMiddleware::class` to the middleware stack in `config/middleware.php`:

```php
<?php

use Selective\BasePath\BasePathMiddleware;
use Slim\App;
use Slim\Middleware\ErrorMiddleware;

return function (App $app) {
    // Parse json, form data and xml
    $app->addBodyParsingMiddleware();

    // Add the Slim built-in routing middleware
    $app->addRoutingMiddleware();

    $app->add(BasePathMiddleware::class); // <--- here

    // Catch exceptions and errors
    $app->add(ErrorMiddleware::class);
};
```

Now that you have installed the `BasePathMiddleware`,
remove this line (if exists): `$app->setBasePath('...');`.

Be careful: The `public/` directory is only the `DoumentRoot` of your webserver,
but it's never part of your base path and the official url.

<span style="color:green">Good URLs:</span>

* `https://www.example.com`
* `https://www.example.com/users`
* `httsp://www.example.com/my-app`
* `https://www.example.com/my-app/users`

<span style="color:red">Bad URLs:</span>

* `https://www.example.com/public`
* `https://www.example.com/public/users`
* `https://www.example.com/public/index.php`
* `https://www.example.com/my-app/public`
* `https://www.example.com/my-app/public/users`

## Actions

Slim provides some methods for adding controller logic directly in a route callback.
The PSR-7 request object is injected into your Slim application routes as the first
argument to the route callback like this:

```php
<?php

use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface;

// ...

$app->get('/', function (ServerRequestInterface $request, ResponseInterface $response) {
    $response->getBody()->write('Hello World');
    return $response;
});
```

While such interfaces look intuitive, they are not suitable for complex business logic scenarios.
Assuming there are tens or even hundreds of route handlers that need to be registered.
Unless your logic is very simple, I don't recommend using route callbacks.
Isn't it a better practice to implement these handlers in their own classes? Yes.
This is the moment where a **Single Action Controller** come into play.

Each **Single Action Controller** is represented by its own class.

The *Action* does only these things:

* Collects input from the HTTP request (if needed)
* Invokes the **Domain** with those inputs (if required) and retains the result
* Builds an HTTP response (typically with the Domain invocation results).

All other logic, including all forms of input validation, error handling, and so on,
are therefore pushed out of the Action and into the **Domain**
(for domain logic concerns) or the response renderer (for presentation logic concerns).

A response could be rendered to HTML (e.g with Twig) for a standard web request; or
it might be something like JSON for RESTful API requests.

**Note:** [Closures](https://www.php.net/manual/en/class.closure.php) (functions) as routing
handlers are quite "expensive", because PHP has to create all closures for each request.
The use of class names is more lightweight, faster and scales better for larger applications.

More details about the flow of everything that happens when arriving a route
and the communication between the different layers can be found
here: [Action](https://odan.github.io/slim4-skeleton/action.html)

Create a sub-directory: `src/Action`

Create this action class in: `src/Action/HomeAction.php`

```php
<?php

namespace App\Action;

use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface;

final class HomeAction
{
    public function __invoke(
        ServerRequestInterface $request, 
        ResponseInterface $response
    ): ResponseInterface {
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

Now open your website, e.g. http://localhost and you should see the message `Hello, World!`.

### Creating a JSON Response

To create a valid JSON response you can write the json encoded string to the response body
and set the `Content-Type` header to `application/json`:

```php
<?php

namespace App\Action;

use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface;

final class HomeAction
{
    public function __invoke(
        ServerRequestInterface $request, 
        ResponseInterface $response
    ): ResponseInterface {
        $response->getBody()->write(json_encode(['hello' => 'world']));

        return $response->withHeader('Content-Type', 'application/json');
    }
}
```

Open your website, e.g. `http://localhost` and you should see the JSON response `{"hello":"world"}`.

## Conclusion

Remember the relationships:

* Slim - To handle routing and dispatching
* Single Action Controllers - Request and response handling.

The source code with more examples (e.g. reading a user) can be found here: 

* <https://github.com/odan/slim4-tutorial>

A complete skeleton for Slim 4 can be found here: 

* <https://github.com/odan/slim4-skeleton>

For technical questions create an issue here:

* <https://github.com/odan/slim4-tutorial/issues>

If you have Slim-Framework specific questions, visit:

* <https://discourse.slimframework.com/>
