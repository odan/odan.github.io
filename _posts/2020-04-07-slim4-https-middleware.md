---
title: Slim 4 - HTTPS Middleware
layout: post
comments: true
published: false
description: 
keywords: php, slim, ssl, https, middleware, slim-framework
---

## Table of contents

* [Requirements](#requirements)
* [Introduction](#introduction)
* [Middleware](#middleware)
* [Container](#container)
* [Register the middleware](#register-the-middleware)
* [Usage](#usage)
* [Other solutions](#other-solutions)

## Requirements

* PHP 7.2+
* A Slim 4 application

## Introduction

To redirect HTTP traffic to HTTPS, you could try the following middleware.

## Middleware

Create a new file `src/Middleware/HttpsMiddleware.php` and copy / paste this content:

```php
<?php

namespace App\Middleware;

use Psr\Http\Message\ResponseFactoryInterface;
use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface;
use Psr\Http\Server\MiddlewareInterface;
use Psr\Http\Server\RequestHandlerInterface;

/**
 * Middleware.
 */
final class HttpsMiddleware implements MiddlewareInterface
{
    /**
     * @var ResponseFactoryInterface
     */
    private $responseFactory;

    /**
     * The constructor.
     *
     * @param ResponseFactoryInterface $responseFactory The response factory
     */
    public function __construct(ResponseFactoryInterface $responseFactory)
    {
        $this->responseFactory = $responseFactory;
    }

    /**
     * Invoke middleware.
     *
     * @param ServerRequestInterface $request The request
     * @param RequestHandlerInterface $handler The handler
     *
     * @return ResponseInterface The response
     */
    public function process(
        ServerRequestInterface $request, 
        RequestHandlerInterface $handler
    ): ResponseInterface {
        $uri = $request->getUri();

        if ($uri->getHost() !== 'localhost' && $uri->getScheme() !== 'https') {
            $url = (string)$uri->withScheme('https')->withPort(443);

            $response = $this->responseFactory->createResponse();

            // Redirect
            return $response->withStatus(302)->withHeader('Location', $url);
        }

        return $handler->handle($request);
    }
}

```

## Container

If not exists, insert a container definition for `ResponseFactoryInterface::class`:

```php
<?php

use Psr\Http\Message\ResponseFactoryInterface;
use Slim\App;
use Slim\Factory\AppFactory;

return [
    // ...

    ResponseFactoryInterface::class => function (ContainerInterface $container) {
        return $container->get(App::class)->getResponseFactory();
    },

    App::class => function (ContainerInterface $container) {
        AppFactory::setContainer($container);

        return AppFactory::create();
    },

];
```

## Register the middleware

Now add the `HttpsMiddleware`, before the `RoutingMiddleware`, into your Slim middleware stack.

Example:

```php
<?php

use App\Middleware\HttpsMiddleware;
use Slim\App;
use Slim\Middleware\ErrorMiddleware;
use Slim\Views\TwigMiddleware;

return function (App $app) {
    $app->addBodyParsingMiddleware();

    // Redirect HTTP traffic to HTTPS
    $app->add(HttpsMiddleware::class);

    $app->addRoutingMiddleware();
    $app->addErrorMiddleware(true, true, true);
};
```

## Usage

To see if it works, open your browser and enter the `http://` address of your application.
You should now be automatically redirected to the secure `https://` url.

## Other solutions

You could also use a `.htaccess` file to redirect the http traffic to https.

If you have existing code in your .htaccess, add the following:

```
RewriteEngine On
RewriteCond %{HTTPS} !=on
RewriteRule ^ https://%{HTTP_HOST}%{REQUEST_URI} [L,R=301]
```