---
title: Slim 4 - Basic Authentication
layout: post
comments: true
published: true
description:
keywords: php, slim, basic auth
image: https://odan.github.io/assets/images/slim-logo-600x330.png
---

## Table of contents

* [Requirements](#requirements)
* [Introduction](#introduction)
* [Installation](#installation)
* [Configuration](#configuration)
* [Usage](#usage)
* [Passwords](#passwords)
* [Security](#security)
* [Testing](#testing)
* [Read more](#read-more)

## Requirements

* PHP 7.2+
* [A Slim 4 application](https://odan.github.io/2019/11/05/slim4-tutorial.html)

## Introduction

HTTP Basic authentication (BA) implementation is the simplest technique
for enforcing access controls to web resources because it does
not require cookies, session identifiers, or login pages;
rather, HTTP Basic authentication uses standard fields in the HTTP header.

The 'Basic' HTTP Authentication Scheme is specified 
in [RFC 7617](https://datatracker.ietf.org/doc/html/rfc7617) from 2015.
This specification defines the "Basic" Hypertext Transfer Protocol (HTTP)
authentication scheme, which transmits credentials as user-id/
password pairs, encoded using Base64.

The BA mechanism does not provide confidentiality protection 
for the transmitted credentials. They are merely encoded with Base64 
transit and not encrypted or hashed in any way. 
Therefore, basic authentication is typically used in conjunction 
with **HTTPS** to provide confidentiality.

When the user agent wants to send authentication credentials to the server, 
it may use the `Authorization` header field.

For example, if the browser uses `Aladdin` as the username and
`open sesame` as the password, then the field's value is the 
Base64 encoding of `Aladdin:open sesame`, or `QWxhZGRpbjpvcGVuIHNlc2FtZQ==`.
Then the Authorization header field will appear as:

`Authorization: Basic QWxhZGRpbjpvcGVuIHNlc2FtZQ==`

## Installation

The [tuupola/slim-basic-auth](https://github.com/tuupola/slim-basic-auth) package
provides a PSR-7 and PSR-15 Basic Auth Middleware.

To add the package to your application, run:

```php
composer require tuupola/slim-basic-auth
```

## Configuration

Configuration options are passed as an array. 
The only mandatory parameter is `users`. 
This is an array where you pass one or more 
`'username' => 'password'` combinations. 
Username is the key and password is the value.

```php
$settings['api_auth'] = [
     'users' => [
        'admin' => 'secret',
        'user' => 'password',
    ],
];
```

Next, add a DI container definitions for the `HttpBasicAuthentication` class
which also loads and passes the settings array with all users and password.

```php
<?php

use Psr\Container\ContainerInterface;
use Tuupola\Middleware\HttpBasicAuthentication;
// ...

return [

    // ...
    
    HttpBasicAuthentication::class => function (ContainerInterface $container) {
        return new HttpBasicAuthentication($container->get('settings')['api_auth']);
    },
];
```

## Usage

Typically, you add the `HttpBasicAuthentication` middleware to the 
route groups that you want to protect with BasicAuth. 
Of course, you could also add this middleware for each route individually. 
In practice, however, there are always several routes that you 
want to protect altogether.

```php
// config/routes.php
use Tuupola\Middleware\HttpBasicAuthentication;

// Password-protected area
$app->group(
    '/api',
    function (RouteCollectorProxy $app) {
        $app->get('/users', \App\Action\User\UserFindAction::class);
        $app->post('/users', \App\Action\User\UserCreateAction::class);
        $app->get('/users/{id}', \App\Action\User\UserReadAction::class);
        $app->put('/users/{id}', \App\Action\User\UserUpdateAction::class);
        $app->delete('/users/{id}', \App\Action\User\UserDeleteAction::class);
    }
)->add(HttpBasicAuthentication::class);
```

Of course, you can also add the middleware to the global middleware 
stack to cover all requests.
In this case you don't need to add this middleware to the routes anymore, 
because it will always be executed.
Choose the appropriate strategy depending on the project.

```php
// config/middleware.php
use Tuupola\Middleware\HttpBasicAuthentication;

// Protected all routes
$app->add(HttpBasicAuthentication::class);
```

## Passwords

Cleartext passwords are only good for quick testing. 
You probably want to use hashed passwords. 
Hashed password can be generated with `htpasswd`
command line tool or `password_hash()` PHP function:

```php
echo password_hash('secret', PASSWORD_DEFAULT);

// $2y$10$JKLleIlqaHs1HVbRAE2G3.sOmLMbxWHBu6dnd8iQ3BFvhw9wE1S9m
```

Your array with the usernames and password could then be stored like this:

```php
$settings['api_auth'] = [
     'users' => [
        'admin' => '$2y$10$JKLleIlqaHs1HVbRAE2G3.sOmLMbxWHBu6dnd8iQ3BFvhw9wE1S9m',
        'user' => '$2y$10$BAcmFwDluG.7eKcVbpcl.uEdBa6MWQkJp2NPufgn7ScU6aR2CrOBC',
    ],
];
```

Even if you are using hashed passwords it is not the best idea 
to store credentials in the code. Instead, you could store them in 
environment, external file or a database which is not committed to GitHub.

## Security

Basic authentication transmits credentials in base64 encoded. 
For this reason `HTTPS` should always be used together with basic authentication. 
If the middleware detects insecure usage over HTTP it will throw 
a `RuntimeException` with the following message:
`Insecure use of middleware over HTTP denied by configuration.`

By default, `localhost` is allowed to use HTTP. 
The security behavior of `HttpBasicAuthentication` can also 
be [configured](https://github.com/tuupola/slim-basic-auth#how-to-configure-a-whitelist).

## Testing

When you write HTTP tests you must make sure that the submitted 
request has the appropriate `Authorization` header with the 
username and password added. 

Otherwise, the middleware would immediately reject any test with an error.

First you need to configure the usernames/password array for your phpunit test
environment in e.g. `local.test.php`

```php
// local.test.php
$settings['api_auth'] = [
     'users' => [
        'user' => 'password',
    ],
];
```
Then you could add a test trait that appends the encodes header values to the request object.

File: `tests/Traits/HttpBasicAuthTestTrait.php`

```php
<?php

namespace App\Test\Traits;

use Psr\Http\Message\ServerRequestInterface;

trait HttpBasicAuthTestTrait
{
    protected function withHttpBasicAuth(ServerRequestInterface $request): ServerRequestInterface
    {
        return $request->withHeader('Authorization', 'Basic ' . base64_encode('user:password'));
    }
}

```

Within a PhpUnit test class you can use it at follows:

```php
$request = $this->createRequest('GET', '/api/users');

// Add basic auth to the request
$request = $this->withHttpBasicAuth($request);

// Run the Slim app and fetch the response
$response = $this->app->handle($request);

// Asserts..
```

## Read more

* <https://en.wikipedia.org/wiki/Basic_access_authentication>
