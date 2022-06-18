---
title: Slim 4 - Blade
layout: post
comments: true
published: true
description:
keywords: php, laravel, blade, template, templates
image: https://odan.github.io/assets/images/slim-logo-600x330.png
---

## Table of contents

* [Requirements](#requirements)
* [Introduction](#introduction)
* [Installation](#installation)
* [Configuration](#configuration)
* [Usage](#usage)
* [Sharing Data With All Views](#sharing-data-with-all-views)
* [Components](#components)
* [Conclusion](#conclusion)
* [Read more](#read-more)

## Requirements

* PHP 7.4+
* [A Slim 4 application](https://odan.github.io/2019/11/05/slim4-tutorial.html)

## Introduction

Blade is a template engine for PHP that is included with Laravel.
Unlike other PHP templating engines, Blade does not restrict
the use of plain PHP code in your templates.
All Blade templates are compiled to normal PHP code and cached until they are changed,
meaning that Blade essentially adds no additional overhead to your application.

In this article I would like to show a way to use the
Laravel's Blade Templating Engine outside of Laravel
and how to integrate it into a Slim 4 project.

## Installation

First, install the following packages using composer:

```php
composer require illuminate/view
composer require illuminate/config
```

Blade template files use the `.blade.php` file extension and are
usually stored in the `templates/` directory.

Create a new directory `templates` in your project root directory.

By default, Blade template views are compiled on demand.
When a request is executed that renders a view,
Blade will determine if a compiled version of the view exists.
If the file exists, Blade will then determine if the uncompiled
view has been modified more recently than the compiled view.
If the compiled view either does not exist, or the uncompiled
view has been modified, Blade will recompile the view.

Create a new template cache directory `{project}/tmp/templates` in your project root directory.
Make sure the directory `{project}/tmp/templates` exists and has read and write access permissions.

## Configuration

Add the new configuration keys in your `config/defaults.php` file:

```php
$settings['template'] = __DIR__ . '/../templates';
$settings['template_temp'] = __DIR__ . '/../tmp/templates';
```

Next, add the following DI container definitions:

```php
<?php

use Illuminate\Config\Repository;
use Illuminate\Container\Container as IlluminateContainer;
use Illuminate\Contracts\Container\Container as IlluminateContainerInterface;
use Illuminate\Contracts\View\Factory as IlluminateViewFactoryInterface;
use Illuminate\Events\Dispatcher;
use Illuminate\Filesystem\Filesystem;
use Illuminate\Support\Facades\Facade;
use Illuminate\View\Compilers\BladeCompiler;
use Illuminate\View\ViewServiceProvider;
use Psr\Container\ContainerInterface;
// ...

return [

    // ...
    
    IlluminateContainerInterface::class => function (ContainerInterface $container) {
        $app = IlluminateContainer::getInstance();
        Facade::setFacadeApplication($app);

        $app->bindIf('files', function () {
            return new Filesystem();
        }, true);

        $app->bindIf('events', function () {
            return new Dispatcher();
        }, true);

        $app->singleton('config', function () {
            return new Repository();
        });

        return $app;
    },

    IlluminateViewFactoryInterface::class => function (ContainerInterface $container) {
        $illuminateContainer = $container->get(IlluminateContainerInterface::class);

        $settings = $container->get('settings');
        $viewPaths = (array)$settings['template'];
        $cachePath = (string)$settings['template_temp'];

        /** @var Repository $config */
        $config = $illuminateContainer->get('config');
        $config->set([
            'view.paths' => $viewPaths,
            'view.compiled' => $cachePath,
        ]);

        $viewServiceProvider = new ViewServiceProvider($illuminateContainer);
        $viewServiceProvider->register();

        $view = $illuminateContainer->get('view');
        $illuminateContainer->singleton(IlluminateViewFactoryInterface::class, function () use ($view) {
            return $view;
        });

        return $view;
    },
];
```

This alone would technically work to render a Blade template,
but we also need to make it work with the PSR-7 response object.

For this purpose we create a special `TemplateRenderer`
class which does this work for us.

So next create a file in `src/Renderer/TemplateRenderer.php` and copy/paste this code:

```php
<?php

namespace App\Renderer;

use Illuminate\Contracts\View\Factory as IlluminateViewFactoryInterface;
use Psr\Http\Message\ResponseInterface;

final class TemplateRenderer
{
    private IlluminateViewFactoryInterface $view;

    public function __construct(IlluminateViewFactoryInterface $view)
    {
        $this->view = $view;
    }

    public function template(
        ResponseInterface $response,
        string $template,
        array $data = []
    ): ResponseInterface {
        $contents = $this->view->make($template, $data)->render();
        $response->getBody()->write($contents);

        return $response;
    }
}

```

## Usage

Instead of using the Blade Engine directly we use the `TemplateRenderer` object
to render the template into a PSR-7 compatible object.

A typical Action handler class might look like this to render
a template with the name `home.blade.php`:

```php
<?php

namespace App\Action\Home;

use App\Renderer\TemplateRenderer;
use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface;

final class HomeAction
{
    private TemplateRenderer $renderer;

    public function __construct(TemplateRenderer $renderer)
    {
        $this->renderer = $renderer;
    }

    public function __invoke(
        ServerRequestInterface $request,
        ResponseInterface $response
    ): ResponseInterface {
        // Passing data to views
        $viewData = [
            'name' => 'World',
        ];
        
        return $this->renderer->template($response, 'home', $viewData);
    }
}

```

To make it work, create a template file in `templates/home.blade.php` with this content:

```twig
Hello, {{ $name }}.
```

If everything is configured correctly you should see the following output:

```
Hello, World.
```

## Sharing Data With All Views

Occasionally, you may need to share data with all views that are rendered by your application.
You may do so by adding a `share` method to the `App\Renderer\TemplateRenderer` class.
Typically, you should place calls to the `share` method within a **middleware** `process` method.

```php
/**
 * Add a piece of shared data to the environment.
 *
 * @param array|string $key
 * @param mixed $value
 *
 * @return void
 */
public function share($key, $value = null): void
{
    $this->view->share($key, $value);
}
```

**Usage**

```
$this->renderer->share('csrf_token', 'random_value');
```

... or pass an array:

```
$this->renderer->share(
    [
        'google_analytics', 'UA-123456789-1',
        'key2' => 'value2',
    ]
);
```

More information about the Blade templating engine can be found on <https://laravel.com/docs/master/blade>.

## Components

[Components](https://laravel.com/docs/9.x/blade#components)
and slots provide similar benefits to sections,
layouts, and includes; however, some may find the mental
model of components and slots easier to understand.

However, if you really want to use Components you also need to
install the `laravel/framework` package. Otherwise, you would get an error message like this:

```
Target [Illuminate\Contracts\Foundation\Application] is not instantiable.
```

Furthermore, you would have to reconfigure the DI container to use
the `\Illuminate\Foundation\Application` class instead of the usual
`Illuminate\Contracts\Container\Container`.
This is only possible with some tricks and will probably only work
properly in a Laravel project.
The usage of Components without Laravel is therefore not recommended.
However, this functionality can also be implemented with the existing
Blade features, so the use of Components is not really necessary.

## Conclusion

The reason Blade is so popular is because of Laravel.

The Blade syntax stays much closer to PHP (which was a template engine itself 25 years ago).
For example, Twig is by design another language, making it much more difficult to wrap your head around.

I think blade is **not** the best example of software design, because
it's really hard to get running without Laravel. Even then, you may
never get all features (like Components) without installing the full Laravel framework package.
The Laravel DI container is a complex "beast" that has to be configured within (and partly beside) 
the PSR-11 DI container. Unfortunately, the package adds a lot of global
singletons (anti-pattern) to the project scope, which I really try to avoid.

Using Blade only makes sense in combination with Laravel.
Outside of Laravel I would not use this package and would not recommend it to anyone.

## Read more

* <https://laravel.com/docs/master/blade>
* <https://twitter.com/brendt_gd/status/1494319708161466369>
