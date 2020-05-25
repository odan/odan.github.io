---
title: Slim 4 - Error handling
layout: post
comments: true
published: true
description: 
keywords: slim, slimphp, php, monolog, logging
---

## Table of contents

* [Requirements](#requirements)
* [Introduction](#introduction)
* [Catching PHP warnings, notices and errors](#catching-php-warnings-notices-and-errors)
* [Catching 404 not found errors](#catching-404-not-found-errors)
* [Catching invalid HTTP requests](#catching-invalid-http-requests)
* [Shutdown Handler](#shutdown-handler)

## Requirements

* PHP 7.2+
* Composer (dev environment)
* A Slim 4 application
* [A Monolog LoggerFactory](https://odan.github.io/2020/05/25/slim4-logging.html)

## Introduction

Several types of errors can occur in a web application. 

Depending on the type, different strategies must be chosen for catching and handling them. 

For example, how to handle exceptions using the integrated middleware is already well 
described in the documentation. 

Therefore I would like to focus on special types of errors that are not yet fully documented.

Please note, that you may not need all of these error handler in your app. Just pick what you realy need.

But before we start you should install a PSR-3 logger implementation, 
like [Monolog](https://seldaek.github.io/monolog/).
 
## Catching PHP warnings, notices and errors

Besides [Throwable](https://www.php.net/manual/en/class.throwable.php)
PHP has its older error handling system of warnings, hints and errors.

The differences are explained here: <http://www.php.net/manual/en/errorfunc.constants.php>

Specifficaly:

* **Errors:** Fatal run-time errors. These indicate errors that can not be recovered from, such as a memory allocation problem. Execution of the script is halted.
* **Warnings:** Run-time warnings (non-fatal errors). Execution of the script is not halted.
* **Notices:** Run-time notices. Indicate that the script encountered something that could indicate an error, but could also happen in the normal course of running a script.

To set a custom error handler PHP provides the function set_error_handler.

Instead of invoking this functions direclty it would be better to wrap it into a middleware:

**Example**

File: `src/Middleware/ErrorHandlerMiddleware.php`

Source code:

```php
<?php

namespace App\Middleware;

use App\Factory\LoggerFactory;
use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface;
use Psr\Http\Server\MiddlewareInterface;
use Psr\Http\Server\RequestHandlerInterface;
use Psr\Log\LoggerInterface;

/**
 * Middleware.
 */
final class ErrorHandlerMiddleware implements MiddlewareInterface
{
    /**
     * @var LoggerInterface
     */
    private $logger;

    /**
     * The constructor.
     *
     * @param LoggerFactory $loggerFactory The logger
     */
    public function __construct(LoggerFactory $loggerFactory)
    {
        $this->logger = $loggerFactory
            ->addFileHandler('errors.log')
            ->createInstance('error_handler_middleware');
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
        $errorTypes = E_ALL;

        // Set custom php error handler
        set_error_handler(
            function ($errno, $errstr, $errfile, $errline) {
                switch ($errno) {
                    case E_USER_ERROR:
                        $this->logger->error("Error number [$errno] $errstr on line $errline in file $errfile");
                        break;
                    case E_USER_WARNING:
                        $this->logger->warning("Error number [$errno] $errstr on line $errline in file $errfile");
                        break;
                    default:
                        $this->logger->notice("Error number [$errno] $errstr on line $errline in file $errfile");
                        break;
                }

                // Don't execute PHP internal error handler
                return true;
            },
            $errorTypes
        );

        return $handler->handle($request);
    }
}

```

In order to make it work, the `ErrorHandlerMiddleware::class` must be added before the Slim ErrorMiddleware:

```php
<?php

use App\Middleware\ErrorHandlerMiddleware;
// ...

$app->add(ErrorHandlerMiddleware::class); // <-- here

$app->addErrorMiddleware(true, true, true, $logger);
```

To see if it works, try running code like this in your code base:

```php
$array = [];
$test = $array['nada'];
```

The expected logfile output should look like this:

```
[2020-05-25 10:10:05] app.NOTICE: Error number [8] Undefined index: nada on line 30 in file filename.php [] []
``` 

## Catching invalid HTTP requests

You may already have received the following error message.


`Bad Request: Header name must be an RFC 7230 compatible string.`

This can happen because the PSR-7 implementation doesn't support numeric header names.

Since `nyholm/psr7` v1.3.0  numeric header names are supported.
You just need to update to the latest version and the issue is fixed.
 
As a workaround (for all other PSR-7 implementations) 
you can try to catch all exceptions of the `$app->run()` method.

```php
<?php

// bootstrap code ...

try {
    $app->run();
} catch (Throwable $exception) {
    http_response_code(400);
    echo sprintf('Bad Request: %s', $exception->getMessage());
}
```

### Read more

* <https://github.com/Nyholm/psr7/pull/149>
* <https://github.com/Nyholm/psr7/pull/34>
* [RFC 7230](https://tools.ietf.org/html/rfc7230#section-3.2.4)
* <https://discourse.slimframework.com/t/handle-invalidargumentexception-thrown-by-requestfactory/3677/3>

## Catching 404 not found errors

In Slim 3 you could add acustom "Not Found Handler" to handle routes that are not defined.

Since Slim 4 this is not possible to add a custom error handler to the `ErrorMiddleware`.

```php
$callableResolver = $app->getCallableResolver();
$responseFactory = $app->getResponseFactory();
$errorHandler = new HttpErrorHandler($callableResolver, $responseFactory);

// ...

$errorMiddleware = $app->addErrorMiddleware($displayErrorDetails, false, false);
$errorMiddleware->setDefaultErrorHandler($errorHandler);
```

## Shutdown Handler

```php
// Create Shutdown Handler
$shutdownHandler = new ShutdownHandler($request, $errorHandler, $displayErrorDetails);
register_shutdown_function($shutdownHandler);
```

### Read more

* <https://github.com/slimphp/Slim-Skeleton/blob/master/public/index.php#L60>



