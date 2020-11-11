---
title: Slim 4 - Testing
layout: post
comments: true
published: true
description: 
keywords: slim, slimphp, php, test, testing, phpunit, integration
---

## Table of contents

* [Requirements](#requirements)
* [Introduction](#introduction)
* [Installation](#installation)
* [Unit Tests](#unit-tests)
* [Integration Tests](#integration-tests)
  * [Container Setup](#container-setup)
  * [HTTP Tests](#http-tests)
* [Read more](#read-more)

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
both integration- and unit tests.

This article explains how to integrate PHPUnit as testing framework, 
but won't cover PHPUnit itself, which has its own 
excellent [documentation](https://phpunit.readthedocs.io/).

## Installation

Before creating the first test we are installing [phpunit](https://phpunit.de/) 
as development dependency with the `--dev` option:

```
composer require phpunit/phpunit --dev
```

Each test - whether it's a unit test or a integration test - 
is a PHP class that should live in the `tests/TestCase/` directory of your application. 

Create a new `tests/TestCase/` directory in your project root.

Open `composer.json` and the following scripts:

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
<phpunit bootstrap="vendor/autoload.php" 
    colors="true" 
    backupGlobals="false" 
    backupStaticAttributes="false">
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

Unit tests ensure that individual components of the app work as expected.

A unit test is a test against a single PHP class, also called a unit. 
If you want to test the overall behavior of your application, 
see the section about [Integration Tests](#integration-tests).

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
namespace App\Test\TestCase\Util;

use App\Util\Calculator;
use PHPUnit\Framework\TestCase;

class CalculatorTest extends TestCase
{
    public function testAdd(): void
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

## Integration Tests

Integration tests ensure that component collaborations work as expected. 

* Test a repository against the actual database
* Test an application service using a real controller action
* Test a HTTP integration against the real webservice

Assertions may test the HTTP API, or side-effects such as database, 
filesystem, datetime, logging etc.

Depending on your needs, you can choose to run your test against 
a integration database (with fixtures) or only against a mocked repository.
Some people prefer to create repository interfaces and replace them with an empty 
implementation for testing. Of course you can do that, 
but the technical effort for testing is much higher then.

### Container setup

To be able to perform complete and realistic integration tests 
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
use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface;
use Psr\Http\Message\UriInterface;
use Slim\App;
use Slim\Psr7\Factory\ServerRequestFactory;
use UnexpectedValueException;

trait AppTestTrait
{
    /** @var Container */
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

        $this->container->set($class, $mock);

        return $mock;
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
        return (new ServerRequestFactory())
            ->createServerRequest($method, $uri, $serverParams);
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

    /**
     * Verify that the given array is an exact match for the JSON returned.
     *
     * @param ResponseInterface $response The response
     * @param array $expected The expected array
     *
     * @return void
     */
    protected function assertJsonData(
        ResponseInterface $response, 
        array $expected
    ): void {
        $actual = (string)$response->getBody();
        $this->assertJson($actual);
        $this->assertSame($expected, (array)json_decode($actual, true));
    }
}
```

### HTTP Tests

HTTP testing allows you to verify your API endpoints. This includes the 
infrastructure supported by the app, such as the database, file system, and network.

HTTP tests have a very specific workflow:

* Make a request (click on a link or submit json data)
* Test the response
* Clean up and repeat

All HTTP requests are performed in memory.
We don't need a http client (like Guzzle) or a webserver to perform the requests.

Ok, now add your first API test. Let's assume that you have 
implemented a RESTful API with Slim.

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

    /**
     * Test.
     *
     * @dataProvider provideUserReaderAction
     *
     * @param UserData $user The user
     * @param array $expected The expected result
     *
     * @return void
     */
    public function testUserReaderAction(UserData $user, array $expected): void
    {
        // Mock the repository resultset
        $this->mock(UserReaderRepository::class)
            ->method('getUserById')->willReturn($user);

        // Create request with method and url
        $request = $this->createRequest('GET', '/users/1');

        // Make request and fetch response
        $response = $this->app->handle($request);

        // Asserts
        $this->assertSame(200, $response->getStatusCode());
        $this->assertJsonData($response, $expected);
    }

    /**
     * Provider.
     *
     * @return array The data
     */
    public function provideUserReaderAction(): array
    {
        $user = new UserData();
        $user->id = 1;
        $user->username = 'admin';
        $user->email = 'john.doe@example.com';
        $user->firstName = 'John';
        $user->lastName = 'Doe';

        return [
            'User' => [
                $user,
                [
                    'user_id' => 1,
                    'username' => 'admin',
                    'first_name' => 'John',
                    'last_name' => 'Doe',
                    'email' => 'john.doe@example.com',
                ]
            ]
        ];
}
```

The PHPUnit documentation contains more information about 
[Data Providers](https://phpunit.readthedocs.io/en/9.0/writing-tests-for-phpunit.html#data-providers)

Now run all tests:

```
composer test
```

To test a JSON endpoint you can use the `createJsonRequest` method, e.g.:

```php
$request = $this->createJsonRequest('POST', '/users', ['name' => 'Sally']);
$response = $this->app->handle($request);
```

This HTTP test doesn't hit the database and is very fast. 
If you also want to test against the database, you should read my next blog post about database testing.
 
Stay tuned.

## Read more

* [The Practical Test Pyramid](https://martinfowler.com/articles/practical-test-pyramid.html)
* [Testing: Unit vs Integration vs Regression vs Acceptance](http://www.getlaura.com/testing-unit-vs-integration-vs-regression-vs-acceptance/)
* [The Differences Between Unit Testing, Integration Testing And Functional Testing](https://www.softwaretestinghelp.com/the-difference-between-unit-integration-and-functional-testing/)
* [A Testing Strategy for Hexagonal Applications](https://www.youtube.com/watch?v=LtbHAFsEu5g)
* <https://github.com/odan/slim4-tutorial>
