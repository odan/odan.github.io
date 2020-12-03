---
title: XAMPP - XDebug Setup for PHP 8
layout: post
comments: true
published: true
keywords: php xdebug php8 debugger
---

## Requirements

* XAMPP for Windows: <https://www.apachefriends.org/download.html>
* [Microsoft Visual C++ Redistributable for Visual Studio 2015-2019](https://aka.ms/vs/16/release/vc_redist.x64.exe)

## Setup

If the file `C:\xampp\php\ext\php_xdebug.dll` already exists, you can skip the download.

* Download Xdebug for the specific PHP version:
  * PHP 8.0 (64-Bit): <https://xdebug.org/files/php_xdebug-3.0.0-8.0-vs16-x86_64.dll>
* Move the downloaded dll file to: `C:\xampp\php\ext`
* Open the file `C:\xampp\php\php.ini` with Notepad++
* Disable output buffering: `output_buffering = Off`
* Scroll down to the `[XDebug]` section (or create it) and copy/paste these lines:

```ini
[XDebug]
zend_extension = "c:\xampp\php\ext\php_xdebug-3.0.0-8.0-vs16-x86_64.dll"
xdebug.mode=debug
xdebug.start_with_request=yes
```

* Restart Apache

## PhpStorm

* Change the Debug port from 9000 to 9003. [Screenshot](https://user-images.githubusercontent.com/781074/101086364-0daf2e00-35b1-11eb-89d3-87647d619fb1.png)
* Use the [PhpStorm bookmarklets generator](https://www.jetbrains.com/phpstorm/marklets/) to activate Xdebug from the browser side.
* Enable the Xdebug option: "Can accept external connections". See [screenshot](https://user-images.githubusercontent.com/781074/83182499-ba9e7f00-a126-11ea-88c0-f28d0cbff260.png)

## Netbeans

* Change the Netbeans debugging options: [Screenshot](https://user-images.githubusercontent.com/781074/101086805-ae9de900-35b1-11eb-9c4d-a6a7aaaf3279.png)

## Visual Studio Code

* [Installing XDebug on anything for VSCode in 5 minutes](https://technex.us/2020/06/installing-xdebug-on-anything-for-vscode-in-5-minutes/)
* Install the [PHP Debug Adapter for Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=felixfbecker.php-debug).
* [Debug PHP In VSCode With XDebug](https://www.codewall.co.uk/debug-php-in-vscode-with-xdebug/)

## Sublime Text 2 and 3

* Install the [Xdebug Client Package](https://packagecontrol.io/packages/Xdebug%20Client)

## Start debugger from the console

Enter cmd:

```
set XDEBUG_CONFIG="idekey=xdebug"
php test.php
```

## Postman

Add `XDEBUG_SESSION_START=PHPSTORM` as query parameter to the url, e.g.

* http://localhost?XDEBUG_SESSION_START=PHPSTORM

## Known Issues

* [Can't locate API module structure 'php8_module'](https://community.apachefriends.org/f/viewtopic.php?f=16&t=80105#p270731)
