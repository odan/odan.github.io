---
title: Slim 4 - Symfony Session
layout: post
comments: true
published: true
description: 
keywords: php, slim, session, http, cookies, slim-framework
image: https://odan.github.io/assets/images/slim-logo-600x330.png
---

> **Update:** Please visit this new article:
> [Slim 4 - Session](https://odan.github.io/2021/01/15/slim4-session.html)

## Table of contents

* [Requirements](#requirements)
* [Introduction](#introduction)
* [Installation](#installation)
* [Configuration](#configuration)
* [Middleware](#middleware)
* [Container setup](#container-setup)
* [Usage](#usage)
* [Conclusion](#conclusion)  
* [Read more](#read-more)

## Requirements

* PHP 7.2+
* [A Slim 4 application](https://odan.github.io/2019/11/05/slim4-tutorial.html)

## Introduction

Since HTTP driven applications are stateless, sessions provide a way to store 
information about the user across multiple requests. The PHP ecosystem provides a 
variety of session components to handle sessions. But there are a dozen session
components, that are more or less well maintained.
The bullet-proof and best maintained session component for PHP comes from symfony.
It's part of the [HttpFoundation](https://github.com/symfony/http-foundation) 
component and handles sessions in an expressive, unified API.
Support for popular backends such as Memcached, Redis, and databases 
is included out of the box.

## Installation

Run:

```
composer require symfony/http-foundation
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

Read more: [Session configuration reference](https://symfony.com/doc/current/reference/configuration/framework.html#session)

## Middleware

Create a new file `src/Middleware/SessionMiddleware.php` and copy / paste this content:

```php
<?php

namespace App\Middleware;

use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface;
use Psr\Http\Server\MiddlewareInterface;
use Psr\Http\Server\RequestHandlerInterface;
use Symfony\Component\HttpFoundation\Session\Session;

final class SessionMiddleware implements MiddlewareInterface
{
    /**
     * @var Session
     */
    private $session;

    public function __construct(Session $session)
    {
        $this->session = $session;
    }

    public function process(
        ServerRequestInterface $request, 
        RequestHandlerInterface $handler
    ): ResponseInterface {
        $this->session->start();

        return $handler->handle($request);
    }
}

```

Now add the `SessionMiddleware`, before the `RoutingMiddleware`, into your Slim middleware stack.

```php
<?php

use App\Middleware\SessionMiddleware;
use Slim\App;

return function (App $app) {
    // ...

    // Start the session
    $app->add(SessionMiddleware::class); // <-- here

    $app->addRoutingMiddleware();

    // ...
};
```

## Container setup

Add a container definition for `Session:class` and `SessionInterface:class` in `config/container.php`:

```php
<?php

use App\Middleware\SessionMiddleware;
use Psr\Container\ContainerInterface;
use Psr\Http\Message\ResponseFactoryInterface;
use Slim\App;
use Symfony\Component\HttpFoundation\Session\Session;
use Symfony\Component\HttpFoundation\Session\SessionInterface;
use Symfony\Component\HttpFoundation\Session\Storage\MockArraySessionStorage;
use Symfony\Component\HttpFoundation\Session\Storage\NativeSessionStorage;

// ...

return [

    // ...
    ResponseFactoryInterface::class => function (ContainerInterface $container) {
        return $container->get(App::class)->getResponseFactory();
    },
    
    Session::class => function (ContainerInterface $container) {
        $settings = $container->get('settings')['session'];
        if (PHP_SAPI === 'cli') {
            return new Session(new MockArraySessionStorage());
        } else {
            return new Session(new NativeSessionStorage($settings));
        }
    },

    SessionInterface::class => function (ContainerInterface $container) {
        return $container->get(Session::class);
    },

];

```

## Usage

To access the `Session` instance, we must first declare it in the constructor so that it can 
be automatically injected by the IoC Container.

I want to show a simple login/logout to demonstrate the session and flash message handling.

Create a new file `src/Action/Auth/LoginSubmitAction.php` and copy/paste this content:

```php
<?php

namespace App\Action\Auth;

use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface;
use Slim\Routing\RouteContext;
use Symfony\Component\HttpFoundation\Session\Session;

final class LoginSubmitAction
{
    private Session $session;

    public function __construct(Session $session)
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
        // Check user credentials. You may use an application/domain service
        // and the database here.
        $user = null;
        if($username === 'admin' && $password === 'secret') {
            $user = 'admin';
        }

        // Clear all flash messages
        $flash = $this->session->getFlashBag();
        $flash->clear();

        // Get RouteParser from request to generate the urls
        $routeParser = RouteContext::fromRequest($request)->getRouteParser();

        if ($user) {
            // Login successfully
            // Clears all session data and regenerates session ID
            $this->session->invalidate();
            $this->session->start();
    
            $this->session->set('user', $user);
            $flash->set('success', 'Login successfully');
    
            // Redirect to protected page
            $url = $routeParser->urlFor('users');
        } else {
            $flash->set('error', 'Login failed!');

            // Redirect back to the login page
            $url = $routeParser->urlFor('login');
        }

        return $response->withStatus(302)->withHeader('Location', $url);
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
use Symfony\Component\HttpFoundation\Session\Session;

final class LogoutAction
{
    private Session $session;

    public function __construct(Session $session)
    {
        $this->session = $session;
    }

    public function __invoke(
        ServerRequestInterface $request, 
        ResponseInterface $response
    ): ResponseInterface {
        // Logout user
        $this->session->invalidate();

        $routeParser = RouteContext::fromRequest($request)->getRouteParser();
        $url = $routeParser->urlFor('logout');
        
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

To render the welcome page create a new file `src/Action/User/UserAction.php` 
and copy/paste this content:

```php
<?php

namespace App\Action\User;

use App\Responder\Responder;
use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface;
use Symfony\Component\HttpFoundation\Session\Session;

final class UserAction
{
    private Session $session;

    public function __construct(Session $session)
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

use Psr\Http\Message\ResponseFactoryInterface;
use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface;
use Psr\Http\Server\MiddlewareInterface;
use Psr\Http\Server\RequestHandlerInterface;
use Slim\Routing\RouteContext;
use Symfony\Component\HttpFoundation\Session\Session;

final class UserAuthMiddleware implements MiddlewareInterface
{
    private ResponseFactoryInterface $responseFactory;
    
    private Session $session;

    public function __construct(
        ResponseFactoryInterface $responseFactory, 
        Session $session
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

You can add the `UserAuthMiddleware::class` to individual routes and/or route groups you want to protect.

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

With the time I realized that there are some issues
with this component in combination with a middleware based approach like Slim.

The first issue was the missing lazy session start support. In a middleware
based application it's important to have full control of all HTTP specific
operations, this also includes session start time. This means, only a middleware
should control the start of the session and not the DI container or something else.
The issue here is that as soon as you try to configure the FlashBag handler, the
Session object starts the session. I have found a workaround by using
a `TwigFunction` to address this issue, but it stills feels a little hacky to me.

The second issue I had was the missing middleware support. 
For historical and political reason the Symfony HttpFoundation is not built 
for PSR-15 middleware. This issue can be simply solved by implementing 
a custom middleware as shown above. The signal for me remains the same,
it is not built for a middleware stack.

The third issue is that the Symfony HttpFoundation component brings a lot of
other non PSR-7/PSR15 HTTP classes as dependencies that do not fit into this context.
Some people even think that you have to manually convert the HttpFoundation
request to/from a PSR-7 request/response object. This is not true.
You don't have to convert any objects at all, because the Symfony `Session` class
works without the Symfony request/response classes.
Of course, it would be much better if the entire `Symfony\Component\HttpFoundation\Session`
namespace would be provided in a separate Github repository without all the
Symfony HTTP classes.

On the other hand, if you are looking for a very stable API and support for popular 
backends like Memcached, Redis and databases, then this component is still a very 
good choice for your Slim application.

## Read more

* <https://symfony.com/doc/current/components/http_foundation.html>
* <https://github.com/symfony/http-foundation>
* [Ask a question or report an issue](https://github.com/odan/slim4-tutorial/issues)
