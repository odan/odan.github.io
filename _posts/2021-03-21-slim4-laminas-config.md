---
title: Slim 4 - Laminas Config
layout: post
comments: true
published: true
description:
keywords: php, slim, configuration, config
image: https://odan.github.io/assets/images/slim-logo-600x330.png
---

## Table of contents

* [Requirements](#requirements)
* [Introduction](#introduction)
* [Installation](#installation)
* [Container Setup](#container-setup)
* [Usage](#usage)
* [Conclusion](#conclusion)
* [Read more](#read-more)

## Requirements

* PHP 7.4+ or PHP 8.0+
* [A Slim 4 application](https://odan.github.io/2019/11/05/slim4-tutorial.html)

## Introduction

Configuration files allow you to configure things like your database
connection information, your mail server information, as well as various
other core configuration values such as your application timezone and encryption key.

In my tutorials, I use the `settings` DI container key to load and fetch 
the settings for the sake of simplicity, flexibility and performance. 
For some people, this style is not DI container friendly, or OOP, or modern enough, 
because they prefer to declare a Config class within their application class 
instead of a simple array. The advantage of using a Config object is that 
the DI container can **autowire** this Config object, 
so you don't need to wire the class manually within the DI container.

## Installation

The `laminas/laminas-config` component is designed to simplify access to configuration 
data within applications. It provides a nested object, property-based user 
interface for accessing this configuration data within application code. 
The configuration data may come from a variety of formats supporting hierarchical data storage. 

```
composer require laminas/laminas-config
```

## Container Setup

PHP-based configuration files are often recommended due to the speed with
which they are parsed, and the fact that they can be cached by opcode caches.

Configuration data are made accessible to the `Laminas\Config\Config`
constructor via an associative array, which may be multi-dimensional
so data can be organized from general to specific.

The following code illustrates how to use PHP configuration files.

Add a DI container definition for `Laminas\Config\Config:class` in `config/container.php`.

```php
<?php

use Laminas\Config\Config;
// ...

return [
    // ...
    
    Config::class => function () {
        return new Config(require __DIR__ . '/settings.php');
    },
];
```

## Configuration

All the configuration files will be stored in the `config/` directory.

If not exists, create a file `config/settings.php`.

The settings file must return an associative array with multi-dimensional data.

Example:

```php
$settings = [];

$settings['upload_directory'] = __DIR__ . '/../uploads';

$settings['logger'] = [
    'name' => 'app',
    'path' => __DIR__ . '/../logs',
    'filename' => 'app.log',
    'level' => \Monolog\Logger::DEBUG,
    'file_permission' => 0775,
];

$settings['db'] = [
    // ...
];

// ...

return $settings;
```

## Usage

### Container Settings

To configure other container entries within the DI container
you can access the `Config` object like this:

```php
// Get Config object
$config = $container->get(Config::class);

// Read config value
$loggerSettings = $config->get('logger');
```

If you still have the DI container "settings" entry
from the previous Slim tutorial, you can refactor the
settings array and remove it from the DI container.

Old:

```php
LoggerFactory::class => function (ContainerInterface $container) {
    return new LoggerFactory($container->get('settings')['logger']);
},
```

New:

```php
LoggerFactory::class => function (ContainerInterface $container) {
    return new LoggerFactory($container->get(Config::class)->get('logger'));
},
```

Remove this entry, if exists:

```php
'settings' => function () {
    return require __DIR__ . '/settings.php';
},
```

### Accessing Configuration Values

To access the `Config` object, we must first declare it in the constructor so that it can
be automatically injected by the DI Container.

Example:

```php
<?php

namespace App\Action;

use Laminas\Config\Config;
use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface;

final class ExampleAction
{
    private Config $config;

    public function __construct(Config $config)
    {
        $this->config = $config;
    }

    public function __invoke(
        ServerRequestInterface $request,
        ResponseInterface $response
    ): ResponseInterface {
        $uploadDirectory = $this->config->get('upload_directory');
        // ...
        
        return $response;
    }
}
```

### Accessing Configuration Values

You may easily access your configuration values using the config property within your class.

```php
$value = $this->config->get('timezone');

// Retrieve a default value if the configuration value does not exist...
$value = $this->config->get('timezone', 'Europe/Berlin');
```

## Conclusion

As you can see, it is now possible to inject the Config object where 
you want to read configuration values. The use of an object also 
has the advantage that the configuration can now be injected automatically via autowiring.

## Read more

* [Laminas-Config Documentation](https://docs.laminas.dev/laminas-config/)
* [Github Repository](https://github.com/laminas/laminas-config/)
* [Ask a question or report an issue](https://github.com/odan/slim4-tutorial/issues)