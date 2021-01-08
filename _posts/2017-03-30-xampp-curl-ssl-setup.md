---
title: XAMPP - CURL SSL Setup
layout: post
comments: false
published: true
description: 
keywords: PHP, CURL, SSL, Twitter, CURLOPT_SSL_VERIFYPEER
---

To fix the SSL certificate error message "SSL certificate error: unable to get local issuer certificate" try this:

* Download: <http://curl.haxx.se/ca/cacert.pem>
* Copy the file `cacert.pem` into the directory: C:\xampp\php\extras\ssl\
* Open `c:\xampp\php\php.ini` with Notepad++
* Search for the `[curl]` section
* Add the following lines:

 ```ini
[curl]
; A default value for the CURLOPT_CAINFO option.
; This must be an absolute path.
curl.cainfo="C:\xampp\php\extras\ssl\cacert.pem"
openssl.cafile="C:\xampp\php\extras\ssl\cacert.pem"
 ```
 
* Restart apache
