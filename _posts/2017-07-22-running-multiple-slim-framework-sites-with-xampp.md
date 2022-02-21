---
title: Running multiple Slim Framework sites with XAMPP
layout: post
comments: true
published: false
description: 
keywords: 
---

[Slim 3 cannot work in a subdirectory](https://github.com/slimphp/Slim/issues/1529) by default. 

Here I will show you a simple solution to run multiple Slim applications under Apache.

### Fixing the slim environment

The directory skeleton structure:

```
C:\xampp\htdocs\slim3-app
-- config
---- container.php
-- public
---- .htaccess
---- index.php
-- vendor
-- .htaccess
```

slim3-app/.htaccess

```
RewriteEngine on
RewriteRule ^$ public/ [L]
RewriteRule (.*) public/$1 [L]
```

slim3-app/config/container.php

Add this fix to the container:

```php
<?php

// ...

$container['environment'] = function () {
    $scriptName = $_SERVER['SCRIPT_NAME'];
    $_SERVER['REAL_SCRIPT_NAME'] = $scriptName;
    $_SERVER['SCRIPT_NAME'] = dirname(dirname($scriptName)) . '/' . basename($scriptName);
    return new \Slim\Http\Environment($_SERVER);
};
```

Now just open the application: http://localhost/slim3-app

## Links

* <https://github.com/slimphp/Slim/issues/2294>

### Keywords

Apache, XAMPP, PHP, Slim Framework