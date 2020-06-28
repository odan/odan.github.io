---
title: Installing an SSL Certificate under Apache (XAMPP)
layout: post
comments: true
published: true
description: 
keywords: 
---

## Requirements

* XAMPP for Windows

## Installation

Download and install the 32-Bit version of [OpenSSL](https://slproweb.com/products/Win32OpenSSL.html)

* OpenSSL 32-Bit: [Win32OpenSSL-1_1_0L.exe](https://slproweb.com/download/Win32OpenSSL-1_1_0L.exe)

## Configuration

* OpenSSL requires a configuration file: `openssl.cnf`
* Enter this command:

```
set OPENSSL_CONF=C:\OpenSSL-Win32\bin\openssl.cfg
```

## Create a certificate request file

Run

```
cd C:\OpenSSL-Win32\bin
openssl req -new -nodes -keyout www_example_com.key -out www_example_com.csr -newkey rsa:2048
```
* Reade more: <https://www.psw-group.de/support/?p=20/>

To create a [self-signed certificate with OpenSSL](https://stackoverflow.com/a/10176685/1461181) run:

```php
openssl req -x509 -newkey rsa:4096 -keyout server.key -out server.crt -days 365 -nodes
```

Enter the certificate request data:

```
Country: UK
State: XX
Locality name (City): London
Organisation Name: Acme AG
Organisation Unit: HeadOffice
Common-Name: www.example.com
E-Mail: info@example.com
Password: secret123
Optional company name: keep empty, press enter
```

* OpenSSl creates at least two files:
  * www_example_com.csr
  * www_example_com.key

* Order a real SSL certificate, e.g. from <https://letsencrypt.org/>, psw.net, GlobalSign, Sectigo, Thawte with the certificate request file: `www_example_com.csr`

* Important: Don't upload or share the private key file: `www_example_com.key`

* If you created a self-signed certificate, then you don't have to order a certificate, just copy these files:
  * From `C:\OpenSSL-Win32\bin\server.key` to `C:\xampp\apache\conf\ssl.key\server.key`
  * From `C:\OpenSSL-Win32\bin\server.crt` to `C:\xampp\apache\conf\ssl.crt\server.crt`
  * Restart apache
  * Open: https://localhost/

## Installing the SSL Certificate

* Open the file `C:\xampp\apache\conf\extra\httpd-ssl.conf` with Notepad++
* Add these lines and adjust the path and filenames.:

```
SSLCertificateFile "conf/ssl.crt/certificate.crt"
SSLCertificateKeyFile "conf/ssl.key/www_example_com.key"
SSLCertificateChainFile "conf/ssl.crt/intermediate1.crt"
SSLCACertificateFile "conf/ssl.crt/root.crt"
```

* Restart apache

## Testing the SSL certificate

This [SSL Checker](https://www.sslshopper.com/ssl-checker.html) will help you diagnose problems with your 
SSL certificate installation. You can verify the SSL certificate on your web server to make 
sure it is correctly installed, valid, trusted and doesn't give any errors to any of your users. 
To use the SSL Checker, simply enter your server's hostname (must be public) in the box below and 
click the Check SSL button.
