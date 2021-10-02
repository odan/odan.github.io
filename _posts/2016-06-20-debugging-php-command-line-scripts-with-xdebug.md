---
title: Debugging php command line scripts with xdebug
layout: post
comments: true
published: false
description: 
keywords: 
---

## Setup

### php.ini

* Set remote_host to localhost.

```ini
[XDebug]
xdebug.remote_host = "localhost"
```

## PhpStorm

* Set line breakpoint (Ctrl+F8)
* Menu: Run > Start listening for PHP  Debug connections
* Open the Terminal (Alt+F12) and enter:
* SET XDEBUG_CONFIG=idekey=PHPSTORM
* Start the script: php myscript.php

## Netbeans

* Netbeans setup: <https://postimg.cc/PvbtT8dz>
* Set breakpoint
* Start Debug project (Ctrl+F5)
* Open console (cmd) and enter:
* SET XDEBUG_CONFIG=idekey=netbeans-xdebug
* Start the script: php test.php
