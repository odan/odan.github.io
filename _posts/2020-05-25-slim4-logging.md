---
title: Slim 4 - Logging
layout: post
comments: true
published: true
description: 
keywords: slim, slimphp, php, monolog, logging
---

## Table of contents

* [Requirements](#requirements)
* [Introduction](#introduction)
* [Installation](#installation)
* [Configuration](#configuration)
* [Logger Factory](#logger-factory)
* [Usage](#usage)
* [Logging to the console](#logging-to-the-console)

## Requirements

* PHP 7.2+
* Composer (dev environment)
* A Slim 4 application

## Introduction

This guide shows how to install the PSR-3 logger implementation
[Monolog](https://seldaek.github.io/monolog/) and how to control 
the output according to your needs.

### Installation

To install composer, run:

```
composer require monolog/monolog
```

## Configuration

Insert the logger settings into your configuration file, e.g. `config/settings.php`;

```php
// Logger settings
$settings['logger'] = [
    'name' => 'app',
    'path' => __DIR__ . '/../logs',
    'filename' => 'app.log',
    'level' => \Monolog\Logger::DEBUG,
    'file_permission' => 0775,
];
```

Make sure, that the `{project-root}/logs/` directory exists.

### Logger Factory

By default a monolog instance would output it's messages into a single file.

Now imagine what a mess it would be if all your modules would log everything
to the same file. Of course you could add a `RotatingFileHandler`, but
it's still not very clean in the long run. To make our logging system more flexible,
and compatible with a container we add a special `LoggerFactory` class
to create a custom logger per class.

Create a new file `src/Factory/LoggerFactory.php` and copy/paste this content:

```php
<?php

namespace App\Factory;

use Monolog\Formatter\LineFormatter;
use Monolog\Handler\RotatingFileHandler;
use Monolog\Handler\StreamHandler;
use Monolog\Logger;
use Psr\Log\LoggerInterface;

/**
 * Factory.
 */
final class LoggerFactory
{
    /**
     * @var string
     */
    private $path;

    /**
     * @var int
     */
    private $level;

    /**
     * The constructor.
     *
     * @param array $settings The settings
     */
    public function __construct(array $settings)
    {
        $this->path = (string)$settings['path'];
        $this->level = (int)$settings['level'];
    }

    /**
     * @var array Handler
     */
    private $handler = [];

    /**
     * Build the logger.
     *
     * @param string $name The name
     *
     * @return LoggerInterface The logger
     */
    public function createInstance(string $name): LoggerInterface
    {
        $logger = new Logger($name);

        foreach ($this->handler as $handler) {
            $logger->pushHandler($handler);
        }

        $this->handler = [];

        return $logger;
    }

    /**
     * Add rotating file logger handler.
     *
     * @param string $filename The filename
     * @param int $level The level (optional)
     *
     * @return LoggerFactory The logger factory
     */
    public function addFileHandler(string $filename, int $level = null): self
    {
        $filename = sprintf('%s/%s', $this->path, $filename);
        $rotatingFileHandler = new RotatingFileHandler($filename, 0, $level ?? $this->level, true, 0777);

        // The last "true" here tells monolog to remove empty []'s
        $rotatingFileHandler->setFormatter(new LineFormatter(null, null, false, true));

        $this->handler[] = $rotatingFileHandler;

        return $this;
    }

    /**
     * Add a console logger.
     *
     * @param int $level The level (optional)
     *
     * @return self The instance
     */
    public function addConsoleHandler(int $level = null): self
    {
        $streamHandler = new StreamHandler('php://stdout', $level ?? $this->level);
        $streamHandler->setFormatter(new LineFormatter(null, null, false, true));

        $this->handler[] = $streamHandler;

        return $this;
    }
}
```

Add a new container definition for the `LoggerFactory::class` in `config/container.ph`:

```php
<?php

use Psr\Container\ContainerInterface;
use Selective\BasePath\BasePathMiddleware;
use Selective\Config\Configuration;
use Slim\App;
use Slim\Factory\AppFactory;
use Slim\Middleware\ErrorMiddleware;

return [
    // ...

    LoggerFactory::class => function (ContainerInterface $container) {
        return new LoggerFactory($container->get(Configuration::class)->getArray('logger'));
    },
];
```

## Usage

Okay, great! From now on you can create your custom logfile per class by using the 
`LoggerFactory` instance. 

Here is an example:

```php
<?php

namespace App\Domain\User\Service;

use App\Factory\EmailFactory;
use App\Factory\LoggerFactory;
use Exception;
use Psr\Log\LoggerInterface;

final class UserCreator
{
    /**
     * @var LoggerInterface
     */
    private $logger;

    public function __construct(LoggerFactory $logger) {
        $this->logger = $logger
            ->addFileHandler('user_creator.log')
            ->createInstance('user_creator');
    }

    public function registerUser(array $user): int
    {
        try {
            // Do something...
            $userId = 1234;
    
            // Log success
            $this->logger->info(sprintf('User created: %s', $userId));

            return $userId;
        } catch (Exception $exception) {
            // Log error message
            $this->logger->error($exception->getMessage());
            
            throw $exception;
        }
    }
}

```

## Logging to the console

If you want to display the log messages in the console, 
you only need to add the console handler:

```php
public function __construct(LoggerFactory $logger) {
    $this->logger = $logger
        ->addConsoleHandler()
        ->createInstance('my_console_command');
}
```

You can also combine the file and console handlers:

```php
public function __construct(LoggerFactory $logger) {
    $this->logger = $logger
        ->addFileHandler('my_cronjob.log')
        ->addConsoleHandler()
        ->createInstance('my_cronjob');
}
```
