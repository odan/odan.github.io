---
title: Creating your first Slim 3 Framework Application
layout: post
comments: true
published: true
description: 
keywords: 
---

The Slim Framework is a great micro frameworks for Web application, RESTful API's and Websites. 
This Tutorial shows how to create a very basic but flexible project for every use case.

**Update:** Please check out the new **[Slim 4 tutorial](https://odan.github.io/2019/11/05/slim4-tutorial.html)**!

## Table of contents

* [Requirements](#requirements)
* [Introduction](#introduction)
* [Installation](#installation)
* [Directory structure](#directory-structure)
* [Front controller](#front-controller)
* [Bootstrap](#bootstrap)
* [Container](#container)
* [Middleware](#middleware)
* [Routes](#routes)
* [Configuration](#configuration)
* [Composer](#composer)
* [Starting](#starting)
* [Errors and Logging](#errors-and-logging)
* [Views and Templates](#views-and-templates)
* [Database](#database)
* [Deployment](#deployment)

## Requirements

* PHP 7.x
* MySQL 5.7+
* [Apache](https://gist.github.com/odan/dcf6c3155899677ee88a4f7db5aac284#install-apache-php-mysql)
* Apache [mod_rewrite](https://gist.github.com/odan/dcf6c3155899677ee88a4f7db5aac284#enable-apache-mod_rewrite) module
* [Composer](https://gist.github.com/odan/dcf6c3155899677ee88a4f7db5aac284#install-composer)

## Introduction

I'm sure you've read the official Slim tutorial "[First Application Walkthrough](https://www.slimframework.com/docs/tutorial/first-app.html)" and found that it doesn't work as expected on your server. Most of the people I spoke to on StackOverflow and in the Slim Forum had similar problems due to this tutorial.

I think this tutorial should follow good practices and the directory structure of [Slim-Skeleton](https://github.com/slimphp/Slim-Skeleton). The current version of the official tutorial is "outdated" because it does not reflect the best practices of the PHP community and makes it very difficult, especially for beginners.

This tutorial tries to make the start easier and less confusing.

## Installation

Composer is the best way to install Slim Framework. Open the console and change to the project root directory. Then enter:

```bash
composer require slim/slim
```

This creates a new folder `vendor/` and the files `composer.json` + `composer.lock` with all the dependencies of your application.

Please don't commit the `vendor/` to your git repository. To set up the git repository correctly, create a file called `.gitignore` in the project root folder and add the following lines to this file:

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

In a web application, it is important to distinguish between the public and non-public areas.

The folder `public` serves your application and will therefore also be directly accessible by all browsers, search engines and API clients. All other folders are not public and must not be accessible online. This can be done by defining the `public` folder in Apache as `DocumentRoot` of your website. But more about that later.

## Front controller

The [front controller](https://en.wikipedia.org/wiki/Front_controller) is the entry point to your slim application and handles all requests by channeling requests through a single handler object.

The content of  `public/.htaccess`:

```htaccess
# Redirect to front controller
RewriteEngine On
# RewriteBase /
RewriteCond %{REQUEST_FILENAME} !-d
RewriteCond %{REQUEST_FILENAME} !-f
RewriteRule ^ index.php [QSA,L] 
```

### Internal redirect to the front controller

The `.htaccess` file in the project root is required in at least two cases.

* For the development environment

For example, if you are developing with XAMPP, you have only one host and several subdirectories. So that Slim runs in this environment without additional setup, this file was created. This simplifies the entire development environment, as you can easily manage any number of applications at the same time.

* In case your application is deployed within subfolders of the vhost. (I've seen that many times before).

The content of `.htaccess`:

```htaccess
RewriteEngine on
RewriteRule ^$ public/ [L]
RewriteRule (.*) public/$1 [L]
```

## Bootstrap

Add the following code to the front controller file `public/index.php`.

```php
<?php

/** @var Slim\App $app */
$app = require __DIR__ . '/../config/bootstrap.php';

// Start
$app->run();

```

The entire configuration of your application is stored and managed in the `config` folder. 

Create the following config files with all settings, routings, middleware etc.

Content of file `config/bootstrap.php`

```php
<?php

require_once __DIR__ . '/../vendor/autoload.php';

// Instantiate the app
$app = new \Slim\App(['settings' => require __DIR__ . '/../config/settings.php']);

// Set up dependencies
require  __DIR__ . '/container.php';

// Register middleware
require __DIR__ . '/middleware.php';

// Register routes
require __DIR__ . '/routes.php';

return $app;

```

## Container

Slim uses a dependency container to prepare, manage, and inject application dependencies. 
Slim supports containers that implement PSR-11 or the Container-Interop interface.

The container configuration is an importand part of a good application setup.

Content of file `config/container.php`

```php
<?php

use Slim\Container;

/** @var \Slim\App $app */
$container = $app->getContainer();

// Activating routes in a subfolder
$container['environment'] = function () {
    $scriptName = $_SERVER['SCRIPT_NAME'];
    $_SERVER['SCRIPT_NAME'] = dirname(dirname($scriptName)) . '/' . basename($scriptName);
    return new Slim\Http\Environment($_SERVER);
};
```

## Middleware

Content of file `config/middleware.php`.

At the moment this file contains no middleware.

```php
<?php

// Slim middleware


```

## Routes

Content of file `config/routes.php`

```php
<?php

use Slim\Http\Request;
use Slim\Http\Response;

$app->get('/', function (Request $request, Response $response) {
    $response->getBody()->write("It works! This is the default welcome page.");

    return $response;
})->setName('root');

$app->get('/hello/{name}', function (Request $request, Response $response) {
    $name = $request->getAttribute('name');
    $response->getBody()->write("Hello, $name");

    return $response;
});
```

## Configuration

Content of file `config/settings.php`

```php
<?php

$settings = [];

// Slim settings
$settings['displayErrorDetails'] = true;
$settings['determineRouteBeforeAppMiddleware'] = true;

// Path settings
$settings['root'] = dirname(__DIR__);
$settings['temp'] = $settings['root'] . '/tmp';
$settings['public'] = $settings['root'] . '/public';

// View settings
$settings['twig'] = [
    'path' => $settings['root'] . '/templates',
    'cache_enabled' => false,
    'cache_path' =>  $settings['temp'] . '/twig-cache'
];

// Database settings
$settings['db'] = [
    'driver' => 'mysql',
    'host' => 'localhost',
    'username' => 'root',
    'database' => 'test',
    'password' => '',
    'charset' => 'utf8',
    'collation' => 'utf8_unicode_ci',
    'flags' => [
        PDO::ATTR_PERSISTENT => false,
        // Enable exceptions
        PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
        // Set default fetch mode
        PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
    ],
];

return $settings;
```

## Composer

By convention all application specific PHP classes are stored in the `src` folder. We create the namespace `App` and use the `src` folder as a starting point for composer:

Content of `composer.json`

```json
{
  "require": {
    "php": "^7.0",
    "slim/slim": "^3.9"
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

Run `composer update` so that the changes take effect.

## Starting

Now open the browser and navigate to the slim application:

* http://localhost/

If you are running the web app within a subdirectory just add the name of the subdirectory to the url.

* http://localhost/{my-project-name}/

You should see the message: `It works! This is the default welcome page.`

Then open http://localhost/hello/world or http://localhost/{my-project-name}/hello/world

You should see the message: `Hello, world`

## Views and Templates

The [Twig View](https://github.com/slimphp/Twig-View) is a Slim Framework view helper built on top of the [Twig](https://twig.symfony.com/) templating component. You can use this component to create and render templates in your Slim Framework application.

### Twig installation

Run composer

```bash
composer require slim/twig-view
```

Add this code to the container settings file: `config/container.php`

```php
use Slim\Views\Twig;

// Register Twig View helper
$container[Twig::class] = function (Container $container) {
    $settings = $container->get('settings');
    $viewPath = $settings['twig']['path'];

    $twig = new Twig($viewPath, [
        'cache' => $settings['twig']['cache_enabled'] ? $settings['twig']['cache_path'] : false
    ]);

    /** @var Twig_Loader_Filesystem $loader */
    $loader = $twig->getLoader();
    $loader->addPath($settings['public'], 'public');

    // Instantiate and add Slim specific extension
    $router = $container->get('router');
    $uri = \Slim\Http\Uri::createFromEnvironment($container->get('environment'));
    $twig->addExtension(new \Slim\Views\TwigExtension($router, $uri));

    return $twig;
};
```

### Twig templates

Add a new template: `templates/time.twig`

{% raw %}
```twig
<!DOCTYPE html>
<html>
<head>
    <base href="{{ base_url() }}/"/>
</head>
<body>
Current time: {{ now }}
</body>
</html>
```
{% endraw %}

Add a new route in `config/routes.php`

```php
use Slim\Views\Twig;

$app->get('/time', function (Request $request, Response $response) {
    $viewData = [
        'now' => date('Y-m-d H:i:s')
    ];

    return $this->get(Twig::class)->render($response, 'time.twig', $viewData);
});
```

Then open http://localhost/time or http://localhost/{my-project-name}/time

You should see the message: `Current time: 2017-12-06 21:52:57`

## Errors and Logging

### Display Error Details

What can I do with a 500 Internal Server Error?

That's probably happened to all of us before: You open your website and find only a more or less meaningful page that reports a "Code 500 - Internal Server Error". But what does that mean?

This is a very general message from the server that an error has occurred, which is almost certainly due to the configuration of the server or the incorrect execution of a server script.

The default error handler can also include detailed error diagnostic information. To enable this you need to set the `displayErrorDetails` setting to `true`:

```php
$settings['displayErrorDetails'] = true;
```

### Logging of errors

Error logging is an essential part of a web application or API. After each change to the system or configuration, new minor errors may occur. Therefore it is important to log these errors at least in log files and to check them regularly.

A quick checklist:

* Does your Apache `DocumentRoot` points to the `public` folder?
* Is the Apache `mod_rewrite` module enabled?
* Are the file system permissions correct?
* Are the database connection parameters correct?

To see where the problem is, you should log all errors in a logfile. Here you can find more instructions:

[Logging errors in Slim 3](https://akrabat.com/logging-errors-in-slim-3/)

### Logging

No we add Monolog to our Slim application.

Installation

```
composer require monolog/monolog
```

Add this new settings in `config/settings.php`

```php
// Logger settings
$settings['logger'] = [
    'name' => 'app',
    'file' => $settings['temp'] . '/logs/app.log',
    'level' => \Monolog\Logger::ERROR,
];
```

Add a new logger entry in `config/container.php`

```php
use Monolog\Handler\RotatingFileHandler;
use Monolog\Logger;
use Psr\Log\LoggerInterface;

$container[LoggerInterface::class] = function (Container $container) {
    $settings = $container->get('settings')['logger'];
    $level = isset($settings['level']) ?: Logger::ERROR;
    $logFile = $settings['file'];

    $logger = new Logger($settings['name']);
    $handler = new RotatingFileHandler($logFile, 0, $level, true, 0775);
    $logger->pushHandler($handler);

    return $logger;
};
```

Add a new route in `config/routes.php`

```php
use Psr\Log\LoggerInterface;
use Slim\Container;

$app->get('/logger-test', function (Request $request, Response $response) {
    /** @var Container $this */
    /** @var LoggerInterface $logger */

    $logger = $this->get(LoggerInterface::class);
    $logger->error('My error message!');

    $response->getBody()->write("Success");

    return $response;
});
```

Then open http://localhost/logger-test or http://localhost/{my-project-name}/logger-test

Check the content of the logfile in the project sub-directory: `tmp/logs/app-2018-04-24.log`

You should see the logged error message. Example: `[2018-04-24 21:12:47] app.ERROR: My error message! [] []`

## Database

### Database configuration

Adjust the necessary connection settings to the file `config/settings.php`.

### PDO

Add a container entry for the PDO connection:

```php
$container[PDO::class] = function (Container $container) {
    $settings = $container->get('settings');
    
    $host = $settings['db']['host'];
    $dbname = $settings['db']['database'];
    $username = $settings['db']['username'];
    $password = $settings['db']['password'];
    $charset = $settings['db']['charset'];
    $collate = $settings['db']['collation'];
    
    $dsn = "mysql:host=$host;dbname=$dbname;charset=$charset";
    
    $options = [
        PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
        PDO::ATTR_PERSISTENT => false,
        PDO::ATTR_EMULATE_PREPARES => true,
        PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
        PDO::MYSQL_ATTR_INIT_COMMAND => "SET NAMES $charset COLLATE $collate"
    ];

    return new PDO($dsn, $username, $password, $options);
};
```

### Query Builder

For security reasons (SQL injections), SQL statements should no longer be written by yourself, but generated using a query builder. For this purpose, the PHP community offers already established and tested libraries. Here is a selection I can recommend:

* [CakePHP Database](https://github.com/cakephp/database)
* ~~[Illuminate Database](https://github.com/illuminate/database)~~

You should use a query builder only within a persistent oriented Repository or a DataMapper class.
Here you can find some examples of a Data Mapper class. [TicketMapper](https://github.com/slimphp/Tutorial-First-Application/blob/master/src/classes/TicketMapper.php), [ComponentMapper.php](https://github.com/slimphp/Tutorial-First-Application/blob/master/src/classes/ComponentMapper.php).

In this tutorial I will use the [CakePHP Database query builder](https://github.com/cakephp/database), because with Illuminate Database we would get too much global laravel functions and classes as dependencies on board. Laravel Illuminate does not return arrays, but collections with stdClass instances. The CakePHP Database query builder is a very flexible and powerful Database abstraction library with a familiar PDO-like API. I have been able to achieve the best results so far, even with very complex queries.

**Installation**

```
composer require cakephp/database
```

Add a new container entry in the file: `config/container.php`

```php
use Cake\Database\Connection;
use Cake\Database\Driver\Mysql;

$container[Connection::class] = function (Container $container) {
    $settings = $container->get('settings');
    $driver = new Mysql($settings['db']);

    return new Connection(['driver' => $driver]);
};

$container[PDO::class] = function (Container $container) {
    /** @var Connection $connection */
    $connection = $container->get(Connection::class);
    $connection->getDriver()->connect();

    return $connection->getDriver()->getConnection();
};
```

Generate and execute a query:

```php
use Cake\Database\Connection;
use Slim\Container;
use Slim\Http\Request;
use Slim\Http\Response;

$app->get('/databases', function (Request $request, Response $response) {
    /** @var Container $this */

    $query = $this->get(Connection::class)->newQuery();

    // fetch all rows as array
    $query = $query->select('*')->from('information_schema.schemata');
    
    $rows = $query->execute()->fetchAll('assoc') ?: [];

    // return a json response
    return $response->withJson($rows);
});
```

You can also create very complex SQL queries. Here are just some CRUD examples:

```php
use Cake\Database\Connection;

// Create a new Query instance for this connection
$connection = $this->get(Connection::class);
$query = $connection->newQuery();

// Retrieving a single Row
$query = $query->select(['id', 'username', 'email'])
    ->from('users')
    ->andWhere(['id' => 1]);
    
$userRow = $query->execute()->fetch('assoc') ?: [];

// Retrieving all rows from a table as array
$query = $query->select(['id', 'username', 'email'])
    ->from('users');

$userRows = $query->execute()->fetchAll('assoc') ?: [];

// Insert a new row
$data = ['username' => 'max', 'email' => 'max@example.com'];

$query->insert(array_keys($data))
    ->into('users')
    ->values($data)
    ->execute();

// Insert a new row and get the last inserted id (primary key)
$data = ['username' => 'max', 'email' => 'max@example.com'];

$userId = (int)$query->insert(array_keys($data))
    ->into('users')
    ->values($data)
    ->execute()
    ->lastInsertId();
    
// A shortcut for insert statements
$userId = (int)$connection->insert('users', $data)->lastInsertId();

// Delete user with id = 1
$query->delete('users')->andWhere(['id' => 1])->execute();

// Delete all users where users.votes > 100
$query->delete('users')->andWhere(['users.votes >' => 100])->execute();
```

Here you can find the full [documentation](https://book.cakephp.org/3.0/en/orm/query-builder.html).

## Deployment

For deployment on a productive server, there are some important settings and security releated things to consider.

You can use composer to generate an optimized build of your application. 
All dev-dependencies are removed and the Composer autoloader is optimized for performance. 

Run this command in the same directory as the project’s composer.json file:

```
composer install --no-dev --optimize-autoloader
```

Furthermore, you should activate caching for Twig to increase the performance of the template engine. 

```php
$settings['twig']['cache_enabled'] = true;
```

Routing can also be accelerated by using a cache file:

```php
$settings['routerCacheFile'] = __DIR__ . '/../tmp/routes.cache.php',
```

Set the option `displayErrorDetails` to `false` in production:

```php
$settings['displayErrorDetails'] = false;
```

**Important**: For security reasons you MUST set the Apache `DocumentRoot` and the `<Directory` of the webhosting to the `public` folder. Otherwise, it may happen that someone else accesses the internal files from outside. [More details](https://www.digitalocean.com/community/tutorials/how-to-move-an-apache-web-root-to-a-new-location-on-ubuntu-16-04)

`/etc/apache2/sites-enabled/000-default.conf`

```htacess
DocumentRoot /var/www/example.com/htdocs/public
```

Tip: Never store secret passwords in a Git / SVN repository. 
Instead you could store them in a file like `env.php` and place this file one directory above your application directory. e.g.

```
/var/www/example.com/env.php
```

Read more: [Part 2](https://odan.github.io/2019/03/18/creating-your-first-slim-framework-application-part-2.html)
