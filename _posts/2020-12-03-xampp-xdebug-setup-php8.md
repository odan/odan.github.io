---
title: XAMPP - XDebug Setup for PHP 8
layout: post
comments: true
published: true
keywords: php, xdebug, php8, debugger
---

## Requirements

* XAMPP for Windows: <https://www.apachefriends.org/download.html>
* [Microsoft Visual C++ Redistributable for Visual Studio 2015-2019](https://aka.ms/vs/16/release/vc_redist.x64.exe)

## Setup

* Download Xdebug for the specific PHP version:
  * PHP 8.0 (64-Bit): <https://xdebug.org/files/php_xdebug-3.1.2-8.0-vs16-x86_64.dll>
  * PHP 8.1 (64-Bit): <https://xdebug.org/files/php_xdebug-3.1.2-8.1-vs16-x86_64.dll>
* Move the downloaded dll file to: `C:\xampp\php\ext`
* Rename the dll file to: `php_xdebug.dll`
* Open the file `C:\xampp\php\php.ini` with Notepad++
* Disable output buffering: `output_buffering = Off`
* Scroll down to the `[XDebug]` section (or create it) and copy/paste these lines:

```ini
[XDebug]
zend_extension=xdebug
xdebug.mode=debug
xdebug.start_with_request=trigger
```

* Restart Apache

## PhpStorm

* Enable the Xdebug option: "Can accept external connections" and "Additionally listen on Xdebug 3 default port 9003". 
  [Screenshot](https://user-images.githubusercontent.com/781074/103152131-2e6c3d00-4785-11eb-8680-bdeb886e8bfa.png)
* Use the [PhpStorm bookmarklets generator](https://www.jetbrains.com/phpstorm/marklets/) to activate Xdebug from the browser side.

## Netbeans

* Change the Netbeans debugging options: [Screenshot](https://user-images.githubusercontent.com/781074/101086805-ae9de900-35b1-11eb-9c4d-a6a7aaaf3279.png)

## Visual Studio Code

* [Installing XDebug on anything for VSCode in 5 minutes](https://technex.us/2020/06/installing-xdebug-on-anything-for-vscode-in-5-minutes/)
* Install the [PHP Debug Adapter for Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=felixfbecker.php-debug).
* [Debug PHP In VSCode With XDebug](https://www.codewall.co.uk/debug-php-in-vscode-with-xdebug/)

## Postman

Add `XDEBUG_SESSION_START=PHPSTORM` as query parameter to the url, e.g.

* http://localhost?XDEBUG_SESSION_START=PHPSTORM

## Starting the debugger from the console

Enter cmd:

```
set XDEBUG_CONFIG="idekey=xdebug"
php test.php
```

## Known Issues

### Can't locate API module structure 'php8_module'
 
Read more: <https://community.apachefriends.org/f/viewtopic.php?f=16&t=80105#p270731>

### Xdebug: Step Debug Time-out connecting to debugging client, waited: 200 ms. Tried: localhost:9003 (through xdebug.client_host/xdebug.client_port) :-(

Change `xdebug.start_with_request=yes` to `xdebug.start_with_request=trigger`

### Warning: XDEBUG_MODE=coverage or xdebug.mode=coverage has to be set

If you change the php.ini setting from `xdebug.mode=debug` to `xdebug.mode=coverage` 
it should work. But then it is not possible to trigger the debugger in the browser for PhpStorm. 

Instead, you can run phpunit from the console with this command:

```
php -d xdebug.mode=coverage -r "require 'vendor/bin/phpunit';" -- --configuration phpunit.xml --do-not-cache-result --coverage-clover build/logs/clover.xml --coverage-html build/coverage
```

This is how you can use the same command in a `composer.json` script:

```json
"test:coverage": "php -d xdebug.mode=coverage -r \"require 'vendor/bin/phpunit';\" -- --configuration phpunit.xml --do-not-cache-result --coverage-clover build/logs/clover.xml --coverage-html build/coverage"
```

The `php -d` command line option defines the php.ini entry `xdebug.mode` with value `coverage`.

Then it starts the phpunit php script and passes the console arguments after the additional `--` separator.
This works on Linux and on Windows.
