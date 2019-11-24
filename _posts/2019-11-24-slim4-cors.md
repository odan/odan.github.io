---
title: Slim 4 - CORS setup
layout: post
comments: true
published: true
description:
keywords: php slim cors preflight
---

Please note that proper CORS is not just sending a couple of headers with every response.
The flowchart describes it well: <https://gluu.org/docs/ce/admin-guide/cors/>.

Here is a complete Slim 4 example application.

## Installation

Run:

```
composer require slim/slim:^4
composer require slim/psr7
```

The code for the front controller in: `public/index.php`

```php
<?php

use Psr\Http\Message\ResponseInterface as Response;
use Psr\Http\Message\ServerRequestInterface as Request;
use Psr\Http\Server\RequestHandlerInterface;
use Slim\Factory\AppFactory;
use Slim\Routing\RouteCollectorProxy;
use Slim\Routing\RouteContext;

require_once __DIR__ . '/../vendor/autoload.php';

$app = AppFactory::create();

//$app->setBasePath('/slim-cors');

// This middleware will append the response header Access-Control-Allow-Methods with all allowed methods
$app->add(function (Request $request, RequestHandlerInterface $handler): Response {
    $routeContext = RouteContext::fromRequest($request);
    $routingResults = $routeContext->getRoutingResults();
    $methods = $routingResults->getAllowedMethods();
    $requestHeaders = $request->getHeaderLine('Access-Control-Request-Headers');

    $response = $handler->handle($request);

    $response = $response->withHeader('Access-Control-Allow-Origin', '*');
    $response = $response->withHeader('Access-Control-Allow-Methods', implode(',', $methods));
    $response = $response->withHeader('Access-Control-Allow-Headers', $requestHeaders);

    // Optional: Allow Ajax CORS requests with Authorization header
    $response = $response->withHeader('Access-Control-Allow-Credentials', 'true');

    return $response;
});

// The RoutingMiddleware should be added after our CORS middleware so routing is performed first
$app->addRoutingMiddleware();

//
// The routes
//
$app->group('/api/v0', function (RouteCollectorProxy $group) {
    $group->get('/users', function (Request $request, Response $response): Response {
        $response->getBody()->write('List all users');

        return $response;
    });

    $group->post('/users', function (Request $request, Response $response): Response {
        // Retrieve the JSON data
        $parameters = (array)$request->getParsedBody();

        // Your code here
        $response->getBody()->write('Create user');

        return $response;
    });

    // Allow preflight requests for /api/v0/users
    // Due to the behaviour of browsers when sending a request,
    // you must add the OPTIONS method. Read about preflight.
    $group->options('/users', function (Request $request, Response $response): Response {
        // Do nothing here. Just return the response.
        return $response;
    });

    $group->get('/users/{id}', function (Request $request, Response $response, array $arguments): Response {
        $userId = (int)$arguments['id'];
        $response->getBody()->write(sprintf('Get user: %s', $userId));

        return $response;
    });

    $group->delete('/users/{id}', function (Request $request, Response $response, array $arguments): Response {
        $userId = (int)$arguments['id'];
        $response->getBody()->write(sprintf('Delete user: %s', $userId));

        return $response;
    });

    $group->put('/users/{id}', function (Request $request, Response $response, array $arguments): Response {
        // Your code here...
        $userId = (int)$arguments['id'];
        $response->getBody()->write(sprintf('Put user: %s', $userId));

        return $response;
    });

    $group->patch('/users/{id}', function (Request $request, Response $response, array $arguments): Response {
        $userId = (int)$arguments['id'];
        $response->getBody()->write(sprintf('Patch user: %s', $userId));

        return $response;
    });

    // Allow preflight requests for /api/v0/users/{id}
    $group->options('/users/{id}', function (Request $request, Response $response): Response {
        // Don't add any code here. Keep it unchanged.
        return $response;
    });
});

$app->run();
```

### Test script

```html
<html>

<script src="//ajax.googleapis.com/ajax/libs/jquery/3.4.1/jquery.min.js"></script>

<script>
$.ajax({
    // change the url here
    url: "http://localhost/api/v0/users",
    type: "GET",
    // Add the Authorization header if needed
    headers: {'Authorization' : 'Bearer 12345'},
    cache: false,
    // contentType: 'application/json',
    dataType: 'json'
}).done(function (data) {
    alert('Successfully');
    console.log(data);
}).fail(function (xhr) {
    var message = 'Server error';
    if (xhr.responseJSON && xhr.responseJSON.error.message) {
       message = xhr.responseJSON.error.message;
    }
    alert(message);
});
</script>
</html>
```

[Comments](https://gist.github.com/odan/c3c41fb8b4d6d6b943b79f69aa58f27e)