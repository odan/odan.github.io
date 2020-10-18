---
title: Slim 4 - Respect/Validation
layout: post
comments: false
published: true
description: 
keywords: php validaton slim
---

## Table of contents

* [Requirements](#requirements)
* [Introduction](#introduction)
* [Installation](#installation)
* [Usage](#usage)
* [Conclusion](#conclusion)
* [Read more](#read-more)

## Requirements

* PHP 7.3+
* [A Slim 4 application](https://odan.github.io/2019/11/05/slim4-tutorial.html)

## Introduction

The `Respect/Validation` component is a popular and flexible validation library for PHP.
It works as standalone libary and without much additional dependencies.



## Installation

To install Respect/Validation, run:

```
composer require respect/validation
```

## Usage

I think the most common use case is to validate nested arrays. 
For this reason I will show how to validate a simple array with this fields:

* `username`: **required**, min. length is 3, the max. length is 60
* `password`: **required**, min. length is 8, the max. length is 60
* `email`: **required**, must be a valid email address
* `mobile`: *optional*, must be a valid phone number

To fetch the form data as an array use this code in your action class:

```php
$formData = (array)$request->getParsedBody();
```

Then pass this array into your service for validation. Within a service class
you are able to perform the validation as follows:

```php
use Respect\Validation\Validator as v;
// ...

$validator = new v();

$validator->addRule(v::key('username', v::allOf(
    v::notEmpty()->setTemplate('The username must not be empty'),
    v::length(3, 24)->setTemplate('Invalid length')
))->setTemplate('The key "username" is required'));

$validator->addRule(v::key('email', v::allOf(
    v::notEmpty()->setTemplate('The email must not be empty'),
    v::email()->setTemplate('Invalid email address')
))->setTemplate('The key "email" is required'));

$validator->addRule(v::key('password', v::allOf(
    v::notEmpty()->setTemplate('The password must not be empty'),
    v::length(8, 60)->setTemplate('Invalid length')
))->setTemplate('The key "password" is required'));
//...
```

Optional fields can be added like this. Just pass `false` to make the key optional:

```php
$validator->addRule(v::key('mobile', v::allOf(
    v::notEmpty()->setTemplate('The mobile number must not be empty'),
    v::phone()->setTemplate('Invalid phone number')
), false));
```

To start the validation use the `assert` method.

```php
$validator->assert($formData);
```

## Transaltions

To translate the strings for the error messages, just use the `_` or `__` function of gettext.

```php
$validator->addRule(v::key('username', v::allOf(
    v::notEmpty()->setTemplate(__('The username must not be empty')),
    v::length(3, 24)->setTemplate(__('Invalid length'))
))->setTemplate(__('The key %s is required', 'username'));
```

## Validation Middleware

As soon as the validation fails a `NestedValidationException` will be thrown.

This is great, because you can transform this specific exception into an JSON response with status code 422.

For this purpose we add a custom middleware to the stack.

Create a new file `src/middleware/RespectValidationMiddleware.php` and copy / paste this content:

```php
<?php

namespace App\Middleware;

use Psr\Http\Message\ResponseFactoryInterface;
use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface;
use Psr\Http\Server\MiddlewareInterface;
use Psr\Http\Server\RequestHandlerInterface;
use Respect\Validation\Exceptions\NestedValidationException;

final class RespectValidationMiddleware implements MiddlewareInterface
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
        try {
            return $handler->handle($request);
        } catch(NestedValidationException $exception) {
            $messages = [];
            /** @var ValidationException $message */
            foreach($exception->getIterator() as $message) {
                $key = $message->getParam('name');
                if($key === null) {
                    continue;
                }
                $messages[$key] = $message->getMessage();
            }
            
            $response = $this->responseFactory->createResponse();
        
            $result = [
                'error' => [
                    'message' => $exception->getMessage(),
                    'details' => $messages,
                ],
            ];
            $response->getBody()->write(json_enode($result));
            $response->withHeader('Content-Type', 'application/json');

            return $response->withStatus(422);
        }
    }
}

```

In order to make it work, the `RespectValidationMiddleware::class` must be added after the Slim ErrorMiddleware:

```php
<?php

use App\Middleware\RespectValidationMiddleware;

// ...
$app->addErrorMiddleware(true, true, true);
$app->add(RespectValidationMiddleware::class); // <-- here
```

## Conclusion

Respect\Validation is very flexible and provides a lot of rules out of the box.
It's maintained and the download number on github is impressive.

A lot of magic is going on when you use the static method of the validator class.
I prefer less magic and more rubust and faster code. For me a real
fluent interface would be more readable and elegant, but this is just a matter of taste.

It was quite hard for me to find out how to add a custom message and translations
to each validation rule. In the end it worked.

In my next article I will show how to use the [CakePHP validation library](https://github.com/cakephp/validation) in Slim.

## Read more

* [The Respect\Validation Github repository](https://github.com/Respect/Validation)
* [The Respect\Validation documentation](https://respect-validation.readthedocs.io/en/latest/)
* [What makes Respect\Validation awesome?](https://www.reddit.com/r/PHP/comments/1telis/respectvalidation_the_most_awesome_validation/ce7hvcs/?utm_source=reddit&utm_medium=web2x&context=3)
* [Slim forum](https://discourse.slimframework.com)
* [Feedback and support](https://github.com/odan/support/issues)
