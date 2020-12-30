---
title: Generating Doctrine Entities from Existing Database
layout: post
comments: true
published: false
description: 
keywords: php doctrine orm models
---

## Requirements

* PHP 7+

## Setup

Add Doctrine ORM to the project:

```
composer require doctrine/orm
```

## Console Configuration

Create a file `doctrine.php` in the project root directory:

```php
<?php

// doctrine.php

use Doctrine\DBAL\Tools\Console\Helper\ConnectionHelper;
use Doctrine\ORM\EntityManager;
use Doctrine\ORM\Tools\Console\ConsoleRunner;
use Doctrine\ORM\Tools\Console\Helper\EntityManagerHelper;
use Doctrine\ORM\Tools\Setup;
use Doctrine\ORM\Version;
use Symfony\Component\Console\Application;
use Symfony\Component\Console\Helper\HelperSet;

// composer require doctrine/orm

// Set up autoloader
require_once __DIR__ . '/vendor/autoload.php';

// Load database configuration
require_once __DIR__ . '/config/doctrine-config.php';

$config = Setup::createAnnotationMetadataConfiguration($paths, $isDevMode);
$entityManager = EntityManager::create($dbParams, $config);

$helperSet = new HelperSet(array(
    'db' => new ConnectionHelper($entityManager->getConnection()),
    'em' => new EntityManagerHelper($entityManager)
));

$cli = new Application('Doctrine Command Line Interface', Version::VERSION);
$cli->setCatchExceptions(true);
$cli->setHelperSet($helperSet);

// Register All Doctrine Commands
ConsoleRunner::addCommands($cli);

// Optional: Register your own command
//$cli->addCommand(new \MyProject\Tools\Console\Commands\MyCustomCommand);

// Runs console application
$cli->run();
```

Create a file for the database settings (or include an existing file) e.g. `config/doctrine-config.php`:

```php
<?php

// doctrine-config.php

// Paths to Entities that we want Doctrine to see
$paths = array(
    __DIR__ . '/src/Domain/Model',
);

// Tells Doctrine what mode we want
$isDevMode = true;

// Doctrine connection configuration
$dbParams = array(
    'driver' => 'pdo_mysql',
    'user' => 'root',
    'password' => '',
    'dbname' => 'test',
    'charset' => 'utf8',
);
```

Change the database settings according to the environment specific credentials.

## Generating Entities

Open the console and validate the schema:

```shell
php vendor/bin/doctrine orm:validate-schema
```

Generate the PHP classes from the database schema:

```shell
php vendor/bin/doctrine orm:convert-mapping --from-database --force annotation ./src/Domain/Model
```

Now you can find the generated files under `src/Domain/Model`.

Note that Doctrine does only support PSR-0 
([and not PSR-4](https://github.com/doctrine/orm/issues/6716)) autoloading / namespaces. 
For this reason it's better to add the PSR-4 namespaces manually. 
Open the generated classes and add a new namespace like this:

```php
<?php

namespace App\Domain\Model;

// ...
```

Done
