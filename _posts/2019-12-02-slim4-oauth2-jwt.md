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
* [Limitations](#limitations)
* [Installation](#installation)
* [Generating Public and Private Keys](#generating-public-and-private-keys)
* [Configuration](#configuration)
* [Creating a JWT](#creating-a-jwt)
* [Bearer Authentication Middleware](#bearer-authentication-middleware)
* [Protecting routes with JWT](#protecting-routes-with-jwt)
* [FAQ](#faq)

## Requirements

* PHP 7.2+
* Composer
* [OpenSSL](https://www.openssl.org/)
* [A Slim 4 application](https://odan.github.io/2019/11/05/slim4-tutorial.html)

## Introduction

This tutorial demonstrates how signed JSON Web Tokens (JWTs) can be used as [OAuth 2.0](https://oauth.net/2/)
[Bearer Tokens](https://tools.ietf.org/html/rfc6750).

Please note that OAuth 2.0 is a **Authorization Framework** and not an [authentication protocol](https://oauth.net/articles/authentication/). 

Clients may use the HTTP Basic authentication scheme, as defined in [RFC2617](https://tools.ietf.org/html/rfc2617),
to authenticate with the server. In this tutorial we will use a RESTful approach to implement the authentication protocol.

## Limitations

Before you dig deeper, you should know when and especially when **NOT** to use OAuth 2.0 and JWTs.

The advantage of token-based authorization is that the client usually only needs to log-in once and doesn't have 
to submit its credentials again for each request. A token requires no state on the server and
therefore scales better across server boundaries.

Because a token is time-limited, the client must take care of the renewal itself. This can be tricky sometimes.

A **logout** functionality with tokens is not feasible without 
giving up the [stateless principle](https://restfulapi.net/statelessness/). This is because you cannot invalidate a 
token without keeping its status on a server. This would break the stateless principle.

Try to [avoid JWT for session management](http://cryto.net/~joepie91/blog/2016/06/13/stop-using-jwt-for-sessions/) 
or server-side storage for sessions. 

Do not include sensitive information in JWT tokens.

Think carefully about where to store the tokens:

* [Where to Store Tokens](https://auth0.com/docs/security/store-tokens#don-t-store-tokens-in-local-storage)
* [Do I have to store tokens in cookies or localstorage or session?](https://stackoverflow.com/a/54258744/1461181)
* If you store a JWT as a cookie, make it "HttpOnly" and "Secure".

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
composer require symfony/polyfill-uuid
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
openssl genrsa -out private.pem 2048
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

    'public_key' => '-----BEGIN PUBLIC KEY-----
        ...
        -----END PUBLIC KEY-----',
    
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

namespace App\Routing;

use Cake\Chronos\Chronos;
use InvalidArgumentException;
use Lcobucci\JWT\Builder;
use Lcobucci\JWT\Parser;
use Lcobucci\JWT\Signer\Key;
use Lcobucci\JWT\Signer\Rsa\Sha256;
use Lcobucci\JWT\Token;
use Lcobucci\JWT\ValidationData;
use UnexpectedValueException;

/**
 * JwtAuth.
 */
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
     * @var string The public key
     */
    private $publicKey;

    /**
     * @var Sha256 The signer
     */
    private $signer;

    /**
     * The constructor.
     *
     * @param string $issuer The issuer name
     * @param int $lifetime The max lifetime
     * @param string $privateKey The private key as string
     * @param string $publicKey The public key as string
     */
    public function __construct(
        string $issuer,
        int $lifetime,
        string $privateKey,
        string $publicKey
    ) {
        $this->issuer = $issuer;
        $this->lifetime = $lifetime;
        $this->privateKey = $privateKey;
        $this->publicKey = $publicKey;
        $this->signer = new Sha256();
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
     * @param array $claims The claims
     *
     * @throws UnexpectedValueException
     *
     * @return string The JWT
     */
    public function createJwt(array $claims): string
    {
        $issuedAt = Chronos::now()->getTimestamp();

        $builder = (new Builder())->issuedBy($this->issuer)
            ->identifiedBy(uuid_create(), true)
            ->issuedAt($issuedAt)
            ->canOnlyBeUsedAfter($issuedAt)
            ->expiresAt($issuedAt + $this->lifetime);

        foreach ($claims as $name => $value) {
            $builder = $builder->withClaim($name, $value);
        }

        return $builder->getToken($this->signer, new Key($this->privateKey));
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
     * Validate the access token.
     *
     * @param string $accessToken The JWT
     *
     * @return bool The status
     */
    public function validateToken(string $accessToken): bool
    {
        $token = $this->createParsedToken($accessToken);

        if (!$token->verify($this->signer, $this->publicKey)) {
            // Token signature is not valid
            return false;
        }

        // Check whether the token has not expired
        $data = new ValidationData();
        $data->setCurrentTime(Chronos::now()->getTimestamp());
        $data->setIssuer($token->getClaim('iss'));
        $data->setId($token->getClaim('jti'));

        return $token->validate($data);
    }
}
```

Add the the following container definitons, e.g. into `config/container.php`:

```php
<?php

use App\Routing\JwtAuth;
use Psr\Container\ContainerInterface;
use Psr\Http\Message\ResponseFactoryInterface;
use Slim\App;

return [
    'settings' => function () {
        return require __DIR__ . '/settings.php';
    },

    App::class => function (ContainerInterface $container) {
        AppFactory::setContainer($container);

        return AppFactory::create();
    },

    // Add this entry
    ResponseFactoryInterface::class => function (ContainerInterface $container) {
        return $container->get(App::class)->getResponseFactory();
    },

    // And add this entry
    JwtAuth::class => function (ContainerInterface $container) {
        $config = $container->get('settings')['jwt'];

        $issuer = (string)$config['issuer'];
        $lifetime = (int)$config['lifetime'];
        $privateKey = (string)$config['private_key'];
        $publicKey = (string)$config['public_key'];

        return new JwtAuth($issuer, $lifetime, $privateKey, $publicKey);
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

use App\Routing\JwtAuth;
use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface;

final class TokenCreateAction
{
    private $jwtAuth;

    public function __construct(JwtAuth $jwtAuth)
    {
        $this->jwtAuth = $jwtAuth;
    }

    public function __invoke(
        ServerRequestInterface $request,
        ResponseInterface $response
    ): ResponseInterface {
        $data = (array)$request->getParsedBody();

        $username = (string)($data['username'] ?? '');
        $password = (string)($data['password'] ?? '');

        // Validate login (pseudo code)
        // Warning: This should be done in an application service and not here!
        // $userAuthData = $this->userAuth->authenticate($username, $password);
        $isValidLogin = ($username === 'user' && $password === 'secret');

        if (!$isValidLogin) {
            // Invalid authentication credentials
            return $response
                ->withHeader('Content-Type', 'application/json')
                ->withStatus(401, 'Unauthorized');
        }

        // Create a fresh token
        $token = $this->jwtAuth->createJwt([
            'uid' => $username,
        ]);
        
        $lifetime = $this->jwtAuth->getLifetime();

        // Transform the result into a OAuh 2.0 Access Token Response
        // https://www.oauth.com/oauth2-servers/access-tokens/access-token-response/
        $result = [
            'access_token' => $token,
            'token_type' => 'Bearer',
            'expires_in' => $lifetime,
        ];

        // Build the HTTP response
        $response = $response->withHeader('Content-Type', 'application/json');
        $response->getBody()->write((string)json_encode($result));

        return $response->withStatus(201);
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

The Slim [BodyParsingMiddleware](https://www.slimframework.com/docs/v4/middleware/body-parsing.html) only 
parses the request body if the request header is set correctly.
Make sure that your client also sends this request header: 

```
Content-Type: application/json
```

> **Please note:** The [OAuth 2 spec](https://tools.ietf.org/html/rfc6749#section-2.3.1) states 
> that the username and password MAY use the HTTP Basic authentication or form data (so, no JSON).

## Bearer Authentication Middleware

Bearer authentication (also called token authentication) is an HTTP authentication 
scheme that involves security tokens called bearer tokens. The Bearer authentication 
scheme was originally created as part of OAuth 2.0 in [RFC 6750](https://tools.ietf.org/html/rfc6750).
The client must send the JWT within the `Authorization` request header in this format:

```
Authorization: Bearer <token>
```

Create the following middleware to validate the Bearer authentication header: 
`src/Middleware/JwtAuthMiddleware.php`

```php
<?php

namespace App\Middleware;

use App\Routing\JwtAuth;
use Psr\Http\Message\ResponseFactoryInterface;
use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface;
use Psr\Http\Server\MiddlewareInterface;
use Psr\Http\Server\RequestHandlerInterface;

/**
 * JWT Auth middleware.
 */
final class JwtAuthMiddleware implements MiddlewareInterface
{
    /**
     * @var JwtAuth
     */
    private $jwtAuth;

    /**
     * @var ResponseFactoryInterface
     */
    private $responseFactory;

    /**
     * The constructor.
     *
     * @param JwtAuth $jwtAuth The JWT auth
     * @param ResponseFactoryInterface $responseFactory The response factory
     */
    public function __construct(
        JwtAuth $jwtAuth,
        ResponseFactoryInterface $responseFactory
    ) {
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
    public function process(
        ServerRequestInterface $request,
        RequestHandlerInterface $handler
    ): ResponseInterface {
        $token = explode(' ', (string)$request->getHeaderLine('Authorization'))[1] ?? '';

        if (!$token || !$this->jwtAuth->validateToken($token)) {
            return $this->responseFactory->createResponse()
                ->withHeader('Content-Type', 'application/json')
                ->withStatus(401, 'Unauthorized');
        }

        return $handler->handle($request);
    }
}

```

Create the following middleware to extract the claims from the token: 
`src/Middleware/JwtClaimMiddleware.php`

```php
<?php

namespace App\Middleware;

use App\Routing\JwtAuth;
use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface;
use Psr\Http\Server\MiddlewareInterface;
use Psr\Http\Server\RequestHandlerInterface;

/**
 * JWT Claim middleware.
 */
final class JwtClaimMiddleware implements MiddlewareInterface
{
    /**
     * @var JwtAuth
     */
    private $jwtAuth;

    /**
     * The constructor.
     *
     * @param JwtAuth $jwtAuth The JWT auth
     */
    public function __construct(JwtAuth $jwtAuth)
    {
        $this->jwtAuth = $jwtAuth;
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
        $token = explode(' ', (string)$request->getHeaderLine('Authorization'))[1] ?? '';

        if ($token && $this->jwtAuth->validateToken($token)) {
            // Append valid token
            $parsedToken = $this->jwtAuth->createParsedToken($token);
            $request = $request->withAttribute('token', $parsedToken);

            // Append the user id as request attribute
            $request = $request->withAttribute('uid', $parsedToken->getClaim('uid'));

            // Add more claim values...
            //$request = $request->withAttribute('locale', $parsedToken->getClaim('locale'));
        }

        return $handler->handle($request);
    }
}

```

Add the `JwtClaimMiddleware` into your middleware stack, e.g. in `config/middleware.php`:

```php
<?php

use App\Middleware\JwtClaimMiddleware;
// ...

return function (App $app) {
    // ...

    $app->add(JwtClaimMiddleware::class);

    // ...
};
```

## Protecting routes with JWT

If you want to protect a single route just add the `JwtAuthMiddleware` to 
the route you want to protect:

```php
$app->post('/users', \App\Action\UserCreateAction::class)
    ->add(\App\Middleware\JwtAuthMiddleware::class);
```

If you want to protect a route group just add the `JwtAuthMiddleware` 
to the route group you want to protect:

```php
<?php

use App\Middleware\JwtAuthMiddleware;
use Slim\App;
use Slim\Routing\RouteCollectorProxy;

return function (App $app) {
    // API login. This route must not be protected.
    $app->post('/tokens', \App\Action\TokenCreateAction::class);

    // API endpoints. This group is protected with JWT.
    $app->group('/api', function (RouteCollectorProxy $group) {
        $group->get('/users/{id}', \App\Action\UserReadAction::class);
        $group->post('/users', \App\Action\UserCreateAction::class);
    })->add(JwtAuthMiddleware::class);

};
```

## FAQ

### Where to store the token?

If you store a JWT as a cookie, make it `HttpOnly` and `Secure`.

A cookie with the `HttpOnly` attribute is inaccessible 
to the JavaScript `Document.cookie` API; it is sent only to the server.

### How to implement a logout?

JSON Web Tokens are stateless. You can’t change a token to be invalid in 
a straightforward way. The easiest way to implement logging out is just 
to remove the token from the browser. Since the cookies that we designed are 
`HttpOnly`, you need to create an endpoint that clears it.

A logout route, e.g. `GET /logout`:

```php
$response = $response->withHeader(
    'Set-Cookie', 
    'Authentication=; HttpOnly; Secure; Path=/; Max-Age=0'
);
```

**How to handle CORS with OPTIONS preflight requests?**

Read more: [Slim 4 - CORS Setup](https://odan.github.io/2019/11/24/slim4-cors.html)

**The `Authorization` header missing in POST request**

If using Apache add the following to the `.htaccess` file. 
Otherwise PHP won't have access to the `Authorization` header.

```
RewriteRule .* - [env=HTTP_AUTHORIZATION:%{HTTP:Authorization}]
```

Read more: <https://stackoverflow.com/a/26791450/1461181>

**Is there a working library?**

* For JWT auth you may try: [tuupola/slim-jwt-auth](https://github.com/tuupola/slim-jwt-auth)
* For BasicAuth try: [tuupola/slim-basic-auth](https://github.com/tuupola/slim-basic-auth)
