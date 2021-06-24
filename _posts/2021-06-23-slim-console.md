---
title: Slim 4 - Console
layout: post
comments: true
published: true
description:
keywords: php, slim, console, cmd, commands
image: https://odan.github.io/assets/images/slim-logo-600x330.png
---

## Table of contents

* [Requirements](#requirements)
* [Introduction](#introduction)
* [Installation](#installation)
* [Console Script](#console-script)
* [Console Command](#console-command)
* [Usage](#usage)
* [Conclusion](#conclusion)
* [Read more](#read-more)

## Requirements

* PHP 7.4+ or PHP 8.0+
* [A Slim 4 application](https://odan.github.io/2019/11/05/slim4-tutorial.html)

## Introduction

Console commands can be used for any recurring task, such as cronjobs, imports, or other batch jobs.
Slim does not come with a console component,
so we will install it with Composer and implement our own console command with it.

## Installation

This will install Symfony Console and all required dependencies. 

Run:

```
composer require symfony/console
```

## Console Script

First we have to create a console executable for all our commands.

The `bin/` directory should contain all excecutable files.
If not exists, create a new directory in your project root: `bin/`

The file `console.php` will act as our main command line tool (like "artisan").
So create a new file: `bin/console.php` and copy/paste the following content:

```php
<?php

use Psr\Container\ContainerInterface;
use Symfony\Component\Console\Application;
use Symfony\Component\Console\Input\ArgvInput;

require_once __DIR__ . '/../vendor/autoload.php';

/** @var ContainerInterface $container */
$container = (require __DIR__ . '/../config/bootstrap.php')->getContainer();

$application = $container->get(Application::class);
$application->run();
```

In detail: First we bootstrap the Slim application to fetch the DI container from it.
Then we fetch a `Symfony\Component\Console\Application` 
instance from the DI container to invoke the `Application::run()` method to
run the actual console command application. 

Add a DI container definition for `Symfony\Component\Console\Application`
in `config/container.php`.

```php
<?php

use Psr\Container\ContainerInterface;
use Symfony\Component\Console\Application;
use Symfony\Component\Console\Input\InputOption;
// ...

return [
    // ...
    
    Application::class => function (ContainerInterface $container) {
        $application = new Application();

        foreach ($container->get('settings')['commands'] as $class) {
            $application->add($container->get($class));
        }

        return $application;
    },
];
```

You may have already noticed that the container definition reads the `commands` from the 
settings array and adds them into the console application.

## Console Command

For demo purposes we want to create a very simple console command.
All custom commands must at least be extended from `Symfony\Component\Console\Command\Command`.

Create a new command class, e.g. `src/Console/ExampleCommand.php` and copy/paste this class:

```php
<?php

namespace App\Console;

use Psr\Container\ContainerInterface;
use Symfony\Component\Console\Command\Command;
use Symfony\Component\Console\Input\InputInterface;
use Symfony\Component\Console\Output\OutputInterface;

final class ExampleCommand extends Command
{
    protected function configure(): void
    {
        parent::configure();

        $this->setName('example');
        $this->setDescription('A sample command');
    }

    protected function execute(InputInterface $input, OutputInterface $output): int
    {
        $output->writeln(sprintf('<info>Hello, console</info>'));

        return 0;
    }
}
```

To get objects from the DI container, just declare them within the class constructor:

```php
private LoggerInterface $logger;

public function __construct(LoggerInterface $logger) {
    parent::__construct(null);
    $this->logger = $logger;
}
```

To register a new command you have to open `config/settings.php`
and add a new `commands` array entry as show below:

```php
$settings['commands'] = [
    \App\Console\ExampleCommand::class,
    // Add more here...
];
```

## Usage

After configuring and registering the command, you can run it in the terminal:

```
php bin/console.php example
```

The output:

```
Hello, console
```

As you might expect, this command will not do that much as you didn't write
any complex logic yet. Add your own logic inside the `execute()` method.

## Conclusion

From my point of view, the Symfony Console component is the perfect tool for implementing
custom console commands. Of course there is much more to discover in the
[documentation](https://symfony.com/doc/current/console.html), like parameters,
user console input, different colors, testing, etc.

## Read more

* [Symfony Console Commands](https://symfony.com/doc/current/console.html)
* [Ask a question or report an issue](https://github.com/odan/slim4-tutorial/issues)