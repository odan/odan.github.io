---
title: Slim 4 - Tutorial
layout: post
comments: true
published: true
description: 
keywords: php slim tutorial
---

This tutorial shows you how to work with the powerful and lightweight Slim 4 framework.

## Table of contents

* [Requirements](#requirements)
* [Introduction](#introduction)
* [Installation](#installation)
* [Directory structure](#directory-structure)
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
* [PSR-4 autoloading](#psr-4-autoloading)
* [Actions](#action)
* [Writing JSON to the response](#writing-json-to-the-response)
* [Domain](#domain)
  * [Services](#services)
  * [Data Transfer Objects](#data-transfer-objects-dto)
  * [Repositories](#repositories)
* [Deployment](#deployment)
* [Conclusion](#conclusion)
* [FAQ](#faq)

## Requirements

* PHP 7.2+
* MySQL 5.7+
* Apache webserver
* Composer

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

To access the application configuration install the `selective/config` package:

```
composer require selective/config
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

// Error reporting for production
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

// Error Handling Middleware settings
$settings['error_handler_middleware'] = [

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

use Selective\Config\Configuration;
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

**[Dependency injection](https://en.wikipedia.org/wiki/Dependency_injection)** is passing dependency to other objects.
Dependency injection makes testing easier. The injection can be done through a constructor.

A **dependencies injection container** (DIC) is a tool for injecting dependencies.

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

Dependency injection is a programming practice of passing into an object it’s collaborators, 
rather the object itself creating them. 

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
use Selective\Config\Configuration;
use Slim\App;
use Slim\Factory\AppFactory;
use Slim\Middleware\ErrorMiddleware;

return [
    Configuration::class => function () {
        return new Configuration(require __DIR__ . '/settings.php');
    },

    App::class => function (ContainerInterface $container) {
        AppFactory::setContainer($container);
        $app = AppFactory::create();

        // Optional: Set the base path to run the app in a sub-directory
        // The public directory must not be part of the base path
        //$app->setBasePath('/slim4-tutorial');

        return $app;
    },

    ErrorMiddleware::class => function (ContainerInterface $container) {
        $app = $container->get(App::class);
        $settings = $container->get(Configuration::class)->getArray('error_handler_middleware');

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

If you run your Slim app in a sub-directory, resp. not directly within the 
[DocumentRoot](https://httpd.apache.org/docs/2.4/en/mod/core.html#documentroot)
of your webserver, you must set the "correct" base path.

Ideally the `DoumentRoot` of your production server points directly to the `public/` directory.

In all other cases you have to make sure, that your base path is correct. For example,
the DocumentRoot directory is `/var/www/domain.com/htdocs/`, but the application
is stored under `/var/www/domain.com/htdocs/my-app/`, then you have to set `/my-app` as base path.

Example:

```php
$app->setBasePath('/my-app');
```

Be careful: The `public/` directory is only the `DoumentRoot` of your webserver, 
but it's never part of your base path and the official url.

Bad urls:
* `http://www.example.com/public`
* `http://www.example.com/public/users`
* `http://www.example.com/my-app/public`
* `http://www.example.com/my-app/public/users`

Good urls:
* `http://www.example.com`
* `http://www.example.com/users`
* `http://www.example.com/my-app`
* `http://www.example.com/my-app/users`

## Your first route

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

Now open your website, e.g. http://localhost and you should see the message `Hello, World!`.

If you get a **404 error (not found)**, you should define the correct **basePath** in `config/container.php`.

**Example:**
```
$app->setBasePath('/slim4-tutorial');
```

## PSR-4 autoloading

For the next steps we have to register the `\App` namespace for the PSR-4 autoloader.

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
        "selective/config": "^0.2.0",
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

## Action

Each **Single Action Controller** is represented by a individual class or closure.

The *Action* does only these things:

* collects input from the HTTP request (if needed)
* invokes the **Domain** with those inputs (if required) and retains the result
* builds an HTTP response (typically with the Domain invocation results).

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

* Create a directory: `src/`
* Create a sub-directory: `src/Action`
* Create this action class in: `src/Action/HomeAction.php`

```php
<?php

namespace App\Action;

use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface;

final class HomeAction
{
    public function __invoke(ServerRequestInterface $request, ResponseInterface $response): ResponseInterface
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
    public function __invoke(ServerRequestInterface $request, ResponseInterface $response): ResponseInterface
    {
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

return $response->withHeader('Content-Type', 'application/json')->withStatus(422);
```

## Domain

Forget CRUD! Your API should reflect the business [use cases](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html) 
and not the technical "database operations" aka. [CRUD](https://en.wikipedia.org/wiki/Create,_read,_update_and_delete). 
Don't put business logic into actions. The action invokes the domain layer, 
resp. the application service. If you want to reuse the same logic in another action, 
then just invoke that application service you need in your action.

### Services

The [Domain](https://github.com/pmjones/adr/blob/master/ADR.md#model-versus-domain) is the place for the
complex [business logic](https://en.wikipedia.org/wiki/Business_logic).

Instead of putting the logic into gigantic (fat) "Models", we but the logic into smaller, 
specialized **Service** classes, aka Application Service.

A service provides a specific functionality or a set of functionalities, such as the retrieval of 
specified information or the execution of a set of operations, with a purpose that different clients 
can reuse for different purposes.

There can be multiple clients for a service, e.g. the action (request), 
another service, the CLI (console) and the unit-test environment (phpunit).

> A service class is not a "Manager" or "Utility" class.

Each service class should have only one responsibility, e.g. to transfer money from A to B, and not more.

Separate **data** from **behavior** by using services for the behavior and DTO's for the data.

The directory for all (domain) modules and sub-modules is: `src/Domain`

**Pseudo example:**

```php
use App\Domain\User\Data\UserCreateData;
use App\Domain\User\Service\UserCreator;

$user = new UserCreateData();
$user->username = 'john.doe';
$user->firstName = 'John';
$user->lastName = 'Doe';
$user->email = 'john.doe@example.com';

$service = new UserCreator();
$service->createUser($user);
```

### Data Transfer Objects (DTO) 
  
A DTO contains only pure **data**. There is no business or domain specific logic. 
There is also no database access within a DTO. 
A service fetches data from a repository and  the repository (or the service) 
fills the DTO with data. A DTO can be used to transfer data inside or outside the domain.

Create a DTO class to hold the data in this file: `src/Domain/User/Data/UserCreateData.php`

```php
<?php

namespace App\Domain\User\Data;

final class UserCreateData
{
    /** @var string */
    public $username;

    /** @var string */
    public $firstName;

    /** @var string */
    public $lastName;

    /** @var string */
    public $email;
}
```

Create the code for the service class `src/Domain/User/Service/UserCreator.php`:

```php
<?php

namespace App\Domain\User\Service;

use App\Domain\User\Data\UserCreateData;
use App\Domain\User\Repository\UserCreatorRepository;
use InvalidArgumentException;

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
     * @param UserCreateData $user The user data
     *
     * @throws InvalidArgumentException
     *
     * @return int The new user ID
     */
    public function createUser(UserCreateData $user): int
    {
        // Validation
        if (empty($user->username)) {
            throw new InvalidArgumentException('Username required');
        }

        // Insert user
        $userId = $this->repository->insertUser($user);

        // Logging here: User created successfully

        return $userId;
    }
}
```

Take a look at the **constructor**! You can see that we have declared the `UserCreatorRepository` as a
dependency, because the service can only interact with the database through the repository.

### Repositories

A repository is responsible for the data access logic, communication with database(s).

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

Insert a `PDO::class` container definition to `config/container.php`:

```php
PDO::class => function (ContainerInterface $container) {
    $settings = $container->get(Configuration::class)->getArray('db');

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

Create a new directory: `src/Domain/User/Repository`

Create the file: `src/Domain/User/Repository/UserCreatorRepository.php` and insert this content:

```php
<?php

namespace App\Domain\User\Repository;

use App\Domain\User\Data\UserCreateData;
use PDO;

/**
 * Repository.
 */
class UserCreatorRepository
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
     * @param UserCreateData $user The user
     *
     * @return int The new ID
     */
    public function insertUser(UserCreateData $user): int
    {
        $row = [
            'username' => $user->username,
            'first_name' => $user->firstName,
            'last_name' => $user->lastName,
            'email' => $user->email,
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

use App\Domain\User\Data\UserCreateData;
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

    public function __invoke(ServerRequestInterface $request, ResponseInterface $response): ResponseInterfacee
    {
        // Collect input from the HTTP request
        $data = (array)$request->getParsedBody();

        // Mapping (should be done in a mapper class)
        $user = new UserCreateData();
        $user->username = $data['username'];
        $user->firstName = $data['first_name'];
        $user->lastName = $data['last_name'];
        $user->email = $data['email'];

        // Invoke the Domain with inputs and retain the result
        $userId = $this->userCreator->createUser($user);

        // Transform the result into the JSON representation
        $result = [
            'user_id' => $userId
        ];

        // Build the HTTP response
        $response->getBody()->write((string)json_encode($result));

        return $response->withHeader('Content-Type', 'application/json')->withStatus(201);
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
All dev-dependencies are removed and the Composer autoloader is optimized for performance. 

Run this command in the same directory as the project’s composer.json file:

```
composer install --no-dev --optimize-autoloader
```

You don't have to run composer on your production server. Instead you should implement a [build pipeline](https://www.amazon.com/dp/B003YMNVC0) that creates
an so called "artifact". An artifact is an ZIP file you can upload and deploy on your production server.

* [Apache Ant](https://ant.apache.org/bindownload.cgi) is a software great tool for automating software build processes.
* [selective-php/artifact](https://github.com/selective-php/artifact) is a tool, written in PHP, to build artifacts from your source code.

For security reasons, you should switch off the output of all error details in production:

```php
$settings['error_handler_middleware'] = [
    'display_error_details' => false,
];
```

If you have to run your Slim application in a sub-directory, you could try this library: [selective/basepath](https://github.com/selective-php/basepath)

**Important**: It's very important to set the Apache `DocumentRoot` to the `public/` directory. 
Otherwise, it may happen that someone else could access internal files from the web. [More details](https://www.digitalocean.com/community/tutorials/how-to-move-an-apache-web-root-to-a-new-location-on-ubuntu-16-04)

`/etc/apache2/sites-enabled/000-default.conf`

```htacess
DocumentRoot /var/www/example.com/htdocs/public
```

**Tip:** Never store secret passwords in your git / SVN repository. 
Instead you could store them in a file like `env.php` and place this file one directory above your application directory. e.g.

```
/var/www/example.com/env.php
```

## Conclusion

Remember the relationships:

* Slim - To handle routing and dispatching
* Single Action Controllers - Request, response handling. Invoking the domain (service method).
* Domain - The core of your application
* Services - To handle business logic
* DTO - To carry data (no behavior)
* Repositories - To execute database queries

## FAQ

### General questions

Read more: [Slim 4 - Cheatsheet and FAQ](https://odan.github.io/2019/09/09/slim-4-cheatsheet-and-faq.html)

### Where can I find the code on github?

The source code with more examples (e.g. reading a user) can be found here: <https://github.com/odan/slim4-tutorial>

A complete skeleton for slim 4 can be found here: <https://github.com/odan/slim4-skeleton>

### How to add JSON Web Token (JWT) / Bearer authentication?

Read this article: [Slim 4 - OAuth 2.0 and JSON Web Token Setup](https://odan.github.io/2019/12/02/slim4-oauth2-jwt.html) 

### How to setup CORS?

Read this article:  [Slim 4 - CORS setup](https://odan.github.io/2019/11/24/slim4-cors.html) 

### How to add a logger?

You could inject a logger factory, e.g. like the [LoggerFactory](https://github.com/odan/slim4-skeleton/blob/master/src/Factory/LoggerFactory.php)
The settings are defined [here](https://github.com/odan/slim4-skeleton/blob/master/config/defaults.php#L43). 

### I get a 404 (not found) error

Follow the instructions and define the correct base path with `$app->setBasePath('my-sub-directory/');`

If you have to run your Slim application in a sub-directory, you could try this library: [selective/basepath](https://github.com/selective-php/basepath)

### Error message: Callable (...) does not exist

Run `composer update` to fix it.

### How to add a database connection?

You can add a query builder as described here:

* [Slim 4 - Laminas Query Builder Setup](https://odan.github.io/2019/12/01/slim4-laminas-db-query-builder-setup.html)
* [Slim 4 - CakePHP Query Builder Setup](https://odan.github.io/2019/12/03/slim4-cakephp-query-builder.html)
* [Slim 4 - Eloquent Setup](https://odan.github.io/2019/12/03/slim4-eloquent.html)

### How to add multiple database connections?

* [Slim 4 - Eloquent multiple connections setup](https://odan.github.io/2019/12/03/slim4-eloquent.html#setup-multiple-connections)

### How to build assets with webpack?

* [Slim 4 - Compiling Assets with Webpack](https://odan.github.io/2019/09/21/slim4-compiling-assets-with-webpack.html)

### Where can I donate?

If you think this tutorial is useful for you, 
I would appreciate a small donation: 
**[Donate](../../../donate.html)** 

### I have a very special question

You can consult me by email: d.opitz (at) outlook.com

[Comments](https://gist.github.com/odan/c8bee474b0054a06776481a6c8de1d8f)
