---
title: Slim 4 - Testing
layout: post
comments: true
published: true
description: 
keywords: slim, slimphp, php, test, testing, phpunit
---

## Table of contents

* [Requirements](#requirements)
* [Introduction](#introduction)
* [Installation](#installation)
* [Unit Tests](#unit-tests)
* [HTTP Tests](#http-tests)

## Requirements

* PHP 7.2+
* Composer (dev environment)
* [A Slim 4 application](https://odan.github.io/2019/11/05/slim4-tutorial.html)

This topic assumes a basic understanding of unit tests and phpunit.
If unfamiliar with phpunit, read the [Getting Started with PHPUnit](https://phpunit.de/getting-started-with-phpunit.html)
tutorial and its linked content.

## Introduction

Whenever you write a new line of code, you also potentially add new bugs. 
To build better and more reliable applications, you should test your code using 
both functional and unit tests.

This article explains how to integrate PHPUnit as testing framework, 
but won't cover PHPUnit itself, which has its own 
excellent [documentation](https://phpunit.readthedocs.io/).

This article covers the most used test topics: Unit tests and HTTP tests.

In the future I plan to write about other topics like database tests, 
browser tests and console tests.

## Installation

Before creating the first test we are installing [phpunit](https://phpunit.de/) 
as development dependency with the `--dev` option:

```
composer require phpunit/phpunit --dev
```

Each test - whether it's a unit test or a functional test - 
is a PHP class that should live in the `tests/TestCase/` directory of your application. 

Create a new `tests/TestCase/` directory in your project root.

Open `composer.json` and the the following scripts:

```json
"scripts": {
    "test": "phpunit --configuration phpunit.xml",
    "test:coverage": "phpunit --configuration phpunit.xml --coverage-clover build/logs/clover.xml --coverage-html build/coverage"
}
```

The code coverage output directory is: `build/coverage/`

PHPUnit is configured by the `phpunit.xml` file in the root of your Slim application.

Create a new file `phpunit.xml` and copy/paste this configuration:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<phpunit bootstrap="vendor/autoload.php" colors="true" backupGlobals="false" backupStaticAttributes="false">
    <testsuites>
        <testsuite name="Tests">
            <directory suffix="Test.php">tests</directory>
        </testsuite>
    </testsuites>
    <filter>
        <whitelist processUncoveredFilesFromWhitelist="false">
            <directory suffix=".php">src</directory>
            <exclude>
                <directory>vendor</directory>
                <directory>build</directory>
            </exclude>
        </whitelist>
    </filter>
</phpunit>
```

Try executing PHPUnit by running :

```
composer test
```

To start all tests with code coverage, run:

```
composer test:coverage
```

If you receive the error message `Error: No code coverage driver is available`
you have to [enable Xdebug](https://gist.github.com/odan/1abe76d373a9cbb15bed).

To enable Xdebug, locate or create the `[XDebug]` section in the `php.ini` 
file and update it as follows:

```ini
[XDebug]
zend_extension = "<path to xdebug extension>"
xdebug.remote_autostart = 1
xdebug.remote_enable = 1
```

## Unit Tests

A unit test is a test against a single PHP class, also called a unit. 
If you want to test the overall behavior of your application, 
see the section about [Functional Tests](#functional-tests).

Suppose, for example, that you have an incredibly simple class called `Calculator` in 
the `src/Util/` directory of the app:

```php
namespace App\Util;

class Calculator
{
    public function add(int $a, int $b): int
    {
        return $a + $b;
    }
}
```

To test this, create a `CalculatorTest` file in the `tests/TestCase/Util`
directory of your application:

```php
// tests/TestCase/Util/CalculatorTest.php
namespace App\Tests\Util;

use App\Util\Calculator;
use PHPUnit\Framework\TestCase;

class CalculatorTest extends TestCase
{
    public function testAdd()
    {
        $calculator = new Calculator();
        $result = $calculator->add(30, 12);

        // assert that your calculator added the numbers correctly!
        $this->assertEquals(42, $result);
    }
}
```

By convention, the `tests/TestCase/` directory should replicate the directory of your 
module for unit tests. So, if you're testing a class in the `src/Util/` directory, 
put the test in the `tests/TestCase/Util/` directory.

Now run all tests:

```
composer test
```

## HTTP Tests

HTTP testing allows you to verify your API endpoints. This includes the 
infrastructure supported by the app, such as the database, file system, and network.

They are no different from unit tests as far as PHPUnit is concerned, 
but they have a very specific workflow:

* Make a request (click on a link or submit json data)
* Test the response
* Clean up and repeat

All HTTP request will run in-memory without a webserver.

Depending on your needs, you can choose to run your test against 
a test database (with fixtures) or only against a mocked data set (data provider).

### Container setup

To be able to perform complete and realistic integration- and functional tests 
we have to setup the container (PSR-11) for each test first.
The advantage is that we can also test the complete middleware stack
and use the autowire functionality of the depenency injection container.

The following trait will bootstrap the Slim application with the depenency injection container
and provides some convenient methods for mocking and creating http requests.

Create a new file `tests/AppTestTrait.php` and copy/paste this content:

```php
<?php

namespace App\Test;

use DI\Container;
use InvalidArgumentException;
use PHPUnit\Framework\MockObject\Builder\InvocationMocker;
use PHPUnit\Framework\MockObject\MockObject;
use Psr\Container\ContainerInterface;
use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface;
use Psr\Http\Message\UriInterface;
use Slim\App;
use Slim\Psr7\Factory\ServerRequestFactory;
use UnexpectedValueException;

trait AppTestTrait
{
    /** @var ContainerInterface|Container */
    protected $container;

    /** @var App */
    protected $app;

    /**
     * Bootstrap app.
     *
     * @throws UnexpectedValueException
     *
     * @return void
     */
    protected function setUp(): void
    {
        $this->app = require __DIR__ . '/../config/bootstrap.php';

        $container = $this->app->getContainer();
        if ($container === null) {
            throw new UnexpectedValueException('Container must be initialized');
        }

        $this->container = $container;
    }

    /**
     * Add mock to container.
     *
     * @param string $class The class or interface
     *
     * @return MockObject The mock
     */
    protected function mock(string $class): MockObject
    {
        if (!class_exists($class)) {
            throw new InvalidArgumentException(sprintf('Class not found: %s', $class));
        }

        $mock = $this->getMockBuilder($class)
            ->disableOriginalConstructor()
            ->getMock();

        if ($this->container instanceof Container) {
            $this->container->set($class, $mock);
        }

        return $mock;
    }

    /**
     * Create a mocked class method.
     *
     * @param array|callable $method The class and method
     *
     * @return InvocationMocker The mocker
     */
    protected function mockMethod($method): InvocationMocker
    {
        return $this->mock((string)$method[0])->method((string)$method[1]);
    }

    /**
     * Create a server request.
     *
     * @param string $method The HTTP method
     * @param string|UriInterface $uri The URI
     * @param array $serverParams The server parameters
     *
     * @return ServerRequestInterface
     */
    protected function createRequest(
        string $method,
        $uri,
        array $serverParams = []
    ): ServerRequestInterface {
        return (new ServerRequestFactory())->createServerRequest($method, $uri, $serverParams);
    }

   /**
     * Create a JSON request.
     *
     * @param string $method The HTTP method
     * @param string|UriInterface $uri The URI
     * @param array|null $data The json data
     *
     * @return ServerRequestInterface
     */
    protected function createJsonRequest(
        string $method, 
        $uri, 
        array $data = null
    ): ServerRequestInterface {
        $request = $this->createRequest($method, $uri);

        if ($data !== null) {
            $request = $request->withParsedBody($data);
        }

        return $request->withHeader('Content-Type', 'application/json');
    }
}
```

Ok, now add your first API test. 

Let's assume that you have implemented a RESTful API with Slim Framework.

Create a new file `tests/TestCase/Action/UserReaderActionTest.php`
and add this code to test the endpoint `GET /users/1`:

```php
<?php

namespace App\Test\TestCase\Action;

use App\Domain\User\Data\UserData;
use App\Domain\User\Repository\UserReaderRepository;
use App\Test\AppTestTrait;
use PHPUnit\Framework\TestCase;

class UserReaderActionTest extends TestCase
{
    use AppTestTrait;

    public function testUserReaderAction(): void
    {
        // Mock the repository resultset
        // It could also be an array or a primitive data type
        // Better use the @dataProvider annotation
        $user = new UserData();
        $user->id = 1;
        $user->username = 'admin';
        $user->email = 'john.doe@example.com';
        $user->firstName = 'John';
        $user->lastName = 'Doe';

        $this->mockMethod([UserReaderRepository::class, 'getUserById'])->willReturn($user);

        $request = $this->createRequest('GET', '/users/1');
        $response = $this->app->handle($request);

        $this->assertSame(200, $response->getStatusCode());

        $body = (string)$response->getBody();
        $this->assertStringContainsString(
            '{"user_id":1,"username":"admin","first_name":"John","last_name":"Doe","email":"john.doe@example.com"}',
            $body
        );
    }
}
```

Pleae note: In real life, you should pass the test data with 
[Data Providers](https://phpunit.readthedocs.io/en/9.0/writing-tests-for-phpunit.html#data-providers).

Now run all tests:

```
composer test
```

To test a JSON endpoint you can use the `createJsonRequest` method, e.g.:

```php
$request = $this->createJsonRequest('POST', '/users', ['email' => 'user@example.com']);
$response = $this->app->handle($request);
```

This HTTP test doesn't hit the database and is very fast. 
If you also want to test against a real database, you should read my next blog post. Stay tuned.

## Read more

* <https://martinfowler.com/articles/practical-test-pyramid.html>
* <http://www.getlaura.com/testing-unit-vs-integration-vs-regression-vs-acceptance/>
* <https://github.com/odan/slim4-tutorial>