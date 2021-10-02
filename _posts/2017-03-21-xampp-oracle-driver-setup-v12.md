---
title: XAMPP - Oracle Driver Setup (v12)
layout: post
comments: true
published: true
description: 
keywords: 
---

Last update: 08.01.2020

## Requirements

This driver requires the Microsoft Visual C++ Redistributable. 
The redistributable can easily be downloaded on the Microsoft website as x86 or x64 edition. 

**Notice:** On a x64 computer the x86 **AND** the x64 version of the Visual C++ Redistributable setup must be installed.

[Visual C++ Redistributable for Visual Studio](https://support.microsoft.com/de-ch/help/2977003/the-latest-supported-visual-c-downloads)

This setup is tested with PHP (32-Bit): 5.6, 7.0, 7.1, 7.2, 7.3 and with PHP (x64) 64-Bit: 7.4

## Setup

* For PHP (**32-Bit**) open: [Instant Client Downloads for Microsoft Windows 32-bit](https://www.oracle.com/database/technologies/instant-client/microsoft-windows-32-downloads.html)
* For PHP (**x64, 64-Bit**) open: [Instant Client Downloads for Microsoft Windows (x64) 64-bit](https://www.oracle.com/database/technologies/instant-client/winx64-64-downloads.html)
* Click: "Accept License Agreement"
* Download and unzip the ZIP file `instantclient-basiclite-nt-12.2.0.1.0.zip`
* Copy all `*.dll` files: to `c:\xampp\php`
* Copy all `*.dll` files to `c:\xampp\apache\bin` (We need a second copy here for apache)
* Make sure that the file `c:\xampp\php\ext\php_oci8_12c.dll` exists.
* Enable php extension in php.ini: `extension=php_oci8_12c.dll` (For PHP 7.2+ use `extension=oci8_12c`)
* Restart Apache

## Known issues

Especially the `WAMP` users reported that they still get the following error message: 

`PHP Warning: PHP Startup: Unable to load dynamic library 'oci8_12c'`.

Make sure you downloaded the correct Oracle Instant Client, **32-bit** or **(x64) 64-bit**.

You can also try to download the correct php_oci8 dll files from this page:

* PHP 7.2 (32-bit): <http://windows.php.net/downloads/pecl/releases/oci8/2.2.0/php_oci8-2.2.0-7.2-ts-vc15-x86.zip>
* PHP 7.3 (32-bit): <https://windows.php.net/downloads/pecl/releases/oci8/2.2.0/php_oci8-2.2.0-7.3-ts-vc15-x86.zip>
* PHP 7.4 (x64, 64-Bit): <https://windows.php.net/downloads/pecl/releases/oci8/2.2.0/php_oci8-2.2.0-7.4-ts-vc15-x64.zip>
* [All releases](https://windows.php.net/downloads/pecl/releases/oci8/)


## Connection test
```php
<?php

error_reporting(E_ALL);

if (function_exists("oci_connect")) {
    echo "oci_connect found\n";
} else {
    echo "oci_connect not found\n";
    exit;
}

$host = 'localhost';
$port = '1521';

// Oracle service name (instance)
$db_name     = 'DBNAME';
$db_username = "USERNAME";
$db_password = "123456";

$tns = "(DESCRIPTION =
	(CONNECT_TIMEOUT=3)(RETRY_COUNT=0)
    (ADDRESS_LIST =
      (ADDRESS = (PROTOCOL = TCP)(HOST = $host)(PORT = $port))
    )
    (CONNECT_DATA =
      (SERVICE_NAME = $db_name)
    )
  )";
$tns = "$host:$port/$db_name";

try {
    $conn = oci_connect($db_username, $db_password, $tns);
    if (!$conn) {
        $e = oci_error();
        throw new Exception($e['message']);
    }
    echo "Connection OK\n";
    
    $stid = oci_parse($conn, 'SELECT * FROM ALL_TABLES');
    
    if (!$stid) {
        $e = oci_error($conn);
        throw new Exception($e['message']);
    }
    // Perform the logic of the query
    $r = oci_execute($stid);
    if (!$r) {
        $e = oci_error($stid);
        throw new Exception($e['message']);
    }
    
    // Fetch the results of the query
    while ($row = oci_fetch_array($stid, OCI_ASSOC + OCI_RETURN_NULLS)) {
        $row = array_change_key_case($row, CASE_LOWER);
        print_r($row);
        break;
    }
    
    // Close statement
    oci_free_statement($stid);
    
    // Disconnect
    oci_close($conn);
    
}
catch (Exception $e) {
    print_r($e);
}
```

### More infos and links

* <https://blogs.oracle.com/opal/installing-xampp-for-php-and-oracle-database>
