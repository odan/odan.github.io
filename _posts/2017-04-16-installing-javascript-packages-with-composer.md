---
title: Installing JavaScript packages with Composer
layout: post
comments: true
published: false
description: 
keywords: 
---

## Setup

Open `composer.json` and add the following lines to the `scripts` section:

```json
 "scripts": {
    "update-assets": "php config/composer.php update-assets",
    "post-update-cmd": "php config/composer.php post-update-cmd",
    "post-install-cmd": "php config/composer.php post-install-cmd"
  }
```

Create a new file `config/composer.php` with this content:

```php
<?php

$event = isset($argv[1]) ? $argv[1] : null;

if ($event == 'post-update-cmd') {
    // composer update
    update_assets();
}

if ($event == 'post-install-cmd') {
    // composer install
    update_assets();
}

if ($event == 'update-assets') {
    update_assets();
}

function update_assets() {
    echo "Update jQuery\n";
    file_put_contents(__DIR__ . '/../public/js/jquery.js', file_get_contents('https://code.jquery.com/jquery-3.2.1.js'));
    file_put_contents(__DIR__ . '/../public/js/jquery.min.js', file_get_contents('https://code.jquery.com/jquery-3.2.1.min.js'));
    file_put_contents(__DIR__ . '/../public/js/jquery.min.map', file_get_contents('https://code.jquery.com/jquery-3.2.1.min.map'));
    
    echo "Update mustache.js\n";
    file_put_contents(__DIR__ . '/../public/js/mustache.js', file_get_contents('https://raw.githubusercontent.com/janl/mustache.js/v2.3.0/mustache.js'));
    file_put_contents(__DIR__ . '/../public/js/mustache.min.js', file_get_contents('https://raw.githubusercontent.com/janl/mustache.js/v2.3.0/mustache.min.js'));
}
```

Now run `composer update` to update all PHP and JavaScript packages:

```shell
$ composer update
```

If you only want to update all JavaScript packages run: 

```shell
$ composer update-assets
```
You will find the JavaScript files under `public/js/*`.


Keywords: PHP, Composer, Assets, JavaScript, npm
