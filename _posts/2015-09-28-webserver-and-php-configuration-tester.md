---
title: Webserver and PHP configuration tester
layout: post
comments: true
published: false
description: 
keywords: 
---

```php
<?php

/**
 * Webserver and PHP configuration tester
 *
 * @author odan
 * @license MIT
 */

if (!isset($_SESSION)) {
    session_name('phptest');
    session_start();
}

if (php_sapi_name() === 'cli') {
    echo "\n" . 'Webserver Test' . "\n\n";
} else {
    echo '<html><body style="font-family: monospace, monospace">' . "\n";
}

write('Time', date('Y-m-d H:i:s'));
//write('Host', $_SERVER['HTTP_HOST']);

if (version_compare(PHP_VERSION, '5.6.0') >= 0) {
    write("PHP Version " . PHP_VERSION, "OK");
} else {
    write("PHP Version " . PHP_VERSION, "ERROR: Not supported");
}

// detect apache mode_rewrite
ob_start();
phpinfo(INFO_MODULES);
$contents = ob_get_contents();
ob_end_clean();
$hasModRewrite = strpos($contents, 'mod_rewrite') !== false;

// apache mode_rewrite fallback
if (!$hasModRewrite && file_exists('/etc/apache2/mods-enabled/')) {
   $hasModRewrite = strpos(shell_exec('ls /etc/apache2/mods-enabled/'), 'rewrite') !== false;
}

// apache mode_rewrite fallback for windows (xampp)
if (!$hasModRewrite && file_exists('c:/xampp/apache/bin/httpd.exe')) {
    $hasModRewrite = strpos(shell_exec('c:/xampp/apache/bin/httpd.exe -M'), 'rewrite_module') !== false;
}

if ($hasModRewrite) {
    write("Apache mod_rewrite is installed", "OK");
} else {
    write("Apache mod_rewrite is not installed", "ERROR");
}

$memoryLimit = (int)ini_get('memory_limit');
if ((int)$memoryLimit >= '128' || (int)$memoryLimit == -1) {
    write("Memory limit ($memoryLimit)", "OK");
} else {
    write("Memory limit ($memoryLimit)", "Warning: min. 128 M");
}

$postMaxSize = ini_get('post_max_size');
if ((int)$postMaxSize >= '8') {
    write("POST max size ($postMaxSize)", "OK");
} else {
    write("POST max size ($postMaxSize)", "Warning: min. 8 M");
}

$uploadMaxFilesize = ini_get('upload_max_filesize');
if ((int)$uploadMaxFilesize >= 'w') {
    write("Upload max filesize ($uploadMaxFilesize)", "OK");
} else {
    write("Upload max filesize ($uploadMaxFilesize)", "Warning: min. 2 M");
}

$suhosin = extension_loaded('suhosin');
if (!$suhosin) {
    write("Suhosin extension is not installed", "OK");
} else {
    write("Suhosin extension is installed", "WARNING: Please remove the suhosin extension");
}

// session check
if (!isset($_SESSION['counter'])) {
    $_SESSION['counter'] = 0;
}
$_SESSION['counter']++;

if ($_SESSION['counter'] > 0) {
    write("Session", "OK");
} else {
    write("Session", "Error: Not correct installed");
}

function write($key, $value)
{
    echo str_pad($key . ' ', 45, '.') . ' ' . $value;
    echo php_sapi_name() === 'cli' ? "\n" : "<br>\n";
}

if (php_sapi_name() !== 'cli') {
    echo '</body></html>';
}
```

