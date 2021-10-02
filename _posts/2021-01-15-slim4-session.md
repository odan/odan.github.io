---
title: Slim 4 - Session
layout: post
comments: true
published: true
description:
keywords: php, slim, session, http, cookies, slim-framework
image: https://odan.github.io/assets/images/slim-logo-600x330.png
---

## Table of contents

* [Requirements](#requirements)
* [Introduction](#introduction)
* [Installation](#installation)
* [Configuration](#configuration)
* [Container Setup](#container-setup)
* [Middleware](#middleware)
* [Usage](#usage)
* [Conclusion](#conclusion)
* [Read more](#read-more)

## Requirements

* PHP 7.3+ or PHP 8.0+
* [A Slim 4 application](https://odan.github.io/2019/11/05/slim4-tutorial.html)

## Introduction

Since HTTP driven applications are stateless, sessions provide a way to store
information about the user across multiple requests. The PHP ecosystem provides a
variety of session components to handle sessions.

This time I want to show you how to install the `odan/session` component. 
Unlike many other PHP session components, this one is optimized for PSR-7 HTTP requests 
and PSR-15 middleware handlers. A unique feature is the "lazy session start" support 
to give the middleware complete control over the session start time.

## Installation

Run:

```
composer require odan/session
```

## Configuration

Insert the session settings into your configuration file, e.g. `config/settings.php`;

```php
// Session
$settings['session'] = [
    'name' => 'webapp',
    'cache_expire' => 0,
];
```

You can use all the standard PHP session configuration options.

Read more: [Session Runtime Configuration](https://www.php.net/manual/en/session.configuration.php)

## Container Setup

Add a DI container definition for `SessionInterface:class` 
and `SessionMiddleware:class` in `config/container.php`.

Make sure you also have a definition for `ResponseFactoryInterface::class` and `App::class`.

```php
<?php

use Odan\Session\PhpSession;
use Odan\Session\SessionInterface;
use Odan\Session\Middleware\SessionMiddleware;
use Psr\Container\ContainerInterface;
use Psr\Http\Message\ResponseFactoryInterface;
use Slim\App;

return [
    // ...

    SessionInterface::class => function (ContainerInterface $container) {
        $settings = $container->get('settings');
        $session = new PhpSession();
        $session->setOptions((array)$settings['session']);

        return $session;
    },

    SessionMiddleware::class => function (ContainerInterface $container) {
        return new SessionMiddleware($container->get(SessionInterface::class));
    },
    
    // ...
    
    App::class => function (ContainerInterface $container) {
        AppFactory::setContainer($container);

        return AppFactory::create();
    },
    
    ResponseFactoryInterface::class => function (ContainerInterface $container) {
        return $container->get(App::class)->getResponseFactory();
    },
];
```

## Middleware

Now add the `SessionMiddleware`, before the `RoutingMiddleware`, into your Slim middleware stack.

This example registers the session middleware for all routes.
It's also possible to register middleware for a single rout and routing groups.

```php
<?php

use Odan\Session\Middleware\SessionMiddleware;
use Slim\App;

return function (App $app) {
    // ...

    // Start the session
    $app->add(SessionMiddleware::class); // <-- here

    $app->addRoutingMiddleware();

    // ...
};
```

## Usage

To access the `SessionInterface` implementation, we must first declare it in the constructor so that it can
be automatically injected by the DI Container.

I want to show a simple login/logout mechanism to demonstrate the session 
and flash message handling.

Create a new file `src/Action/Auth/LoginSubmitAction.php` and copy/paste this content:

```php
<?php

namespace App\Action\Auth;

use Odan\Session\SessionInterface;
use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface;
use Slim\Routing\RouteContext;

final class LoginSubmitAction
{
    private SessionInterface $session;

    public function __construct(SessionInterface $session)
    {
        $this->session = $session;
    }

    public function __invoke(
        ServerRequestInterface $request,
        ResponseInterface $response
    ): ResponseInterface {
        $data = (array)$request->getParsedBody();
        $username = (string)($data['username'] ?? '');
        $password = (string)($data['password'] ?? '');

        // Pseudo example
        // Check user credentials. You may use an application/domain service and the database here.
        $user = null;
        if($username === 'admin' && $password === 'secret') {
            $user = 'admin';
        }

        // Clear all flash messages
        $flash = $this->session->getFlash();
        $flash->clear();

        // Get RouteParser from request to generate the urls
        $routeParser = RouteContext::fromRequest($request)->getRouteParser();

        if ($user) {
            // Login successfully
            // Clears all session data and regenerate session ID
            $this->session->destroy();
            $this->session->start();
            $this->session->regenerateId();
    
            $this->session->set('user', $user);
            $flash->add('success', 'Login successfully');
    
            // Redirect to protected page
            $url = $routeParser->urlFor('users');
        } else {
            $flash->add('error', 'Login failed!');

            // Redirect back to the login page
            $url = $routeParser->urlFor('login');
        }

        return $response->withStatus(302)->withHeader('Location', $url);
    }
}
```

To render the login page create a new file `src/Action/Auth/LoginAction.php`
and copy/paste this content:

```php
<?php

namespace App\Action\Auth;

use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface;

final class LoginAction
{
    public function __invoke(
        ServerRequestInterface $request, 
        ResponseInterface $response
    ): ResponseInterface {
        $html = '<form action="login" method="POST">
        Username: <input type="text" name="username"><br>
        Password: <input type="password" name="password"><br>
        <input type="submit" value="Login">
        </form>';
        $response->getBody()->write($html);

        return $response;
    }
}

```

Create a new file `src/Action/Auth/LogoutAction.php` and copy/paste this content:

```php
<?php

namespace App\Action\Auth;

use App\Responder\Responder;
use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface;
use Slim\Routing\RouteContext;
use Odan\Session\SessionInterface;

final class LogoutAction
{
    private SessionInterface $session;

    public function __construct(SessionInterface $session)
    {
        $this->session = $session;
    }

    public function __invoke(
        ServerRequestInterface $request, 
        ResponseInterface $response
    ): ResponseInterface {
        // Logout user
        $this->session->destroy();

        $routeParser = RouteContext::fromRequest($request)->getRouteParser();
        $url = $routeParser->urlFor('logout');
        
        return $response->withStatus(302)->withHeader('Location', $url);
    }
}

```

To render the welcome page create a new file `src/Action/User/UserAction.php`
and copy/paste this content:

```php
<?php

namespace App\Action\User;

use App\Responder\Responder;
use Odan\Session\SessionInterface;
use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface;

final class UserAction
{
    private SessionInterface $session;

    public function __construct(SessionInterface $session)
    {
        $this->session = $session;
    }
    
    public function __invoke(
        ServerRequestInterface $request, 
        ResponseInterface $response
    ): ResponseInterface {
        $username = $this->session->get('user');
        
        $response->getBody()->write(sprintf('Welcome %s', $username));
        
        return $response;
    }
}

```

To check the user session for each request you can add a middleware that
redirects all "invalid" requests to the login page.

```php
<?php

namespace App\Middleware;

use Odan\Session\SessionInterface;
use Psr\Http\Message\ResponseFactoryInterface;
use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface;
use Psr\Http\Server\MiddlewareInterface;
use Psr\Http\Server\RequestHandlerInterface;
use Slim\Routing\RouteContext;

final class UserAuthMiddleware implements MiddlewareInterface
{
    private ResponseFactoryInterface $responseFactory;
    
    private SessionInterface $session;

    public function __construct(
        ResponseFactoryInterface $responseFactory, 
        SessionInterface $session
    ) {
        $this->responseFactory = $responseFactory;
        $this->session = $session;
    }

    public function process(
        ServerRequestInterface $request, 
        RequestHandlerInterface $handler
    ): ResponseInterface{
        if ($this->session->get('user')) {
            // User is logged in
            return $handler->handle($request);
        }

        // User is not logged in. Redirect to login page.
        $routeParser = RouteContext::fromRequest($request)->getRouteParser();
        $url = $routeParser->urlFor('login');
        
        return $this->responseFactory->createResponse()
            ->withStatus(302)
            ->withHeader('Location', $url);
    }
}

```

You can add the `UserAuthMiddleware::class` to individual routes and/or route 
groups you want to protect.

```php
use App\Middleware\UserAuthMiddleware;
use Slim\Routing\RouteCollectorProxy;
// ...

// Password protected area
$app->group('/users', function (RouteCollectorProxy $group) {
    $group->get('/', \App\Action\User\UserAction::class)->setName('users');
    // add more routes ...
})->add(UserAuthMiddleware::class);
```

Add the routes as follows:

```php
$app->get('/users', \App\Action\User\UserAction::class)->setName('users');
$app->get('/login', \App\Action\Auth\LoginAction::class)->setName('login');
$app->post('/login', \App\Action\Auth\LoginSubmitAction::class);
$app->get('/logout', \App\Action\Auth\LogoutAction::class)->setName('logout'); 
```

## Conclusion

As you can see, with a middleware-based session handler, it's much easier to control the start 
time of the session and add cool middleware features as shown above.

## Read more

* [Documentation](https://odan.github.io/session/v5/)
* [Github Repository](https://github.com/odan/session)
* [Ask a question or report an issue](https://github.com/odan/session/issues)
