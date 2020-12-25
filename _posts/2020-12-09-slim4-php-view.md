---
title: Slim 4 - PHP Templates
layout: post
comments: false
published: true
keywords: php slim
---

## Table of contents

* [Requirements](#requirements)
* [Introduction](#introduction)
* [Installation](#installation)
* [Configuration](#configuration)
* [Container setup](#container-setup)
* [Helper functions](#helper-functions)
* [Usage](#usage)
* [Translations](#translations)
  * [Requirements](#translator-requirements)
  * [Translation helper](#translation-helper)
  * [Translations in PHP](#translations-in-php)
  * [Translations in templates](#translations-in-templates)
  * [Updating translation strings](#updating-translation-strings)
  * [Gettext pitfalls](#gettext-pitfalls)
* [URL helper](#url-helper)
  * [Template Usage](#template-usage)
* [Assets](#assets)
  * [Bundling assets](#bundling-assets)
* [Similar components](#similar-components)

## Requirements

* PHP 7.2+
* [A Slim 4 application](https://odan.github.io/2019/11/05/slim4-tutorial.html)

## Introduction

A very pleasant discussion with [Samuel Gfeller](https://github.com/samuelgfeller) inspired and motivated me to write an article about this topic.
The idea or "spirit" behind Slim is to keep your technology stack simple, lean and flexible.
I already wrote a lot about how to integrate and build a websites and web application using the Twig template engine in Slim.
Of course, Twig is absolutely fantastic, but for some developers, Twig still feels a bit "heavy" and over-engineered.
So I decided to show you a very lightweight or let's say "native" way to use templates in PHP 
without Twig. At the same time, this solution should not be inferior to Twig in terms of 
flexibility and extensibility.
With the help of small helper functions (comparable to Twig Extensions), we can extend 
PHP templates with any functionality we want.

Like [Plates](https://platesphp.com/), the [Slim/PHP-View](https://github.com/slimphp/PHP-View) component
uses PHP to render templates directly. You don't have to learn a new syntax
and your template code is getting cached by the [OPCache](https://www.php.net/manual/de/book.opcache.php)
automatically. This means you no longer need to create a local cache directory
for your template cache files. You also don't need to set special write permissions for directories etc.

While in Twig the output of variables is encoded into HTML by default, you have to take care of it yourself in a PHP template.
My experience shows that the more developers are involved in a project, the more difficult it becomes to enforce this security rule.
Therefore, checking the template code for this vulnerability is even more important when using this approach.

## Installation

To install the PHP-View component, run:

```
composer require slim/php-view
```

## Configuration

Create a new directory in your project root directory: `templates/`

Insert the session settings into your configuration file, e.g. `config/settings.php`;

```php
$settings['view'] = [
    // Path to templates
    'path' => __DIR__ . '/../templates',
];
```

## Container setup

Insert a DI container definition for `PhpRenderer:class` in `config/container.php`:

```php
<?php

use Psr\Container\ContainerInterface;
use Slim\Views\PhpRenderer;
// ...

return [

    // ...

    PhpRenderer::class => function (ContainerInterface $container) {
        return new PhpRenderer($container->get('settings')['view']['path']);
    },

];

```

## Helper functions

The OWASP Top 10 web security risks study lists Cross-Site Scripting (XSS) in second place.

Although XSS is one of the trivial ways of exploiting a web page 
it is the most common vulnerability but very serious. It can lead to identity theft and so on. 
The best defense is consistent escaping of printed data, ie. converting the characters 
which have a special meaning in the given context.
If the developer omits the escaping a security hole is made.

Note that `slim/PHP-View` has no built-in mitigation from XSS attacks. It is the developer's responsibility to
use `htmlspecialchars()` or a component like `laminas-escaper`. In this case I want to keep it lean and we just add this
simple but efficient html encoding function to our application instead.

Create a new directory, if not exists: `src/Util`

Then create new file `src/Util/functions.php` and copy paste this content:

```php
<?php

/**
 * Convert all applicable characters to HTML entities.
 *
 * @param string $text The string
 *
 * @return string The html encoded string
 */
function html(string $text): string
{
    return htmlspecialchars($text, ENT_QUOTES | ENT_SUBSTITUTE, 'UTF-8');
}

```

Open `composer.json` and add the `files` entry under the `autoload` property:

```
"files": [
    "src/Util/functions.php"
]
```

The result may look like this:

```
"autoload": {
    "psr-4": {
        "App\\": "src/"
    },
    "files": [
        "src/Util/functions.php"
    ]
},
```

Then run `composer dump-autoload` to update the autoloader.

## Usage

Create a new view template: `templates/home.php`

```php
Hello <?= html($name) ?>
```

This Action class shows how to inject the PhpRenderer to render an template:

```php
<?php

namespace App\Action;

use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface;
use Slim\Views\PhpRenderer;

final class HomeAction
{
    /**
     * @var PhpRenderer
     */
    private $renderer;

    public function __construct(PhpRenderer $renderer)
    {
        $this->renderer = $renderer;
    }

    public function __invoke(
        ServerRequestInterface $request,
        ResponseInterface $response
    ): ResponseInterface {
        return $this->renderer->render($response, 'home.php', ['name' => 'World']);
    }
}
```

### Layouts

The simplest form would be to include recurring elements such as the page header, 
navigation and footer, as a ready-made layout template. 

Create a new layout file: `templates/layout.php` and copy/paste this content:

**Example:**

```php
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>Slim Tutorial</title>
</head>
<body>
<?= $content ?>
</body>
</html>
```

To render the content into the layout template use the `$content` variable.
The `$content` variable is special variable used inside layouts to render the
wrapped view and should not be set in your view data.

To render the `home.php` template into the `layout.php` template you have 
to set the layout template in your action:

```php
$this->renderer->setLayout('layout.php');

return $this->renderer->render($response, 'home.php', ['name' => 'World']);
```

Now the response should contain the layout and the content of the rendered home template:

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>Slim Tutorial</title>
</head>
<body>
Hello World</body>
</html>
```

Please consult the **[documentation](https://github.com/slimphp/PHP-View#template-variables)**
to learn more about layout templates and global templates variables.

## Translations

For creating international websites and applications, PHP offers a wide range of possibilities. 
In this article I use the classic "gettext" PHP extension to keep the dependencies as low as possible.

The directory for all `*.mo` and `*.po` translation files is: `resources/text/<locale>/LC_MESSAGES/`

The source language is always english, so you don't need a translation file for english.

### Translator requirements

* The PHP [Gettext](https://www.php.net/manual/en/book.gettext.php) extension
* [Poedit](https://poedit.net/download) (A free translation editor)

### Creating a new PO file

The correct directory for the *.mo and *.po files is very imported for the `gettext` PHP extension. Create a directory
structure **exactly** like this:

```
resources/text/de_DE/LC_MESSAGES/
```

* Open Poedit and create a file po-file: `File > New`
* Select the target language, e.g. `German (Germany)`
* Click `File > Save`
* Enter a filename in this format: `<domain>_<locale>.po`
    * Example: `resources/text/de_DE/LC_MESSAGES/messages_de_DE.po`

\newpage
* Open the menu: `Catalogue` > `Properties...`
* Open the tab `Sources path` and add the `src` and the `templates` directory to the parser.

![image](https://user-images.githubusercontent.com/781074/101632581-6570ed80-3a26-11eb-9ceb-955fa51611fc.png)

\newpage

* Open the tab `Sources keywords` and add the `__` as additional keyword.

![image](https://user-images.githubusercontent.com/781074/101632611-70c41900-3a26-11eb-9511-abf24449abbf.png)

Click `OK` and save the file.

\newpage

### Translation helper

To provide a convenient way to retrieve strings in various languages, 
allowing you to easily support multiple languages
within your application, we add some helper functions.

Copy the following functions into your file: `src/Util/functions.php`:

```php
/**
 * Set locale
 *
 * @param string $locale The locale (en_US)
 * @param string $domain The text domain (messages)
 *
 * @throws UnexpectedValueException
 *
 * @retrun void
 */
function set_locale(string $locale, string $domain = 'messages'): void
{
    $codeset = 'UTF-8';
    $directory = __DIR__ . '/../../resources/text';

    // Set locale information
    setlocale(LC_ALL, $locale);

    // Check for extisting mo file (optional)
    $file = sprintf('%s/%s/LC_MESSAGES/%s_%s.mo', $directory, $locale, $domain, $locale);

    if ($locale !== 'en_US' && !file_exists($file)) {
        throw new UnexpectedValueException(sprintf('File not found: %s', $file));
    }

    // Generate new text domain
    $textDomain = sprintf('%s_%s', $domain, $locale);

    // Set base directory for all locales
    bindtextdomain($textDomain, $directory);

    // Set domain codeset
    bind_textdomain_codeset($textDomain, $codeset);

    textdomain($textDomain);
}

/**
 * Text translation.
 *
 * @param string $message The message
 * @param string|int|float|bool ...$context The context
 *
 * @return string The translated string
 */
function __(string $message, ...$context): string
{
    $translated = gettext($message);

    if (!empty($context)) {
        $translated = vsprintf($translated, $context);
    }

    return $translated;
}
```

### Translations in PHP

You can translate messages using the global `__()` helper function.

```php
echo __('I love programming.');
```

To translate a message with a placeholder use the [sprintf](https://www.php.net/manual/en/function.sprintf.php) syntax:

```php
echo __('There are %s users logged in.', 7);
```

### Translations in templates

You can use the same `__` function to translate messages in PHP templates.

```php
<?= html(__('There are %s users logged in.', 7)) ?>
```

**Example:**

```php
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title><?= html($title ?? 'Default title') ?></title>
</head>
<body>
<?= html(__('There are %s users logged in.', 7)) ?>
</body>
</html>
```

### Updating translation strings

To update a PO file, open it in Poedit and click `Update from code`
to insert or update the new messages. Then translate it and save the file.

### Determining the current locale

You can call setlocale like so, and it'll return the current local.

```php
$currentLocale = setlocale(LC_ALL, 0);
```

### Gettext pitfalls

You should know that the translations are cached until you restart the Apache web server. In a production environment
this is quite good for performance reason.

## URL helper

The [Slim Twig-View](https://github.com/slimphp/Twig-View#custom-template-functions) component
provides some useful template function like `url_for()`, `current_url()` and `base_path()` etc.
Unfortunately these functions are not available for the PHP-View component.

To extend the PHP-View template enginge we add a middleware as follows.

Create a new file: `src/Middleware/PhpViewExtensionMiddleware.php` and copy/paste this code:

```php
<?php

namespace App\Middleware;

use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface;
use Psr\Http\Server\MiddlewareInterface;
use Psr\Http\Server\RequestHandlerInterface;
use Slim\App;
use Slim\Views\PhpRenderer;

final class PhpViewExtensionMiddleware implements MiddlewareInterface
{
    /**
     * @var App
     */
    private $app;

    /**
     * @var PhpRenderer
     */
    private $phpRenderer;

    public function __construct(App $app, PhpRenderer $phpRenderer)
    {
        $this->phpRenderer = $phpRenderer;
        $this->app = $app;
    }

    public function process(
        ServerRequestInterface $request,
        RequestHandlerInterface $handler
    ): ResponseInterface {
        $this->phpRenderer->addAttribute('uri', $request->getUri());
        $this->phpRenderer->addAttribute('basePath', $this->app->getBasePath());
        $this->phpRenderer->addAttribute('route', $this->app->getRouteCollector()->getRouteParser());

        return $handler->handle($request);
    }
}

```

Register the middleware in `config/middleware.php`:

```php
<?php

use App\Middleware\PhpViewExtensionMiddleware;
use Slim\App;

return function (App $app) {
    // ...

    $app->add(PhpViewExtensionMiddleware::class);

    // ...
};
```

The `PhpViewExtensionMiddleware` adds the following template variables:

* `$basePath` - The current base path as string
* `$route` - The current Slim RouteParserInterface object from the incoming ServerRequestInterface object
* `$uri` - The PSR-7 UriInterface object from the incoming ServerRequestInterface object

### Template usage

```php
<?php

/* @var string $basePath */
/* @var \Slim\Interfaces\RouteParserInterface $route */
/* @var \Psr\Http\Message\UriInterface $uri */

// Output the base path
echo $basePath;

// Output the URL for a given route. e.g.: /
echo $route->urlFor('home');

// Output the URL for a given route. e.g.: https://www.example.com/hello/world
echo $route->fullUrlFor($uri, 'hello', ['name' => 'world']);

// Output the path for a named route, excluding the base path
echo $route->relativeUrlFor('home');

// The PSR-7 UriInterface object from the incoming ServerRequestInterface object
echo $uri->getPath();

```

## Assets

To inform the browser where to find the assets (js, css, images) we add a `<base>` tag inside the `<head>` element of the document.
The `<base>` tag specifies the base URL for all relative URLs in a document.
In this case we put the `$basePath` into the base `href` attribute.

Create or modify your existing layout file: `templates/layout.php` using this content:

**Example:**

```php
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title><?= html($title ?? 'Slim Tutorial') ?></title>
    <base href="<?= $basePath ?>/"/>
    <script type="text/javascript" src="js/hello.js"></script>
</head>
<body>
<?= $content ?>
</body>
</html>
```

The browser then downloads the assets relative to the base path, e.g. from `/slim4-tutorial/js/hello.js`.

Create a simple JavaScript file in `public/js/hello.js`:

```js
document.addEventListener('DOMContentLoaded', function () {
  alert('The DOM is ready');
});
```

Modify your HomeAction class in `src/Action/HomeAction.php` to render the layout template:

```php
<?php

namespace App\Action;

use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface;
use Slim\Views\PhpRenderer;

final class HomeAction
{
    /**
     * @var PhpRenderer
     */
    private $renderer;

    public function __construct(PhpRenderer $renderer)
    {
        $this->renderer = $renderer;
    }

    public function __invoke(
        ServerRequestInterface $request, 
        ResponseInterface $response
    ): ResponseInterface {
        $this->renderer->setLayout('layout.php');

        // optional
        $this->renderer->addAttribute('title', 'My Slim Application');

        return $this->renderer->render($response, 'home.php', ['name' => 'World']);
    }
}
```

**Result:**

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <title>My Slim Application</title>
  <base href="/slim4-tutorial/"/>
  <script type="text/javascript" src="js/hello.js"></script>
</head>
<body>
Hello World
</body>
</html>
```

### Bundling assets

You should only use minified assets on your **production** server to speed up performance (SEO). 
To bundle assets, you can add [Webpack](https://webpack.js.org/) to your build process.
A webpack script could, for example, recursively traverse the JS and CSS directories and minify its content.

Example: `package.json`

```
{
    "scripts": {
        "build": "npx webpack --mode=production",
        "build:dev": "npx webpack --mode=development"
    },
    "devDependencies": {
        "autoprefixer": "^9.8.6",
        "css-loader": "^3.6.0",
        "file-loader": "^4.3.0",
        "mini-css-extract-plugin": "^0.8.2",
        "optimize-css-assets-webpack-plugin": "^5.0.4",
        "postcss-loader": "^3.0.0",
        "terser-webpack-plugin": "^2.3.8",
        "webpack": "^4.44.2",
        "webpack-assets-manifest": "^3.1.1",
        "webpack-cli": "^3.3.12",
        "webpack-manifest-plugin": "^2.2.0"
    }
}
```

Example: `webpack.config.js`

```js
const path = require('path');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');
const OptimizeCSSAssetsPlugin = require('optimize-css-assets-webpack-plugin');
const TerserJSPlugin = require('terser-webpack-plugin');

module.exports = (env, argv) => ({
    entry: function loadEntries(pattern) {
        const glob = require('glob');
        const path = require('path');
        const object = {};

        glob.sync(pattern).forEach(function (match) {
            const ext = path.extname(match);

            if (!['.js', '.css'].includes(ext)) {
                return;
            }

            const basename = path.basename(match, ext);
            const onlyPath = path.dirname(match);
            const key = onlyPath.replace('\.\/public\/', '');

            object[key + '/' + basename] = match;
        });

        return object;
    }('./public/**'),

    output: {
        path: path.resolve(__dirname, 'public'),
        publicPath: '/',
    },

    mode: 'development',

    optimization: {
        minimizer: [new TerserJSPlugin({}), new OptimizeCSSAssetsPlugin({})],
    },

    module: {
        rules: [
            {
                test: /\.css$/,
                use: [MiniCssExtractPlugin.loader, 'css-loader']
            },
        ],
    },

    plugins: [
        new MiniCssExtractPlugin({
            ignoreOrder: false
        }),
    ],
});

```

To minify all assets for production run: `npm run build`

If you are looking for a simpler way, you may
take a look at **[Apache ant](https://stackoverflow.com/a/1498830/1461181)**
to automate Javascript / CSS minification.

## Similar components

### Plates

[Plates](https://platesphp.com/) is a native PHP template system that’s fast, easy to use and easy to extend. 
It’s inspired by the excellent Twig template engine and strives to bring modern template 
language functionality to native PHP templates. Plates is designed for developers who 
prefer to use native PHP templates over compiled template languages, such as Twig or Smarty.

Please note, that Plates is one of the slowest template engines. See PHP Templating Performance:

![image](https://user-images.githubusercontent.com/781074/103140997-5ceb0900-46ee-11eb-8ae0-80d82be63004.png)

*Image source:* <https://medium.com/@gotzmann/the-fastest-template-engine-for-php-b5b461b46296>

My personal impression is that Plates is not so actively maintained and supported anymore.
Version 4 is in Alpha since 2018. 

**Update**: Plates v4 Alpha has been abandoned. Version 3.4.0 supports PHP 8.

### Twig

Twig is one of the most popular template engine out there.
Twig has a special syntax which is compiled and cached at runtime to native PHP code.
There are two kinds of delimiters: {% raw %}`{% ... %}`{% endraw %} and {% raw %}`{{ ... }}`{% endraw %}.
The first one is used to execute statements such as for-loops, the latter outputs the result of an expression.

![Twig](https://user-images.githubusercontent.com/781074/103141434-8e66d300-46f4-11eb-9e04-0265a00906d2.png)

One of the main reasons for using Twig is security and the ability to NOT put complex (business) logic
into a template file.
There are two approaches: manually escaping each variable or automatically escaping everything.
Twig supports both, [automatic escaping](https://twig.symfony.com/doc/3.x/templates.html#html-escaping)
is enabled by default for secure reasons. Twig can be extended using "Extensions". 
Twig requires a much more complex initial setup and special write permissions on the servers filesystem.

### Symfony Templating

There was also the native [Symfony Templating](https://github.com/symfony/templating) component
that provides a lot of tools needed to build any kind of template system.
But since Symfony 5.0 it's no longer supported.

### Mustache.php

[Mustache](https://github.com/bobthecow/mustache.php) is a simple, logic-less template engine.

They call it "logic-less" because there are no `if` statements, `else` clauses, or `for` loops. 
Instead there are only tags (placeholder). Some tags are replaced with a value, some with nothing, 
and others with a series of values.

Currently there is only a [framework integration for Slim 3](https://github.com/andrewslince/slim3-mustache-view) 
availabe, but not for Slim 4.

### Smarty

Smarty is a template engine which actually compiles the template file to the php file that 
can be later executed. This simply saves time on parsing and variable outputs, 
beating other Template Engines with much smaller memory use and regex.
Slim 4 support will be available soon in the [mathmarques/smarty-view](https://github.com/mathmarques/Smarty-View) component.

### Latte (Nette)

Latte is a template engine for PHP which eases your work and ensures the output is protected against vulnerabilities, 
such as XSS. It's the first PHP engine introducing context-aware escaping and link checking.
Latte also works with Slim 4, e.g. [LatteView](https://github.com/ujpef/latte-view/blob/master/src/LatteView.php#L57)

### Laminas-View 

The Laminas-View [PhpRenderer](https://docs.laminas.dev/laminas-view/php-renderer/) renders view scripts written in PHP, 
capturing and returning the output. You can use helper, or plugin, classes for 
[html escaping](https://docs.laminas.dev/laminas-view/helpers/escape/#escapehtml),
date formatting, generating form elements, or displaying action links.
The [Asset helper](https://docs.laminas.dev/laminas-view/helpers/asset/) can be used to map asset 
names to versioned assets. The Laminas-View component looks very complex, 
brings a lot of other dependencies, but looks very modern and promising.

### Conclusion

At the end of the day, I would say that the `slim/twig-view` component makes 
the most sense if you are using Slim because it's maintained and the built-in support 
for the PSR-7 request and response objects makes it much easier to use.