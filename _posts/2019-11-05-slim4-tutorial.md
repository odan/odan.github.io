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
* [Directory structure](#directory-structure)
* [PSR-4 autoloading](#psr-4-autoloading)
* [Apache URL rewriting](#apache-url-rewriting)
* [Configuration](#configuration)
* [Startup](#startup)
* [Routing](#routing-setup)
* [Middleware](#middleware)
  * [What is a middleware?](#what-is-a-middleware)
  * [Routing and error middleware](#routing-and-error-middleware)
* [Container](#container)
  * [A quick guide to the container](#a-quick-guide-to-the-container)
  * [Container definitions](#container-definitions)
* [Base path](#base-path)
* [Your first route](#your-first-route)
* [Good URLs](#good-urls)
* [Actions](#actions)
* [Writing JSON to the response](#writing-json-to-the-response)
* [Domain](#domain)
  * [Services](#services)
  * [Validation](#validation)
  * [Repositories](#repositories)
* [Deployment](#deployment)
* [Conclusion](#conclusion)
* [Support](#support)
* [Read more](#read-more)

## Requirements

* PHP 7.2+
* MySQL 5.7+ or MariaDB
* Apache webserver with [mod_rewrite](https://httpd.apache.org/docs/2.4/mod/mod_rewrite.html) and [.htaccess](https://httpd.apache.org/docs/2.4/howto/htaccess.html)
* Composer (only for development)

## Introduction

Slim Framework is a great microframework for web applications, RESTful API's and websites.

Our aim is to create a RESTful API with routing, business logic and database operations.

Standards like [PSR](https://www.php-fig.org/psr/) and best practices are very important and integrated part of this tutorial.

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

As next we need a PSR-11 container implementation for **dependency injection** and **autowiring**.

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

A good directory structure helps you organize your code, simplifies setup on the webserver and increases the security of the entire application.

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
├── .gitignore          Git ignore rules
└── composer.json       Project dependencies and autoloader
```

In a web application, it is important to distinguish between the public and 
non-public areas.

The `public/` directory serves your application and will therefore also be 
directly accessible by all browsers, search engines and API clients. 
All other folders are not public and must not be accessible online. 
This can be done by defining the `public` folder in Apache as `DocumentRoot` 
of your website. But more about that later.

## PSR-4 autoloading

One of the most fundamental and important thing is to have a 
working [PSR-4 autoloader](https://www.php-fig.org/psr/psr-4/).
For the next steps we have to define the `src/` directory as root for the `\App` namespace.

Add this autoloading settings into `composer.json`:

```json
"autoload": {
    "psr-4": {
        "App\\": "src/"
    }
},
"autoload-dev": {
    "psr-4": {
        "App\\Test\\": "tests/"
    }
}
```

The complete `composer.json` file should look like this:

```json
{
    "require": {
        "php-di/php-di": "^6.0",
        "slim/psr7": "^1",
        "slim/slim": "^4.4"
    },
    "require-dev": {
        "phpunit/phpunit": "^8.4"
    },
    "autoload": {
        "psr-4": {
            "App\\": "src/"
        }
    },
    "autoload-dev": {
        "psr-4": {
            "App\\Test\\": "tests/"
        }
    },
    "config": {
        "process-timeout": 0,
        "sort-packages": true
    }
}
```

Run `composer update` for the changes to take effect.

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

Please **don't** change the `RewriteRule` directive. It must be exactly as shown above.

* Create a second `.htaccess` file in your project root-directory and copy/paste this content:

```htaccess
RewriteEngine on
RewriteRule ^$ public/ [L]
RewriteRule (.*) public/$1 [L]
```

Don't skip this step. This second `.htaccess` file is important to run your Slim app in a sub-directory
and within your development environment. 

* Create the front-controller file `public/index.php` and copy/paste this content:

```php
<?php

(require __DIR__ . '/../config/bootstrap.php')->run();
```

The [front controller](https://en.wikipedia.org/wiki/Front_controller) is the entry point 
to your slim application and handles all requests by channeling requests through a single handler object.

## Configuration

The directory for all configuration files is: `config/`

The file `config/settings.php` is the main configuration file and combines 
the default settings with environment specific settings. 

* Create a directory: `config/`
* Create a configuration file `config/settings.php` and copy/paste this content:

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

// Path settings
$settings['root'] = dirname(__DIR__);

// Error Handling Middleware settings
$settings['error'] = [

    // Should be set to false in production
    'display_error_details' => true,

    // Parameter is passed to the default ErrorHandler
    // View in rendered output by enabling the "displayErrorDetails" setting.
    // For the console and unit tests we also disable it
    'log_errors' => true,

    // Display error details in error log
    'log_error_details' => true,
];

return $settings;
```

### Startup

The app startup process contains the code that is executed when the application (request) is started. 

The bootstrap procedure includes the composer autoloader and then continues to
build the container, creates the app and registers the routes + middleware entries.

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

### Routing setup

Create a file for all routes `config/routes.php` and copy/paste this content:

```php
<?php

use Slim\App;

return function (App $app) {
    // empty
};

```

## Middleware

### What is a middleware?

A middleware can be executed before and after your Slim application to manipulate the request and response object according to your requirements.

[Read more](https://www.slimframework.com/docs/v4/concepts/middleware.html)

### Routing and error middleware

Create a file to load global middleware handler `config/middleware.php` and copy/paste this content:

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
    $app->add(ErrorMiddleware::class);
};
```

## Container

### A quick guide to the container

**[Dependency injection](https://en.wikipedia.org/wiki/Dependency_injection)** is passing 
dependency to other objects. Dependency injection makes testing easier. 
The injection can be done through a constructor.

A **dependencies injection container** (aka IoC Container) is a tool for injecting dependencies.

**A general rule:** The core application should not use the container.
Injecting the container into a class is an **anti-pattern**. You should declare all class 
dependencies in the constructor explicitly. 

Why is injecting the container (in most cases) an anti-pattern?

In Slim 3 the [Service Locator](https://blog.ploeh.dk/2010/02/03/ServiceLocatorisanAnti-Pattern/) (anti-pattern) was the
default "style" to inject the whole ([Pimple](https://pimple.symfony.com/)) container and fetch the dependencies from it. 
However, there are the following disadvantages:

* The Service Locator (anti-pattern) **hides the actual dependencies** of your class. 
* The Service Locator (anti-pattern) also violates the [Inversion of Control](https://en.wikipedia.org/wiki/Inversion_of_control) (IoC) principle of [SOLID](https://en.wikipedia.org/wiki/SOLID).

Q: How can I make it better? 

A: Use [composition over inheritance](https://en.wikipedia.org/wiki/Composition_over_inheritance) 
and (constructor) **dependency injection**. 

Since **Slim 4** you can use modern tools like [PHP-DI](http://php-di.org/) with the awesome [autowire](http://php-di.org/doc/autowiring.html) feature. 
This means: Now you can declare all dependencies explicitly in your constructor and let the DIC inject these 
dependencies for you. 

To be more clear: Composition has nothing to do with the "autowire" feature of the DIC. You can use composition 
with pure classes and without a container or anything else. The autowire feature just uses the 
[PHP Reflection](https://www.php.net/manual/en/book.reflection.php) classes to resolve and inject the 
dependencies automatically for you.

### Container definitions
 
Slim 4 uses a dependency injection container to prepare, manage and inject application dependencies. 

You can add any container library that implements the [PSR-11](https://www.php-fig.org/psr/psr-11/) interface.

Create a new file for the container entries `config/container.php` and copy/paste this content:

```php
<?php

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
        return $container->get(App::class)->getResponseFactory();
    },

    ErrorMiddleware::class => function (ContainerInterface $container) {
        $app = $container->get(App::class);
        $settings = $container->get('settings')['error'];

        return new ErrorMiddleware(
            $app->getCallableResolver(),
            $app->getResponseFactory(),
            (bool)$settings['display_error_details'],
            (bool)$settings['log_errors'],
            (bool)$settings['log_error_details']
        );
    },

];
```

## Base path

Most people will get a **404 error (not found)**, because the basePath is not set correctly.

If you run your Slim app in a sub-directory, resp. not directly within the 
[DocumentRoot](https://httpd.apache.org/docs/2.4/en/mod/core.html#documentroot)
of your webserver, you must set the "correct" base path.

Ideally the `DoumentRoot` of your production server points directly to the `public/` directory.

In all other cases you have to make sure, that your base path is correct. For example,
the DocumentRoot directory is `/var/www/domain.com/htdocs/`, but the application
is stored under `/var/www/domain.com/htdocs/my-app/`, then you have to set `/my-app` as base path.

To be more precise: In this context “sub-directory” means a sub-directory of the project,
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

```
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

Now that you have installed the `BasePathMiddleware`, remove this line (if exists): `$app->setBasePath('...');`.

## Your first route

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

## Good URLs

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

$app->get('/hello', function (ServerRequestInterface $request, ResponseInterface $response) {
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
and the communication between the different layers can be found here: [Action](https://odan.github.io/slim4-skeleton/action.html)

* Create a sub-directory: `src/Action`
* Create this action class in: `src/Action/HomeAction.php`

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
$app->get('/', \App\Action\HomeAction::class)->setName('home');
```

The complete `config/routes.php` should look like this now:

```php
<?php

use Slim\App;

return function (App $app) {
    $app->get('/', \App\Action\HomeAction::class)->setName('home');
};
```

Now open your website, e.g. http://localhost and you should see the message `Hello, World!`.

### Writing JSON to the response

To create a valid JSON response you can write the json encoded string to the response body
and set the `Content-Type` header to  `application/json`:

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
        $response->getBody()->write(json_encode(['success' => true]));

        return $response->withHeader('Content-Type', 'application/json');
    }
}
```

Open your website, e.g. http://localhost and you should see the JSON response `{"success":true}`.

To change to http status code, just use the `$response->withStatus(x)` method. Example:

```php
$result = ['error' => ['message' => 'Validation failed']];

$response->getBody()->write(json_encode($result));

return $response
    ->withHeader('Content-Type', 'application/json')
    ->withStatus(422);
```

## Domain

Forget CRUD! Your API should reflect the business [use cases](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html) 
and not the technical "database operations" aka. [CRUD](https://en.wikipedia.org/wiki/Create,_read,_update_and_delete). 
Don't put business logic into actions. The action invokes the domain layer, 
resp. the service. If you want to reuse the same logic in another action, 
then just invoke that service you need in your action.

### Services

The Domain is the place for the complex [business logic](https://en.wikipedia.org/wiki/Business_logic).

Instead of putting the logic into gigantic (fat) "Models", we put the logic into smaller, 
specialized **Service** classes.

A service provides a specific functionality or a set of functionalities, such as the retrieval of 
specified information or the execution of a set of operations, with a purpose that different clients 
can reuse for different purposes.

There can be multiple clients for a service, e.g. the action (request), 
another service, the CLI (console) and the unit-test environment (phpunit).

> A service class is not a "Manager" or "Utility" class.

Each service class should have only one responsibility, e.g. to transfer money from A to B, and not more.

Separate **data** from **behavior** by using services for the behavior and [Data transfer objects](https://en.wikipedia.org/wiki/Data_transfer_object) for the data.

The directory for all (domain) modules and sub-modules is: `src/Domain`

Create the code for the service class `src/Domain/User/Service/UserCreator.php`:

```php
<?php

namespace App\Domain\User\Service;

use App\Domain\User\Repository\UserCreatorRepository;
use App\Exception\ValidationException;

/**
 * Service.
 */
final class UserCreator
{
    /**
     * @var UserCreatorRepository
     */
    private $repository;

    /**
     * The constructor.
     *
     * @param UserCreatorRepository $repository The repository
     */
    public function __construct(UserCreatorRepository $repository)
    {
        $this->repository = $repository;
    }

    /**
     * Create a new user.
     *
     * @param array $data The form data
     *
     * @return int The new user ID
     */
    public function createUser(array $data): int
    {
        // Input validation
        $this->validateNewUser($data);

        // Insert user
        $userId = $this->repository->insertUser($data);

        // Logging here: User created successfully
        //$this->logger->info(sprintf('User created successfully: %s', $userId));

        return $userId;
    }

    /**
     * Input validation.
     *
     * @param array $data The form data
     *
     * @throws ValidationException
     *
     * @return void
     */
    private function validateNewUser(array $data): void
    {
        $errors = [];

        // Here you can also use your preferred validation library

        if (empty($data['username'])) {
            $errors['username'] = 'Input required';
        }

        if (empty($data['email'])) {
            $errors['email'] = 'Input required';
        } elseif (filter_var($data['email'], FILTER_VALIDATE_EMAIL) === false) {
            $errors['email'] = 'Invalid email address';
        }

        if ($errors) {
            throw new ValidationException('Please check your input', $errors);
        }
    }
}
```

Take a look at the **constructor**! You can see that we have declared the `UserCreatorRepository` as a
dependency, because the service can only interact with the database through the repository.

### Validation

To validate the clients input we have to check the data and collect all errors 
into a collection (array) of errors. 
This pattern is called [notification pattern](https://martinfowler.com/articles/replaceThrowWithNotification.html).  

Create a new file in `src/Exception/ValidationException.php` and copy/paste this content: 

```php
<?php

namespace App\Exception;

use RuntimeException;
use Throwable;

final class ValidationException extends RuntimeException
{
    private $errors;

    public function __construct(
        string $message, 
        array $errors = [], 
        int $code = 422, 
        Throwable $previous = null
    ){
        parent::__construct($message, $code, $previous);

        $this->errors = $errors;
    }

    public function getErrors(): array
    {
        return $this->errors;
    }
}
```

If you like this pattern for validation, I recommend having a look at this library: 
[selective/validation](https://github.com/selective-php/validation)

### Repositories

A repository is responsible for the data access logic, communication with a database.

There are two types of repositories: collection-oriented and persistence-oriented repositories. 
In this case, we are talking about **persistence-oriented repositories**, since these are better 
suited for processing large amounts of data.

A repository is the source of all the data your application needs 
and mediates between the service and the database. A repository improves code maintainability, testing and readability by separating **business logic** 
from **data access logic** and provides centrally managed and consistent access rules for a data source. 
Each public repository method represents a query. The return values represent the result set 
of a query and can be primitive/object or list (array) of them. Database transactions must 
be handled on a higher level (service) and not within a repository.

### Creating a repository

For this tutorial we need a test database with a `users` table.
Please execute this SQL statement in your test database.

```sql
CREATE TABLE `users` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `username` varchar(255) COLLATE utf8mb4_unicode_ci DEFAULT NULL,
  `email` varchar(255) COLLATE utf8mb4_unicode_ci DEFAULT NULL,
  `first_name` varchar(255) COLLATE utf8mb4_unicode_ci DEFAULT NULL,
  `last_name` varchar(255) COLLATE utf8mb4_unicode_ci DEFAULT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `username` (`username`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

Add the database settings into: `config/settings.php`:

```php
// Database settings
$settings['db'] = [
    'driver' => 'mysql',
    'host' => 'localhost',
    'username' => 'root',
    'database' => 'test',
    'password' => '',
    'charset' => 'utf8mb4',
    'collation' => 'utf8mb4_unicode_ci',
    'flags' => [
        // Turn off persistent connections
        PDO::ATTR_PERSISTENT => false,
        // Enable exceptions
        PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
        // Emulate prepared statements
        PDO::ATTR_EMULATE_PREPARES => true,
        // Set default fetch mode to array
        PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
        // Set character set
        PDO::MYSQL_ATTR_INIT_COMMAND => 'SET NAMES utf8mb4 COLLATE utf8mb4_unicode_ci'
    ],
];
```

> **Note:** I use PDO and SQL here for learning purposes only. In a real application the use 
> of an SQL QuickBuilder would be recommended from a maintainability and security point of view.

Insert a `PDO::class` container definition to `config/container.php`:

```php
PDO::class => function (ContainerInterface $container) {
    $settings = $container->get('settings')['db'];

    $host = $settings['host'];
    $dbname = $settings['database'];
    $username = $settings['username'];
    $password = $settings['password'];
    $charset = $settings['charset'];
    $flags = $settings['flags'];
    $dsn = "mysql:host=$host;dbname=$dbname;charset=$charset";

    return new PDO($dsn, $username, $password, $flags);
},
```

From now on, PHP-DI will always inject the same PDO instance as soon as we declare PDO in a 
constructor as dependency.

The following Repository class uses the [Data Mapper pattern](https://designpatternsphp.readthedocs.io/en/latest/Structural/DataMapper/README.html).
A Data Mapper, is a Data Access Layer that performs bidirectional transfer 
of data between a persistent data store (often a relational database) 
and an in memory data representation (the domain layer). 
The goal of the pattern is to keep the in memory representation and the 
persistent data store independent of each other and the data mapper itself.

Generic mappers will handle many domain entity types, 
dedicated mappers will handle one or a few.

The key point of this pattern is, unlike Active Record pattern, 
the data model follows Single Responsibility Principle.

Create a new directory: `src/Domain/User/Repository`

Create the file: `src/Domain/User/Repository/UserCreatorRepository.php` and insert this content:

```php
<?php

namespace App\Domain\User\Repository;

use PDO;

/**
 * Repository.
 */
final class UserCreatorRepository
{
    /**
     * @var PDO The database connection
     */
    private $connection;

    /**
     * Constructor.
     *
     * @param PDO $connection The database connection
     */
    public function __construct(PDO $connection)
    {
        $this->connection = $connection;
    }

    /**
     * Insert user row.
     *
     * @param array $user The user
     *
     * @return int The new ID
     */
    public function insertUser(array $user): int
    {
        $row = [
            'username' => $user['username'],
            'first_name' => $user['first_name'],
            'last_name' => $user['last_name'],
            'email' => $user['email'],
        ];

        $sql = "INSERT INTO users SET 
                username=:username, 
                first_name=:first_name, 
                last_name=:last_name, 
                email=:email;";

        $this->connection->prepare($sql)->execute($row);

        return (int)$this->connection->lastInsertId();
    }
}

```

Note that we have declared `PDO` as a dependency, because the repository requires a database connection.

The last part is to register a new route for `POST /users`.

Create a new action class in: `src/Action/UserCreateAction.php`:

```php
<?php

namespace App\Action;

use App\Domain\User\Service\UserCreator;
use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface;

final class UserCreateAction
{
    private $userCreator;

    public function __construct(UserCreator $userCreator)
    {
        $this->userCreator = $userCreator;
    }

    public function __invoke(
        ServerRequestInterface $request, 
        ResponseInterface $response
    ): ResponseInterface {
        // Collect input from the HTTP request
        $data = (array)$request->getParsedBody();

        // Invoke the Domain with inputs and retain the result
        $userId = $this->userCreator->createUser($data);

        // Transform the result into the JSON representation
        $result = [
            'user_id' => $userId
        ];

        // Build the HTTP response
        $response->getBody()->write((string)json_encode($result));

        return $response
            ->withHeader('Content-Type', 'application/json')
            ->withStatus(201);
    }
}
```

In this example, we create a "barrier" between source data and output, 
so that schema changes do not affect the clients. For the sake of 
simplicity, this is done using the same method. In reality, you would 
separate the input data mapping and output JSON conversion into 
separate parts of your application.

Add the new route in `config/routes.php`:

```php
$app->post('/users', \App\Action\UserCreateAction::class);
```

The complete project structure should look like this now:

![image](https://user-images.githubusercontent.com/781074/68902256-1a5bac80-0738-11ea-8e7d-d106fe7e2368.png)

Now you can test the `POST /users` route with [Postman](https://www.getpostman.com/) to see if it works.

If successful, the result should look like this:

![image](https://user-images.githubusercontent.com/781074/68299379-ddd6e380-009b-11ea-9ead-4c3b12b62807.png)

## Deployment

For deployment on a productive server, there are some important settings and security related things to consider.

You can use composer to generate an optimized build of your application. 
All dev-dependencies are removed, and the Composer autoloader is optimized for performance. 

Run this command in the same directory as the project’s composer.json file:

```
composer install --no-dev --optimize-autoloader
```

You don't have to run composer on your production server. Instead, you should implement a 
[build pipeline](https://www.amazon.com/dp/B003YMNVC0) that creates a
so called "artifact". An artifact is an ZIP file you can upload and deploy on 
your production server.

These tools are very useful to automate your software build processes:

* [Apache Ant](https://ant.apache.org/bindownload.cgi) - It's similar to Make, but it's implemented in Java.
* [Phing](https://www.phing.info/) - A PHP build tool inspired by Apache Ant.

For security reasons, you should switch off the output of all error details in production:

```php
$settings['error']['display_error_details'] = false;
```

**Important**: It's very important to set the Apache `DocumentRoot` to the `public/` directory. 
Otherwise, it may happen that someone else could access internal files from the web. [More details](https://www.digitalocean.com/community/tutorials/how-to-move-an-apache-web-root-to-a-new-location-on-ubuntu-16-04)

`/etc/apache2/sites-enabled/000-default.conf`

```htacess
DocumentRoot /var/www/example.com/htdocs/public
```

**Tip:** Never store secret passwords in your git / SVN repository. 
Instead, you could store them in a file like `env.php` and place this file 
one directory above your application directory. e.g.

```
/var/www/example.com/env.php
```

## Conclusion

Remember the relationships:

* Slim - To handle routing and dispatching
* Single Action Controllers - Request, response handling. Invoking the domain (service method).
* Domain - The core layer of your application
* Services - To handle business logic
* Repositories - To execute database queries

The source code with more examples (e.g. reading a user) can be found here: <https://github.com/odan/slim4-tutorial>

A complete skeleton for Slim 4 can be found here: <https://github.com/odan/slim4-skeleton>

### Support

For technical questions create an issue here:

* <https://github.com/odan/slim4-tutorial/issues>

If you have Slim-Framework specific questions, visit:

* <https://discourse.slimframework.com/>

### Read more

* [Slim 4 - Cheatsheet and FAQ](https://odan.github.io/2019/09/09/slim-4-cheatsheet-and-faq.html)

#### Videos

* [Slim 4 - Hallo Welt](https://www.youtube.com/watch?v=c6IMi3NlIeU) (german)
