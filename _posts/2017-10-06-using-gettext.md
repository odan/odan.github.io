---
title: Using gettext
layout: post
comments: true
published: false
description: 
keywords: 
---

### Requirements

* PHP with the `php_gettext` extension
* [Poedit](https://poedit.net/)

### Introduction

For creating international websites and applications, PHP offers a wide range of possibilities. On this page I introduce the old and classic "gettext" library.

### Examples

The directory is very imported. Create a directory structure like this:

Format:

```
locale/<locale>/LC_MESSAGES/
```

Example:
```
locale/de_DE/LC_MESSAGES/
```

Then create the .po and .mo files with PoEdit.

Format:

```
<domain>_<locale>.po
<domain>_<locale>.mo

```

Example:

```
locale/de_DE/LC_MESSAGES/messages_de_DE.mo
locale/de_DE/LC_MESSAGES/messages_de_DE.po
```

Then try this example php code to see that it works:

```php
<?php

$locale = 'de_DE';
//$locale = 'fr_CH';
$domain = 'messages';
$codeset = 'UTF-8';
$directory = __DIR__ . '/locale';

// Activate the locale settings
putenv('LC_ALL=' . $locale);
setlocale(LC_ALL, $locale);

// Debugging output 
$file = sprintf('%s/%s/LC_MESSAGES/%s_%s.mo', $directory, $locale, $domain, $locale);
echo $file . "\n";

// Generate new text domain
$textDomain = sprintf('%s_%s', $domain, $locale);

// Set base directory for all locales
bindtextdomain($textDomain, $directory);

// Set domain codeset (optional)
bind_textdomain_codeset($textDomain, $codeset);

// File: ./locale/de_DE/LC_MESSAGES/messages_de_DE.mo
textdomain($textDomain);

// Test a translation
echo _('Yes'); // Ja
```


### Summary

You should know that the translations are cached until you restart the Apache web server. 

In a production environment this is quite good for performance reason, but is not as good at development. 

A more developer friendly solution would be the Symfony Translation Component.