---
title: XAMPP - How to enable PHP OPCache
layout: post
comments: false
published: true
description: 
keywords: php, opcache, xampp
---

XAMPP comes with PHP OPCache already included in the bundle, you just need to enable it. To enable the extension:

* Open the file with Notepad++: `c:\xampp\php\php.ini`

* Add this line near the extension section: `zend_extension=php_opcache.dll`

* Stop/Start Apache