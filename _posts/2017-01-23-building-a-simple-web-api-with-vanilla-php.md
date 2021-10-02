---
title: Building a Simple Web API with Vanilla PHP
layout: post
comments: true
published: false
description: 
keywords: 
---

This article is inspired by Brad Cypert's blog post: <https://www.bradcypert.com/building-a-simple-api-in-php-using-lumen/>.

Why should we build a API with Vanilla PHP? Because you can, and it makes fun.

## Requirements

* PHP

## Let's go

* Create a directory: `/src`

* Create a file: `src/functions.php`

```php
function route($httpMethods, $route, $callback, $exit = true)
{
    static $path = null;
    if ($path === null) {
        $path = parse_url($_SERVER['REQUEST_URI'])['path'];
        $scriptName = dirname(dirname($_SERVER['SCRIPT_NAME']));
        $scriptName = str_replace('\\', '/', $scriptName);
        $len = strlen($scriptName);
        if ($len > 0 && $scriptName !== '/') {
            $path = substr($path, $len);
        }
    }
    if (!in_array($_SERVER['REQUEST_METHOD'], (array) $httpMethods)) {
        return;
    }
    $matches = null;
    $regex = '/' . str_replace('/', '\/', $route) . '/';
    if (!preg_match_all($regex, $path, $matches)) {
        return;
    }
    if (empty($matches)) {
        $callback();
    } else {
        $params = array();
        foreach ($matches as $k => $v) {
            if (!is_numeric($k) && !isset($v[1])) {
                $params[$k] = $v[0];
            }
        }
        $callback($params);
    }
    if ($exit) {
        exit;
    }
}
```

* Create a file `.htaccess` in the project root directory.

```htaccess
RewriteEngine on
RewriteRule ^$ public/ [L]
RewriteRule (.*) public/$1 [L]
```

* Create a directory: `/public`
* Create a file: `public/.htaccess`

```
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-d
RewriteCond %{REQUEST_FILENAME} !-f
RewriteRule ^ index.php [QSA,L]
```

* Create a front controller file: `public/index.php`

```php
<?php

require_once __DIR__ . '/../src/functions.php';

// Default index page
route('GET', '^/$', function() {
    echo '<a href="users">List users</a><br>';
});

// GET request to /users
route('GET', '^/users$', function() {
    echo '<a href="users/1000">Show user: 1000</a>';
});

// With named parameters
route('GET', '^/users/(?<id>\d+)$', function($params) {
    echo "You selected User-ID: ";
    var_dump($params);
});

// POST request to /users
route('POST', '^/users$', function() {
    // Send a json response
    header('Content-Type: application/json');
    $json = json_decode(file_get_contents('php://input'), true);
    echo json_encode(['success' => true]);
});

header('HTTP/1.0 404 Not Found');
echo '404 Not Found';

```

And that’s it! Now open the website and navigate through the pages.

Have fun :-)

> **Attention:** Please do not use this funny example in a production system.<br>
> A better approach would be to use the [Slim framework](https://www.slimframework.com/)
