---
title: Slim 4 - JSON Web Token (JWT) authentication
layout: post
comments: true
published: false
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

* PHP 7.4+ or 8.0+
* [OpenSSL](https://www.openssl.org/)
* [A Slim 4 application](https://odan.github.io/2019/11/05/slim4-tutorial.html)

## Introduction

This tutorial demonstrates how signed JSON Web Tokens (JWTs) can be used as [OAuth 2.0](https://oauth.net/2/)
[Bearer Tokens](https://tools.ietf.org/html/rfc6750).

In **authentication**, when the user successfully logs in using their credentials, 
a JSON Web Token will be returned. 

Please note that OAuth 2.0 is a **Authorization Framework** and not an [authentication protocol](https://oauth.net/articles/authentication/). 

Clients may use the HTTP Basic authentication scheme, as defined in [RFC2617](https://tools.ietf.org/html/rfc2617),
to authenticate with the server. In this tutorial we will use a RESTful approach to implement 
the authentication protocol.

## Limitations

Before you dig deeper, you should know when and especially when **NOT** to use OAuth 2.0 and JWTs.

The advantage of token-based authorization is that the client usually only needs to log-in once and doesn't have 
to submit its credentials again for each request. A token requires no state on the server and
therefore scales better across server boundaries.

Since tokens are credentials, great care must be taken to prevent security issues. In general, 
you should not keep tokens longer than required.

Because a token is time-limited, the client must take care of the renewal itself. This can be tricky sometimes.

A **logout** functionality with tokens is not feasible without 
giving up the [stateless principle](https://auth0.com/blog/stateless-auth-for-stateful-minds/). 
This is because you cannot invalidate a token without keeping its status on a server. 
This would break the stateless principle. However, if your client is only a browser, you
can delete a JWT cookie with this trick: [How to implement a logout?](#how-to-implement-a-logout)

Try to **[avoid JWT for session management](http://cryto.net/~joepie91/blog/2016/06/13/stop-using-jwt-for-sessions/)** 
or server-side storage for sessions. 

Do not include sensitive information in JWT tokens.

Think carefully about where to store the tokens:

* [Where to Store Tokens](https://auth0.com/docs/security/store-tokens#don-t-store-tokens-in-local-storage)
* [Do I have to store tokens in cookies or localstorage or session?](https://stackoverflow.com/a/54258744/1461181)
* If you store a JWT as a cookie, make it "HttpOnly" and "Secure".

## Installation

[lcobucci/jwt](https://github.com/lcobucci/jwt) is a very good component to work 
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

For the issue date and better testability we are installing the Chronos date time library:

```
composer require cakephp/chronos
```

## Generating Public and Private Keys

First we have to create a **private key** for signature creation, and a **public key** for verification.
This means that it's fine to distribute your public key. However, the private key should remain secret.

Generate the private key with this OpenSSL command (enter a password):

```
openssl genrsa -out private.pem 2048
```

> In case that openssl is not installed, this 
> **[Online RSA Key Generator](https://travistidwell.com/jsencrypt/demo/)** 
> can be used as well.

The private key is generated and saved in a file named "private.pem", located in the same directory.

The number "2048" in the above command indicates the size of the private key. 
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

Copy the content of `private.pem` and `public.pem` into the application 
configuration file, e.g. `config/settings.php`:

```php
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

**Important:** Make sure that the private/public key does not contain any extra spaces or invalid newline characters.
Newlines must be saved with `\n` and not `\r\n`. If the key is not valid, the following error could be triggered:

```
It was not possible to parse your key, 
reason: error:0909006C:PEM routines:get_name:no start line
```

In this case, remove additional spaces at the beginning/end of the key.
It's important to add the keys in this format:

```php
$settings['jwt'] = [
    //...
    'private_key' => '-----BEGIN RSA PRIVATE KEY-----\n...\n-----END RSA PRIVATE KEY-----',
    'public_key'  => '-----BEGIN PUBLIC KEY-----\n...\n-----END PUBLIC KEY-----',
];
```

If you still have problems getting the string into the correct format, 
try loading the keys from an external file:

```php
$settings['jwt'] = [
    //...
    'private_key' => file_get_contents('path/to/private.key'),
    'public_key'  => file_get_contents('path/to/public.key'),
];
```

**Security note:** Make sure that you **not** commit the keys into version control.
In reality, the keys should be included from another secure source, e.g. from `env.php`:

```php
// env.php
$settings['jwt']['private_key'] = 'private key';
$settings['jwt']['public_key'] = 'public key';
```

## Creating a JWT

For the sake of simplicity, dependency injection and testability 
add the following class into this file: `src/Routing/JwtAuth.php`.

```php
<?php

namespace App\Routing;

use Cake\Chronos\Chronos;
use Lcobucci\Clock\SystemClock;
use Lcobucci\JWT\Configuration;
use Lcobucci\JWT\Token\Plain;
use Lcobucci\JWT\Validation\Constraint\IssuedBy;
use Lcobucci\JWT\Validation\Constraint\SignedWith;
use Lcobucci\JWT\Validation\Constraint\ValidAt;
use Lcobucci\JWT\Validation\ConstraintViolation;
use Lcobucci\JWT\ValidationData;

/**
 * JwtAuth.
 */
final class JwtAuth
{
    /**
     * @var Configuration
     */
    private Configuration $configuration;

    /**
     * @var string The issuer name
     */
    private string $issuer;

    /**
     * @var int Max lifetime in seconds
     */
    private int $lifetime;

    /**
     * The constructor.
     *
     * @param Configuration $configuration
     * @param string $issuer The issuer name
     * @param int $lifetime The max lifetime in seconds
     */
    public function __construct(Configuration $configuration, string $issuer, int $lifetime)
    {
        $this->configuration = $configuration;
        $this->issuer = $issuer;
        $this->lifetime = $lifetime;
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
     * @param array<string> $claims The claims
     *
     * @return string The JWT
     */
    public function createJwt(array $claims): string
    {
        $now = Chronos::now();

        $builder = $this->configuration->builder()
            // Configures the issuer (iss claim)
            ->issuedBy($this->issuer)
            // Configures the id (jti claim)
            ->identifiedBy(uuid_create())
            // Configures the time that the token was issue (iat claim)
            ->issuedAt($now)
            // Configures the time that the token can be used (nbf claim)
            ->canOnlyBeUsedAfter($now)
            // Configures the expiration time of the token (exp claim)
            ->expiresAt($now->addSeconds($this->lifetime));

        // Add claims like "uid"
        foreach ($claims as $name => $value) {
            $builder = $builder->withClaim($name, $value);
        }

        // Builds a new token using the private key
        return $builder->getToken(
            $this->configuration->signer(),
            $this->configuration->signingKey()
        )->toString();
    }

    /**
     * Parse token.
     *
     * @param string $token The JWT
     *
     * @throws ConstraintViolation
     *
     * @return Plain The parsed token
     */
    private function createParsedToken(string $token): Plain
    {
        $token = $this->configuration->parser()->parse($token);

        if (!$token instanceof Plain) {
            throw new ConstraintViolation('You should pass a plain token');
        }

        return $token;
    }

    /**
     * Validate the access token.
     *
     * @param string $accessToken The JWT
     *
     * @return Plain|null The token, if valid
     */
    public function validateToken(string $accessToken): ?Plain
    {
        $token = $this->createParsedToken($accessToken);
        $constraints = $this->configuration->validationConstraints();

        // Token signature must be valid
        $constraints[] = new SignedWith(
            $this->configuration->signer(),
            $this->configuration->verificationKey()
        );

        // Check whether the issuer is the same
        $constraints[] = new IssuedBy($this->issuer);

        // Check whether the token has not expired
        $constraints[] = new ValidAt(new SystemClock(Chronos::now()->getTimezone()));

        if (!$this->configuration->validator()->validate($token, ...$constraints)) {
            // Token signature is not valid
            return null;
        }

        // Custom constraints
        // Check whether the user id is valid
        $userId = $token->claims()->get('uid');
        if (!$userId) {
            // Token related to an unknown user
            return null;
        }

        return $token;
    }
}
```

Add the following container definitions, e.g. into `config/container.php`:

```php
<?php

use App\Routing\JwtAuth;
use Lcobucci\JWT\Configuration;
use Lcobucci\JWT\Signer\Rsa\Sha256;
use Lcobucci\JWT\Signer\Key\InMemory;
use Psr\Container\ContainerInterface;
use Psr\Http\Message\ResponseFactoryInterface;
use Selective\BasePath\BasePathMiddleware;
use Slim\App;
use Slim\Factory\AppFactory;

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

    // Add this entry
    JwtAuth::class => function (ContainerInterface $container) {
        $configuration = $container->get(Configuration::class);

        $jwtSettings = $container->get('settings')['jwt'];
        $issuer = (string)$jwtSettings['issuer'];
        $lifetime = (int)$jwtSettings['lifetime'];

        return new JwtAuth($configuration, $issuer, $lifetime);
    },

    // Add this entry
    Configuration::class => function (ContainerInterface $container) {
        $jwtSettings = $container->get('settings')['jwt'];

        $privateKey = (string)$jwtSettings['private_key'];
        $publicKey = (string)$jwtSettings['public_key'];

        // Asymmetric algorithms use a private key for signature creation
        // and a public key for verification
        return Configuration::forAsymmetricSigner(
            new Sha256(),
            InMemory::plainText($privateKey),
            InMemory::plainText($publicKey)
        );
    },
];

```

## Creating a token

The http client requires a special route to create a new token: `POST /tokens`.

Add the following route into your routing configuration file, e.g. `config/routes.php`

```php
$app->post('/tokens', \App\Action\Auth\TokenCreateAction::class);
```

Then create the following action class for the route: `src/Action/Auth/TokenCreateAction.php`

```php
<?php

namespace App\Action\Auth;

use App\Routing\JwtAuth;
use JsonException;
use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface;

final class TokenCreateAction
{
    private JwtAuth $jwtAuth;

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
     * Invoke.
     *
     * @param ServerRequestInterface $request The request
     * @param ResponseInterface $response The response
     *
     * @throws JsonException
     *
     * @return ResponseInterface The response
     */
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
        $token = $this->jwtAuth->createJwt(
            [
                'uid' => $username,
            ]
        );

        // Transform the result into a OAuh 2.0 Access Token Response
        // https://www.oauth.com/oauth2-servers/access-tokens/access-token-response/
        $result = [
            'access_token' => $token,
            'token_type' => 'Bearer',
            'expires_in' => $this->jwtAuth->getLifetime(),
        ];

        // Build the HTTP response
        $response = $response->withHeader('Content-Type', 'application/json');
        $response->getBody()->write((string)json_encode($result, JSON_THROW_ON_ERROR));

        return $response->withStatus(201);
    }
}

```

To create a new token, the client must send a POST request to `/tokens` with a valid JSON request, 
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

## Bearer Authentication Middleware

Whenever the user wants to access a protected route or resource, 
the user agent should send the JWT, typically in the Authorization 
header using the Bearer schema. 

The content of the header should look like the following:

```
Authorization: Bearer <token>
```

This can be, in certain cases, a stateless authorization mechanism. 
The protected routes resp. the `JwtAuthMiddleware` will check 
for a valid JWT in the Authorization header, and if it's present, 
the user will be allowed to access protected resources. 
If the JWT contains the necessary data, the need to query the database for 
certain operations may be reduced, though this may not always be the case.

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
    private JwtAuth $jwtAuth;

    /**
     * @var ResponseFactoryInterface
     */
    private ResponseFactoryInterface $responseFactory;

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
    private JwtAuth $jwtAuth;

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
        $authorization = explode(' ', (string)$request->getHeaderLine('Authorization'));
        $type = $authorization[0] ?? '';
        $credentials = $authorization[1] ?? '';

        if ($type !== 'Bearer') {
            return $handler->handle($request);
        }

        $token = $this->jwtAuth->validateToken($credentials);
        if ($token) {
            // Append valid token
            $request = $request->withAttribute('token', $token);

            // Append the user id as request attribute
            $request = $request->withAttribute('uid', $token->claims()->get('uid'));

            // Add more claim values as attribute...
            //$request = $request->withAttribute('locale', $token->claims()->get('locale'));
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

### Should I use JWT or PHP sessions?

JWT should not be used for sessions: [JWTs Suck as Session Tokens](https://developer.okta.com/blog/2017/08/17/why-jwts-suck-as-session-tokens)

For a typical web-application with login/logout functionality a 
cookie based session handling is still a very good option today.

In >98% of all cases a (cookie) based session storage is simpler and "good enough".

In a globally scaling web application with millions of users, 
cookies can also be scaled in the backend by sharing the session 
data on database servers (e.g. [redis](https://redislabs.com/solutions/use-cases/session-management/),
Memcached or even MySql).

### When can / should I use JWT?

In case your "client" is not a browser.

As long as your client transmits its data over HTTPS, 
it doesn't matter which Auth mechanism you use. Even BasicAuth is good enough in most cases.

For APIs that have to be implemented for large corporations and governments, 
the use of JWT is recommended. In this landscape, one usually has the capacity 
and financial resources to deal with the technical complexity and difficult maintainability
that cryptographic tokens bring with it.

### Where to store the token?

We need to save our JWT token somewhere, so that we can forward it to our API as a header. 
You might be tempted to persist it in localstorage; you should not do it! 
This is prone to XSS attacks.

Creating cookies on the client to save the JWT will also be prone to XSS. 
If it can be read on the client from Javascript outside your app - it can be stolen. 

XSS is when users get unsafe JS running on your domain in other users browsers 
when that happens neither JWT in localStorage or sessions and JWT in cookies are safe. 
With `httpOnly` flag on cookies, you can't directly access them, but the browser will 
still send them with AJAX requests to your server. 
If this happens you generally out of luck. To prevent this, make sure to escape all 
user input if it's sent to the browser.

Another option is to store the token **in memory**. But then
the token will be nullified when the user switches between tabs.

If you load 3rd party JS with script tags or iframes this might compromise `localStorage`
unless you are careful, but I haven't worked enough with this to help you here.

You might think an `HttpOnly` cookie (created by the server instead of the client) 
will help, but cookies are vulnerable to CSRF attacks. 
It is important to note that `HttpOnly` and sensible CORS policies cannot prevent 
CSRF form-submit attacks and using cookies require a proper CSRF mitigation strategy.

If you still want to store a JWT as a cookie, then as a 
[SameSite cookie](https://web.dev/samesite-cookies-explained/).
Make the SameSite cookie `SameSite=Lax`, `HttpOnly` and `Secure`.

A cookie with the `HttpOnly` attribute is inaccessible 
to the JavaScript `Document.cookie` API; it is sent only to the server.

### How to implement a logout?

JSON Web Tokens are stateless. You can’t change a token to be invalid in 
a straightforward way. The easiest way to implement logging out is just 
to remove the token from the browser. Since the cookies that should be defined as 
`HttpOnly`, you need to create an endpoint that clears it.

A logout route, e.g. `GET /logout`:

```php
return $response->withHeader(
    'Set-Cookie', 
    'Authentication=; HttpOnly; Secure; Path=/; Max-Age=0'
);
```

### How does a refresh token work?

This tutorial does not cover refresh tokens. A refresh token requires 
some kind of state on the server-side and would go beyond the scope of this tutorial.

A [refresh token](https://openid.net/specs/openid-connect-core-1_0.html#rfc.section.12) 
is issued as part of authentication process along with the JWT. 

The request token endpoint can also use a Refresh Token by using the 
`grant_type` value `refresh_token`, as described in Section 6 of OAuth 2.0 RFC6749.

On the client, before the previous JWT token expires, you could wire up our app 
to make a `PUT /tokens` endpoint and grab a new JWT.

The auth server should save this refresh token and associates it to a 
particular user in its own database, so that it can handle the renewing JWT logic.

Anyway, there are certain additional aspects, that tend to get difficult or are even 
against the [fundamental ideas of JWT](https://auth0.com/blog/stateless-auth-for-stateful-minds/), 
you should consider before using JWTs as refresh-token, 
as this basically means you introduce long-living JWT. Keep in mind the beauty 
and security of the JWT concept lies within JWTs being short-lived.

If you still need refresh tokens, you may try the [Auth0 PHP SDK](https://auth0.com/docs/quickstart/webapp/php)

### How to implement a HTTP Basic or form data authentication?

The [OAuth 2 spec](https://tools.ietf.org/html/rfc6749#section-2.3.1) states 
that the username and password MAY uses the HTTP Basic authentication or form data.

Please write into the comments if you need more information about this topic.

Read more: <https://blog.restcase.com/4-most-used-rest-api-authentication-methods/>

### How to handle CORS with OPTIONS preflight requests?

Read more: [Slim 4 - CORS](https://odan.github.io/2019/11/24/slim4-cors.html)

### The `Authorization` header missing in POST request

If using Apache, add the following to the `.htaccess` file. 
Otherwise, PHP won't have access to the `Authorization` header.

```
RewriteRule .* - [env=HTTP_AUTHORIZATION:%{HTTP:Authorization}]
```

Read more: <https://stackoverflow.com/a/26791450/1461181>

### Is there a working library?

Yes, there are also some other libraries for PHP you could use instead:

* [Auth0 PHP SDK](https://auth0.com/docs/quickstart/webapp/php)
* [tuupola/slim-jwt-auth](https://github.com/tuupola/slim-jwt-auth)

The approach of this article is more middleware and routing "friendly",
while the `tuupola/slim-jwt-auth` approach uses an array to configure the different routes.
 
I think the array based configuration is not so good to maintain in the long run, 
for example when you add or change a route path you may miss changing the configuration
and suddenly some routes are unprotected.

I prefer to explicitly add the `JwtAuthMiddleware` to specific routes or route groups in `routes.php`. 
You can open the `routes.php` file see what is protected. This approach also makes it easier to 
fetch users from the database (see `TokenCreateAction`) instead of loading it from a fixed array.

I think you have to decide what's better for your specific use case.
