---
title: Slim 4 - PHP-View
layout: post comments: false
published: false
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
    * [Gettext fitfalls](#gettext-fitfalls)

###       

## Requirements

* PHP 7.2+
* [A Slim 4 application](https://odan.github.io/2019/11/05/slim4-tutorial.html)

## Introduction

## Installation

To install php-view, run:

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

Note that `slim/PHP-View` has no built-in mitigation from XSS attacks. It is the developer's responsibility to
use `htmlspecialchars()` or a component like `laminas-escaper`. In this case I want to keep it lean and we just add this
simple but efficent html encoding function to our application instead.

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
Hello <?=html($name)?>
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

* Open the menu: `Catalogue` > `Properties...`

![image](https://user-images.githubusercontent.com/781074/101627114-276fcb80-3a1e-11eb-8f2d-d3c7a9fe4b99.png)

Open the tab `Sources path` and add the `src` and the `templates` directory to the parser.

![image](https://user-images.githubusercontent.com/781074/101632581-6570ed80-3a26-11eb-9ceb-955fa51611fc.png)

Open the tab `Sources keywords` and add the `__` as additional keyword.

![image](https://user-images.githubusercontent.com/781074/101632611-70c41900-3a26-11eb-9511-abf24449abbf.png)

Click `OK` and save the file.

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
 *
 * <code>
 * echo __('Hello');
 * echo __('There are %s persons logged in', 7);
 * </code>
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

### Updating translation strings

To update a PO file, open it in Poedit and click `Update from code`
to insert or update the new messages. Then translate it and save the file.

### Determining the current locale

You can call setlocale like so, and it'll return the current local.

```php
$currentLocale = setlocale(LC_ALL, 0);
```

### Gettext fitfalls

You should know that the translations are cached until you restart the Apache web server. In a production environment
this is quite good for performance reason. A more developer friendly solution would be the Symfony Translation
Component.
