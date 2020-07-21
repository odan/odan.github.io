---
title: Word Automation with PHP
layout: post
comments: true
published: false
description: 
keywords: 
---

## Windows Server setup

Make sure to set the correct COM permission to "This User".

- Run: "dcomcnfg"
- Expand: Component Services > Computers > My Computer > DCOM Config.
- Select: "Microsoft Word 97-2003 Document"
- Right-Click on it and open the properties
- Click the Identity tab. 
- Select "This User" and type the username and password for the (Administrator) user.
- Apply these new settings and test your COM application.

If it's not working try to select "Interactive User". [Read more](http://www.verydoc.com/others/configure-word-and-excel.htm)

## PHP Setup

* You must install a 32-bit PHP build.
* Enable the COM extension in php.ini

> extension=php_com_dotnet.dll

* Restart Apache

## Usage

```php

try {
    $word = new COM("word.application");
} catch (Exception $ex) {
    echo $ex->getMessage();
    exit;
}

// Enable screenupdating (slower) but better pdf rendering results
$word->Screenupdating = true;

// Minimize window (0 - normal, 1 - fullscreen, 2 - minimized).
$word->WindowState = 2;

// Hide window
$word->Visible = 0;

// Disable language detection
$word->CheckLanguage = false;

// Disable overwrite mode
$word->Options->Overtype = false;

// Disable auto recovery
$word->Options->SaveInterval = 0;

// Disable the assistant
$word->Assistant->Visible = false;

// Disable alerts
$word->DisplayAlerts = false;

// Do fancy stuff...

// Close word
$word->Quit();
$word = null;
```