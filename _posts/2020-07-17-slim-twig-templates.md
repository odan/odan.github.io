---
title: Slim 4 - Twig Setup
layout: post
comments: true
published: true
description:
keywords: php slim twig templates html engine symfony
---

## Table of contents

* [Requirements](#requirements)
* [Introduction](#introduction)
* [Installation](#installation)
* [Template Location](#installation)
* [Configuration](#configuration)
* [Creating Templates](#creating-templates)
* [Rendering Templates](#rendering-templates)
* [Linking to Pages](#linking-to-pages)
* [Linking to CSS, JavaScript and Image Assets](#linking-to-css-javascript-and-image-assets)
* [Translations](#translations)
* [Read more](#read-more)

## Requirements

* PHP 7.2+
* [A Slim 4 application](https://odan.github.io/2019/11/05/slim4-tutorial.html)

## Introduction

This tutorial shows how to install and use the
[Twig](https://symfony.com/doc/current/mailer.html)
template engine in a Slim 4 project.

### Performance

Twig compiles templates down to plain optimized PHP code. 
The overhead compared to regular PHP code was reduced to the very minimum.

### Security

I think one of the main benefits of Twig, over a
native PHP templates, is the killer feature: 
[automatic output escaping](https://twig.symfony.com/doc/3.x/filters/escape.html).
This means, by default, Twig uses the PHP native `htmlspecialchars` function for the HTML output.

> **HTML encoding** replaces certain characters that are semantically meaningful in HTML markup, 
> with equivalent characters that can be displayed to the user without affecting parsing 
> the markup.
> 
> The most significant and obvious characters are <, >, &, and " which are are replaced with 
> &lt;, &gt;, &amp;, and &quot;, respectively. Additionally, an encoder may replace high-order 
> characters with the equivalent HTML entity encoding, so content can be preserved and properly 
> rendered even in the event the page is sent to the browser as ASCII.

In a native PHP template you must manually encode the output like this:

```php
echo htmlspecialchars($var, ENT_QUOTES | ENT_SUBSTITUTE, 'UTF-8');
```

In Twig html encoding is enabled by default:

{% raw %}
```twig
{{ var }}
```
{% endraw %}

But you still have to be carefull, because contextual 
[escaping](https://twig.symfony.com/doc/3.x/filters/escape.html)
in HTML documents is hard.

For example, if you want to put an value into an HTML attribute,
then the value has to be [html attribute encoded](https://twig.symfony.com/doc/3.x/filters/escape.html).

> **HTML attribute encoding**, on the other hand, only replaces a subset of those characters 
> that are important to prevent a string of characters from breaking the attribute of an HTML 
> element. Specifically, you'd typically just replace ", &, and < with &quot;, &amp;, and &lt;. 
> This is because the nature of attributes, the data they contain, and how they are parsed and 
> interpreted by a browser or HTML parser is different than how an HTML document and its 
> elements are read.

**Example:**

{% raw %}
```twig
<a href="{{ var|e('html_attr') }}">Link</a>
```
{% endraw %}

At the end you still have to choose the right escaping strategy for the specific context.

## Installation

To install the Slim Twig component run the following command:

```
composer require slim/twig-view
```

## Template Location

Templates are stored by default in the `templates/` directory. 
When a controller or action renders the `product/index.twig` template, 
they are actually referring to the file:

`{your-project}/templates/product/index.twig`

The default templates directory is configurable and you can add more template
directories as explained later in this article.

Create a new directory in your project root directory: `templates/`

## Configuration

Twig has several configuration options to define things like the format 
used to display numbers and dates, the template caching, etc. 
Read the [Twig configuration reference](https://symfony.com/doc/current/reference/configuration/twig.html)
to learn about them.

Add the following settings to your Slim settings array, e.g `config/settings.php`:

```php
// Twig settings
// Configuration reference: https://symfony.com/doc/current/reference/configuration/twig.html
$settings['twig'] = [
    // Template paths
    'paths' => [
        __DIR__ . '/../templates',
    ],
    // Twig environment options
    'options' => [
        // Should be set to true in production
        'cache_enabled' => false,
        'cache_path' => __DIR__ . '/../tmp/twig',
    ],
];
```

Create the twig cache directory: `{project}/tmp/twig/`

### Container setup

Autowire the `Twig` component and the `TwigMiddleware` in `config/container.php`:

```php
<?php

use Psr\Container\ContainerInterface;
use Selective\Config\Configuration;
use Slim\App;
use Slim\Factory\AppFactory;
use Slim\Views\Twig;
use Slim\Views\TwigMiddleware;

return [

    // ...    

    // Twig templates
    Twig::class => function (ContainerInterface $container) {
        $config = $container->get(Configuration::class);
        $settings = $config->getArray('twig');

        $options = $settings['options'];
        $options['cache'] = $options['cache_enabled'] ? $options['cache_path'] : false;

        $twig = Twig::create($settings['paths'], $options);

        // Add more extension here
        // ...
        
        return $twig;
    },

    TwigMiddleware::class => function (ContainerInterface $container) {
        return TwigMiddleware::createFromContainer($container->get(App::class), Twig::class);
    },
];

```

### Middleware

Add the `TwigMiddleware` into the middleware stack:

```php
<?php

use Slim\App;
use Slim\Middleware\ErrorMiddleware;
use Slim\Views\TwigMiddleware;

return function (App $app) {
    $app->addBodyParsingMiddleware();

    $app->add(TwigMiddleware::class); // <--- here

    $app->addRoutingMiddleware();
    $app->add(ErrorMiddleware::class);
};

```

## Creating Templates

Symfony recommends `snake_case` for filenames and directories, e.g.

* `blog_posts.twig`
* `admin/default_theme/blog/index.twig` 
* etc...

First, you need to create a new `hello.twig` file in the `templates/` directory 
to store  the template contents:

```twig
<h1>Hello {{ name }}!</h1>
<p>You have {{ notifications|length }} new notifications.</p>
```

## Rendering Templates

Inject the twig instance into your own controller action and use its render() method.
When using autowiring you only need to add an argument in the constructor 
and type-hint it.

Create the file `src/Action/HelloAction.php` and copy/paste this content:

```php
<?php

namespace App\Action;

use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface;
use Slim\Views\Twig;

final class HelloAction
{
    private $twig;

    public function __construct(Twig $twig)
    {
        $this->twig = $twig;
    }

    public function __invoke(ServerRequestInterface $request, ResponseInterface $response): ResponseInterface
    {
        $viewData = [
            'name' => 'World',
            'notifications' => [
                'message' => 'You are good!'
            ],
        ];
        
        return $this->twig->render($response, 'hello.twig', $viewData);
    }
}
```

Create a new route for the `HelloAction` in `src/config/routes.php`:

```php
$app->get('/hello', \App\Action\HelloAction::class);
```

Now open the browser and navigate to the `/hello` route, e.g. `http://localhost/hello`

You should see a rendered output like this:

> <h1>Hello World!</h1>
> <p>You have 1 new notifications.</p>

## Linking to Pages

The [Slim Twig View](https://github.com/slimphp/Twig-View#custom-template-functions) component 
provides special functions to your Twig templates like `url_for()` etc.

## Linking to CSS, JavaScript and Image Assets

In case you are using webpack to bundle your assets, you should take
a look at the [Twig Webpack extension](https://github.com/fullpipe/twig-webpack-extension).

**Read more**

* [Twig Webpack extension setup](https://odan.github.io/2019/09/21/slim4-compiling-assets-with-webpack.html#twig-webpack-extension-setup)

## Translations

The [symfony/twig-bridge](https://github.com/symfony/twig-bridge) provides a Twig 3 `TranslationExtension` 
to translate messages with the [trans](https://symfony.com/doc/current/reference/twig_reference.html#trans)
filter.

To install the translator component run:

```
composer require symfony/translation
```

To install the translator extension for Twig run:

```
composer require symfony/twig-bridge
```

Add the `TranslationExtension` to the Twig environment:

```php
use Symfony\Bridge\Twig\Extension\TranslationExtension;
use Symfony\Component\Translation\Translator;
use Psr\Container\ContainerInterface;

// ...
return [
    // ...
    Twig::class => function (ContainerInterface $container) {
        // ...
        $twig = Twig::create($paths, $options);

        $translator = $container->get(Translator::class);
        $twig->addExtension(new TranslationExtension($translator)); // <--- here
        
        // ...
        
        return $twig;
    }
];
```

To extract the messages you could use the [PoEdit Pro Version](https://poedit.net/pro).

If yout don't want to buy the Pro version, then you should "compile" the Twig
templates to PHP and parse the Twig cache files with 
[PoEdit](https://poedit.net/download) (free).

A Twig compiler for the console can be found here: [TwigCompilerCommand](https://github.com/odan/slim4-skeleton/blob/master/src/Console/TwigCompilerCommand.php)

In PoEdit add the twig cache path as additional sources path, e.g. `tmp/twig`: 

![image](https://user-images.githubusercontent.com/781074/79620639-ca32ad00-8110-11ea-8582-d74707220dbc.png)

Add `trans` as additional keyword to your po-file:

![image](https://user-images.githubusercontent.com/781074/79620557-8b9cf280-8110-11ea-8f8c-3a49fdde5997.png)

Then compile all Twig templates to php files and let PoEdit parse all PHP files for new messages.

How to use the `trans` filter you can learn here:

* <https://symfony.com/doc/current/translation.html>
* <https://symfony.com/doc/current/reference/twig_reference.html#trans>

## Read more

* [Twig](https://twig.symfony.com/)
* [Creating and Using Templates](https://symfony.com/doc/current/templates.html)

[Donate](../../../donate.html)