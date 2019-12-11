---
title: Slim 4 - OAuth 2.0 and JSON Web Token (JWT) Setup
layout: post
comments: true
published: true
description: 
keywords: php slim oauth jwt json authentication security
---

## Table of contents

* [Requirements](#requirements)
* [Introduction](#introduction)
* [Installation](#installation)
* [Generating Public and Private Keys](#generating-public-and-private-keys)
* [Configuration](#configuration)
* [Creating a JWT](#creating-a-jwt)
* [Bearer Authentication Middleware](#bearer-authentication-middleware)
* [Protecting routes with JWT](#protecting-routes-with-jwt)

## Requirements

* PHP 7.1+
* Composer
* [OpenSSL](https://www.openssl.org/)
* [A Slim 4 application](https://odan.github.io/2019/11/05/slim4-tutorial.html)

## Introduction

This tutorial demonstrates how to implement the [OAuth 2.0](https://oauth.net/2/) authentication 
standard in combination with a [JSON Web Token](https://oauth.net/2/jwt/) (JWT).

Please note that a logout functionality with tokens is not feasible without 
giving up the stateless principle.

## Installation

[lcobucci/jwt](https://github.com/lcobucci/jwt) is a very good library to work 
with JSON Web Token (JWT) and JSON Web Signature based on RFC 7519.

The Package is available on [packagist](https://packagist.org/packages/lcobucci/jwt), 
you can install it using composer:

```
composer require lcobucci/jwt
```

For the JWT claim we are installing a UUID generator:

```
composer require ramsey/uuid
```

For the issue date and better testability we are installing the chronos date time library:

```
composer require cakephp/chronos
```

## Generating Public and Private Keys

First we have to create a **private key** for signature creation and a **public key** for verification.
This means that it's fine to distribute your public key. However, the private key should remain secret.

Generate the private key with this OpenSSL command (enter a password):

```
openssl genrsa -aes256 -out private.pem 2048
```

The private key is generated and saved in a file named "private.pem", located in the same directory.

**Note** The number "2048" in the above command indicates the size of the private key. 
You can choose one of these sizes: 512, 758, 1024, 2048 or 4096 (these numbers represent bits). 
The larger sizes offer greater security, but this is offset by a penalty in CPU performance. 

**Generating the Public Key**

Later we need a public key for the token verification.
To extract the public key file, type the following:

```
openssl rsa -in private.pem -outform PEM -pubout -out public.pem
```
 
The public key is saved in a file named `public.pem` located in the same directory.

## Configuration

Copy the content of your private key `private.pem` into your application 
configuration file, e.g. `config/settings.php`:

```
$settings['jwt'] = [

    // The issuer name
    'issuer' => 'www.example.com',

    // Max lifetime in seconds
    'lifetime' => 14400,

    // The private key
    'private_key' => '-----BEGIN RSA PRIVATE KEY-----
    ...
    -----END RSA PRIVATE KEY-----',
];
```

Make sure that you not commit the private key into your version control (e.g git).
In reality you could merge the private key from an external file (e.g. `env.php`) or load it
from another (secure) source.

## Creating a JWT

For the sake of simplicity, dependency injection and testability 
add the following class into this file: `src/Auth/JwtAuth.php`.

```php
<?php

namespace App\Auth;

use Cake\Chronos\Chronos;
use Cake\Core\Configure;
use InvalidArgumentException;
use Lcobucci\JWT\Builder;
use Lcobucci\JWT\Parser;
use Lcobucci\JWT\Signer\Key;
use Lcobucci\JWT\Signer\Hmac\Sha256;
use Lcobucci\JWT\Token;
use Lcobucci\JWT\ValidationData;
use Ramsey\Uuid\Uuid;
use UnexpectedValueException;

final class JwtAuth
{
    /**
     * @var string The issuer name
     */
    private $issuer;

    /**
     * @var int Max lifetime in seconds
     */
    private $lifetime;

    /**
     * @var string The private key
     */
    private $privateKey;

    /**
     * The constructor.
     *
     * @param string $issuer The issuer name
     * @param int $lifetime The max lifetime
     * @param string $privateKey The private key as string
     */
    public function __construct(string $issuer, int $lifetime, string $privateKey)
    {
        $this->issuer = $issuer;
        $this->lifetime = $lifetime;
        $this->privateKey = $privateKey;
    }

    /**
     * Get JWT max lifetime.
     *
     * @return int The lifetime in seconds
     */
    public function getLifetime(): int
    {
        return $this->lifetime;
    }

    /**
     * Create JSON web token.
     *
     * @param string $uid The user id
     *
     * @throws UnexpectedValueException
     *
     * @return string The JWT
     */
    public function createJwt(string $uid): string
    {
        $issuedAt = Chronos::now()->getTimestamp();

        return (new Builder())
            ->issuedBy($this->issuer)
            // (JWT ID) Claim, a unique identifier for the JWT
            ->identifiedBy(Uuid::uuid4()->toString(), true)
            ->issuedAt($issuedAt)
            ->canOnlyBeUsedAfter($issuedAt)
            ->expiresAt($issuedAt + $this->lifetime)
            ->withClaim('uid', $uid)
            // AES256 CBC + HMAC SHA-256
            ->getToken(new Sha256(), new Key($this->privateKey));
    }


    /**
     * Parse token.
     *
     * @param string $token The JWT
     *
     * @throws InvalidArgumentException
     *
     * @return Token The parsed token
     */
    public function createParsedToken(string $token): Token
    {
        return (new Parser())->parse($token);
    }

    /**
     * Validate token.
     *
     * @param string $token The JWT
     *
     * @return bool The status
     */
    public function validateToken(string $token): bool
    {
        return $this->createParsedToken($token)->validate($this->createValidationData());
    }

    /**
     * Create validation data.
     *
     * @return ValidationData The data
     */
    private function createValidationData(): ValidationData
    {
        $data = new ValidationData();
        $data->setCurrentTime(Chronos::now()->getTimestamp());
        $data->setIssuer($this->issuer);

        return $data;
    }
}
```

Add the the following container definitons, e.g. into `config/container.php`:

```php
<?php

use App\Auth\JwtAuth;
use Psr\Container\ContainerInterface;
use Psr\Http\Message\ResponseFactoryInterface;
use Selective\Config\Configuration;
use Slim\App;

return [
    Configuration::class => function () {
        return new Configuration(require __DIR__ . '/settings.php');
    },

    App::class => function (ContainerInterface $container) {
        AppFactory::setContainer($container);
        $app = AppFactory::create();

        // Optional: Set the base path to run the app in a sub-directory.
        //$app->setBasePath('/sub-directory');

        return $app;
    },

    // Add this entry
    ResponseFactoryInterface::class => function (ContainerInterface $container) {
        return $container->get(App::class)->getResponseFactory();
    },

    // And add this entry
    JwtAuth::class => function (ContainerInterface $container) {
        $config = $container->get(Configuration::class);

        $issuer = $config->getString('jwt.issuer');
        $lifetime = $config->getInt('jwt.lifetime');
        $privateKey = $config->getString('jwt.private_key');

        return new JwtAuth($issuer, $lifetime, $privateKey);
    },
];

```

## Creating a token

The http client requires a special route to create a new token: `POST /api/tokens`.

Add the following route into your routing configuration file, e.g. `config/routes.php`

```php
$app->post('/api/tokens', \App\Action\TokenCreateAction::class);
```

Then create the following action class for the route: `src/Action/TokenCreateAction.php`

```php
<?php

namespace App\Action;

use App\Auth\JwtAuth;
use Slim\Http\Response;
use Slim\Http\ServerRequest;

final class TokenCreateAction
{
    private $jwtAuth;

    public function __construct(JwtAuth $jwtAuth)
    {
        $this->jwtAuth = $jwtAuth;
    }

    public function __invoke(ServerRequest $request, Response $response): Response
    {
        $data = (array)$request->getParsedBody();

        $username = (string)($data['username'] ?? '');
        $password = (string)($data['password'] ?? '');

        // Validate login (pseudo code)
        // Warning: This should be done in an application service and not here!
        // e.g. $isValidLogin = $this->userAuth->checkLogin($username, $password); 
        $isValidLogin = ($username === 'user' && $password === 'secret');

        if (!$isValidLogin) {
            // Invalid authentication credentials
            return $response
                ->withHeader('Content-Type', 'application/json')
                ->withStatus(401, 'Unauthorized');
        }

        // Create a fresh token
        $token = $this->jwtAuth->createJwt($username);
        $lifetime = $this->jwtAuth->getLifetime();

        // Transform the result into a OAuh 2.0 Access Token Response
        // https://www.oauth.com/oauth2-servers/access-tokens/access-token-response/
        $result = [
            'access_token' => $token,
            'token_type' => 'Bearer',
            'expires_in' => $lifetime,
        ];

        // Build the HTTP response
        return $response->withJson($result)->withStatus(201);
    }
}
```

To create a new token, the client must send a POST request to `/api/tokens` with a valid JSON request
and a body content like this:

```json
{
    "username": "user",
    "password": "secret"
}
```

The Slim [BodyParsingMiddleware](http://www.slimframework.com/docs/v4/middleware/body-parsing.html) only 
parses the request body if the request header is set correctly.
Make sure that your client also sends this request header: 

```
Content-Type: application/json
```

## Bearer Authentication Middleware

Bearer authentication (also called token authentication) is an HTTP authentication 
scheme that involves security tokens called bearer tokens. The Bearer authentication 
scheme was originally created as part of OAuth 2.0 in [RFC 6750](https://tools.ietf.org/html/rfc6750).
The client must send the JWT within the `Authorization` request header in this format:

```
Authorization: Bearer <token>
```

Create the following middleware to parse the Bearer authentication header: `src/Middleware/JwtMiddleware.php`

```php
<?php

namespace App\Middleware;

use App\Auth\JwtAuth;
use Psr\Http\Message\ResponseFactoryInterface;
use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface;
use Psr\Http\Server\MiddlewareInterface;
use Psr\Http\Server\RequestHandlerInterface;

/**
 * JWT middleware.
 */
final class JwtMiddleware implements MiddlewareInterface
{
    /**
     * @var JwtAuth
     */
    private $jwtAuth;

    /**
     * @var ResponseFactoryInterface
     */
    private $responseFactory;

    public function __construct(JwtAuth $jwtAuth, ResponseFactoryInterface $responseFactory)
    {
        $this->jwtAuth = $jwtAuth;
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
    public function process(ServerRequestInterface $request, RequestHandlerInterface $handler): ResponseInterface
    {
        $authorization = explode(' ', (string)$request->getHeaderLine('Authorization'));
        $token = $authorization[1] ?? '';

        if (!$token || !$this->jwtAuth->validateToken($token)) {
            return $this->responseFactory->createResponse()
                ->withHeader('Content-Type', 'application/json')
                ->withStatus(401, 'Unauthorized');
        }

        // Append valid token
        $parsedToken = $this->jwtAuth->createParsedToken($token);
        $request = $request->withAttribute('token', $parsedToken);

        // Append the user id as request attribute
        $request = $request->withAttribute('uid', $parsedToken->getClaim('uid'));

        return $handler->handle($request);
    }
}
```

## Protecting routes with JWT

If you want to protect a single route just add the `JwtMiddleware` to the route you want to protect:

```php
$app->post('/users', \App\Action\UserCreateAction::class)
    ->add(\App\Middleware\JwtMiddleware::class);
```

If you want to protect a route group just add the `JwtMiddleware` to the route group you want to protect:

```php
<?php

use App\Middleware\JwtMiddleware;
use Slim\App;
use Slim\Routing\RouteCollectorProxy;

return function (App $app) {
    // This route must not be protected
    $app->post('/api/tokens', \App\Action\TokenCreateAction::class);

    // Protect the whole group
    $app->group('/api', function (RouteCollectorProxy $group) {
        $group->get('/users/{id}', \App\Action\UserReadAction::class);
        $group->post('/users', \App\Action\UserCreateAction::class);
    })->add(JwtMiddleware::class);

};
```

## Other solutions

Instead of implementing the JWT middleware yourself (which has some advantages), you can
also try this [PSR-7 and PSR-15 JWT Authentication](https://github.com/tuupola/slim-jwt-auth) middleware.

[Comments](https://gist.github.com/odan/2885ccd0d2f3a3df41bf5c3d6e9b4999)


