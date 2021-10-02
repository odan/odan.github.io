---
title: Using the Symfony Service Container with a simple service object
layout: post
comments: true
published: false
description: 
keywords: 
---

Documentation - http://symfony.com/doc/current/service_container.html

```php
<?php

// Run: composer require symfony/dependency-injection
require_once __DIR__ . '/vendor/autoload.php';

use Symfony\Component\DependencyInjection\ContainerBuilder;
use Symfony\Component\DependencyInjection\Definition;

$container = new ContainerBuilder();

// Create a PDO service
$host = '127.0.0.1';
$db = 'test';
$user = 'root';
$pass = '';
$charset = 'utf8';

$dsn = "mysql:host=$host;dbname=$db;charset=$charset";
$opt = [
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
    PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
    PDO::ATTR_EMULATE_PREPARES => false,
];

$container->setDefinition('app.pdo', new Definition(
    PDO::class, array($dsn, $user, $pass, $opt)
));

var_dump($container->get('app.pdo')); // object(PDO)[8]
```

## DIC with configuration

```php
<?php

use Symfony\Component\DependencyInjection\ContainerBuilder;
use Symfony\Component\DependencyInjection\Definition;
use Symfony\Component\DependencyInjection\ParameterBag\ParameterBag;

// ------------------------------------------------------------------
// Configuration
// ------------------------------------------------------------------
$config = [];

// Database settings
$config['db'] = array(
    'dsn' => "mysql:host=127.0.0.1;dbname=test;charset=utf8",
    'username' => 'root',
    'password' => '',
    'options' => [
        // Enable exceptions
        PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
        // Set default fetch mode
        PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC
    ]
);

// ------------------------------------------------------------------
// Container services and definitions
// ------------------------------------------------------------------
$parameterBag = new ParameterBag($config);
$container = new ContainerBuilder($parameterBag);

// Using a callback function
$container->setDefinition('db', call_user_func(function () use ($container) {
    $settings = $container->getParameter('db');
    $arguments = [
        $settings['dsn'],
        $settings['username'],
        $settings['password'],
        $settings['options'],
    ];
    $definition = new Definition(PDO::class, $arguments);
    //$definition->addMethodCall('quote', ['test']);
    return $definition;
}));

var_dump($container->get('db')); // object(PDO)[8]

// ------------------------------------------------------------------
// Using a factory class
// ------------------------------------------------------------------
class PdoFactory
{
    public static function create($dsn, $username, $password, $options)
    {
        return new PDO($dsn, $username, $password, $options);
    }
}

$container->setDefinition('db2', call_user_func(function () use ($container) {
    $settings = $container->getParameter('db');
    $arguments = [
        $settings['dsn'],
        $settings['username'],
        $settings['password'],
        $settings['options'],
    ];
    $definition = new Definition(null, $arguments);
    $definition->setFactory('PdoFactory::create');
    return $definition;
}));

var_dump($container->get('db2')); // object(PDO)[11]
```