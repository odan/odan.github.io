---
title: Slim 4 - CakePHP Validation
layout: post
comments: true
published: true
description: 
keywords: php, validation, slim, cakephp
image: https://odan.github.io/assets/images/slim-logo-600x330.png
---

## Table of contents

* [Requirements](#requirements)
* [Introduction](#introduction)
* [Installation](#installation)
* [Usage](#usage)
* [Translations](#translations)
* [Validation Middleware](#validation-middleware)
* [Conclusion](#conclusion)
* [Read more](#read-more)

## Requirements

* PHP 7.3+
* [A Slim 4 application](https://odan.github.io/2019/11/05/slim4-tutorial.html)

## Introduction

The [CakePHP validation package](https://packagist.org/packages/cakephp/validation) provides features to build validators that can 
validate arbitrary arrays of data with ease.

## Installation

To install the component, run:

```
composer require cakephp/validation
```

## Usage

I think the most common use case is to validate nested arrays, e.g form data. 
To fetch the form data as an array use this code in your action class:

```php
$formData = (array)$request->getParsedBody();
```

Then pass this array into your service for validation. Within a service class
you are able to perform the validation.

For this example we want to validate our form data according to this rules:

* `username`: **required**, min. length is 3, the max. length is 60
* `password`: **required**, min. length is 8, the max. length is 60
* `email`: **required**, must be a valid email address
* `mobile`: *optional*, must be a valid phone number

The `Validator` object defines the rules that apply to a set of fields
and contains a mapping between fields and validation sets. 

Creating a validator is simple:

```php
use Cake\Validation\Validator;
// ...

$validator = new Validator();

$validator
    ->requirePresence('username', 'This field is required')
    ->notEmptyString('username', 'Username is required')
    ->minLength('username', 3, 'Too short')
    ->maxLength('username', 60, 'Too long')
    ->requirePresence('password', 'This field is required')
    ->notEmptyString('password', 'Password is required')
    ->minLength('password', 8, 'Too short')
    ->maxLength('password', 60, 'Too long')
    ->requirePresence('email', 'This field is required')
    ->email('email', false, 'E-Mail must be valid')
    ->notEmptyString('mobile', 'Mobile number must not be empty')
    ->regex('mobile', '/^\+[0-9]{6,}$/', 'Invalid mobile number');

$errors = $validator->validate($formData);
if ($errors) {
    // Throw validation exception
    // ...
}
```

By default, all fields are optional. Once you add the 'requirePresence' rule, the
element must always be present. To disallow empty strings, you can add the rule `notEmptyString`.

There are a lot of predefined rules. 
Here you can find a list of [available validation rules in the API](https://api.cakephp.org/4.0/class-Cake.Validation.Validation.html).

More specific (and precise) rules can be added using a custom regex.

```php
->regex('mobile', '/^\+[0-9]{6,}$/', 'Invalid mobile number');
```

```php
->regex('password', '/[0-9]+/', 'Password must include at least one number!')
->regex('password', '/[a-zA-Z]+/', 'Password must include at least one letter!');
```

It is possible to add a custom callback function if the default validation methods do not fit your needs.
If the callback functions return `true` the validation has passed and `false` if the validation of the
field has failed.

```php
$validator
    ->add(
        'field_name',
        'field_name',
        [
            'rule' => function ($value) {
                return my_custom_function($value);
            },
            'message' => 'Invalid',
        ]
    );
```

To start the validation use the `validate` method.

```php
$errors = $validator->validate($formData);
```

As result, you should get a list of validation errors like this:

```php
array (
  'username' => 
  array (
    '_required' => 'This field is required',
  ),
  'password' => 
  array (
    '_required' => 'This field is required',
  ),
  'email' => 
  array (
    '_required' => 'This field is required',
  ),
)
```

With this result you should be able to render the error messages into a twig template.

If you implement a RESTful API, you usually don't use a server-side template engine (like Twig).
In this case I would recommend throwing a `ValidationException` to catch it in a special `ValidationExceptionMiddleware`.
 
For this purpose we add a [selective/validation](https://github.com/selective-php/validation) component 
to our application. Run:

```php
composer require selective/validation
```

The `selective/validation` component is able to convert the result from the `$validator->validate()` 
method into a proper `ValidationResult` object using the `CakeValidationErrorCollector`.

Change the code example from above just a little bit: 

```php
use Cake\Validation\Validator;
use Selective\Validation\Factory\CakeValidationFactory;
use Selective\Validation\Exception\ValidationException;
// ...

$validationFactory = new CakeValidationFactory();
$validator = $validationFactory->createValidator();

$validator
    ->requirePresence('username', 'This field is required')
    ->notEmptyString('username', 'Username is required')
    ->minLength('username', 3, 'Too short')
    ->maxLength('username', 60, 'Too long')
    ->requirePresence('password', 'This field is required')
    ->notEmptyString('password', 'Password is required')
    ->minLength('password', 8, 'Too short')
    ->maxLength('password', 60, 'Too long')
    ->requirePresence('email', 'This field is required')
    ->email('email', false, 'E-Mail must be valid')
    ->notEmptyString('mobile', 'Mobile number must not be empty')
    ->regex('mobile', '/^\+[0-9]{6,}$/', 'Invalid mobile number');

// Convert validator errors to ValidationResult
$validationResult = $validationFactory->createValidationResult(
    $validator->validate($data)
);

if ($validationResult->fails()) {
    throw new ValidationException('Validation failed. Please check your input.', $validationResult);
}
```

Optionally, you can also perform more complex validations and append them to the validation result:

```php
if ($this->existsUsername($formData['username'])) {
    $validationResult->addError('username', 'Username is already taken');
}
```

## Translations

To translate the strings for the error messages, just use the `_` or `__` function of gettext.

```php
$validator
    ->requirePresence('username', __('This field is required'))
    ->notEmptyString('username', __('Username is required'))
    ->minLength('username', 3, __('Too short'))
    ->maxLength('username', 60, __('Too long'))
    // ...
```

## Validation Middleware

As soon as the validation fails a `ValidationException` will be thrown to 
transform this specific exception into an JSON response with status code 422.

The `ValidationExceptionMiddleware` middleware catches all exceptions and 
converts it into a nice JSON response.

Insert a container definition for the `ValidationExceptionMiddleware::class`:

```php
<?php

use Psr\Container\ContainerInterface;
use Psr\Http\Message\ResponseFactoryInterface;
use Selective\Validation\Encoder\JsonEncoder;
use Selective\Validation\Middleware\ValidationExceptionMiddleware;
use Selective\Validation\Transformer\ErrorDetailsResultTransformer;
use Slim\App;
use Slim\Factory\AppFactory;
// ...

return [
    ValidationExceptionMiddleware::class => function (ContainerInterface $container) {
        $factory = $container->get(ResponseFactoryInterface::class);

        return new ValidationExceptionMiddleware(
            $factory, 
            new ErrorDetailsResultTransformer(), 
            new JsonEncoder()
        );
    },

    ResponseFactoryInterface::class => function (ContainerInterface $container) {
        $app = $container->get(App::class);

        return $app->getResponseFactory();
    },

    App::class => function (ContainerInterface $container) {
        AppFactory::setContainer($container);

        return AppFactory::create();
    },

    // ...

];
```

Add the `ValidationExceptionMiddleware` into your middleware stack:

```php
<?php

use Selective\Validation\Middleware\ValidationExceptionMiddleware;
use Slim\Factory\AppFactory;

require_once __DIR__ . '/../vendor/autoload.php';

$app = AppFactory::create();

// ...

$app->add(ValidationExceptionMiddleware::class);

// ...

$app->run();
```

## Conclusion

The `CakePHP Validation` component is very fast, flexible and provides some useful rules out of the box.

Now you should be able to validate even complex API and form data.

## Read more

* [The CakePHP Validation Github repository](https://github.com/cakephp/validation)
* [The CakePHP Validation documentation](https://book.cakephp.org/4/en/core-libraries/validation.html)
* [A list of available Validation rules in the API](https://api.cakephp.org/4.0/class-Cake.Validation.Validation.html#).
* [Slim forum](https://discourse.slimframework.com)
* [Ask a question or report an issue](https://github.com/odan/slim4-tutorial/issues)
