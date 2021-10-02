---
title: Slim 4 - Testing
layout: post
comments: true
published: true
description: 
keywords: slim, php, test, testing, phpunit, integration
---

## Table of contents

* [Requirements](#requirements)
* [Introduction](#introduction)
* [Installation](#installation)
* [Test Traits](#test-traits)
* [Unit Tests](#unit-tests)
* [Mocking](#mocking)
* [Integration Tests](#integration-tests)
* [HTTP Tests](#http-tests)
* [Database Testing](#database-testing)
* [Conclusion](#conclusion)
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
    "test": "phpunit --configuration phpunit.xml --do-not-cache-result --colors=always",
    "test:coverage": "php -d xdebug.mode=coverage -r "require 'vendor/bin/phpunit';" -- --configuration phpunit.xml --do-not-cache-result --colors=always --coverage-clover build/logs/clover.xml --coverage-html build/coverage"
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
you have to [enable Xdebug](https://odan.github.io/2020/12/03/xampp-xdebug-setup-php8.html).

To enable Xdebug, locate or create the `[XDebug]` section in the `php.ini` 
file and update it as follows:

```ini
[XDebug]
zend_extension=xdebug
xdebug.mode=debug
xdebug.start_with_request=trigger
```

### Test Traits

To be able to perform complete and realistic integration tests 
we have to set up the container (PSR-11) for each test first.
The advantage is that we can also test the complete middleware stack
and use the autowire functionality of the dependency injection container.

Phpunit itself provides a lot of useful methods for testing, asserting and mocking
out of the box. But when you use a DI container,
then you also need some helper functions for HTTP- and database tests.

The `selective/test-traits` component provides a variety of helpful tools to
make it easier to test your Slim applications.

Installation:

```
composer require selective/test-traits --dev
```

The following trait will bootstrap the Slim application with the dependency 
injection container and provides some convenient methods for mocking 
and creating http requests.

Create a new directory: `tests/Traits`.

Create a new file `tests/Traits/AppTestTrait.php` and copy/paste this content:

```php
<?php

namespace App\Test\Traits;

use Selective\TestTrait\Traits\ContainerTestTrait;
use Selective\TestTrait\Traits\MockTestTrait;
use Slim\App;
use UnexpectedValueException;

trait AppTestTrait
{
    use ContainerTestTrait;
    use MockTestTrait;
    // add more traits here if needed

    protected App $app;

    protected function setUp(): void
    {
        $this->app = require __DIR__ . '/../../config/bootstrap.php';

        $container = $this->app->getContainer();
        if ($container === null) {
            throw new UnexpectedValueException('Container must be initialized');
        }

        $this->setUpContainer($container);
    }
}
```

## Unit Tests

Unit tests ensure that individual components of the app work as expected.

A unit test is a test against a single PHP class, also called a unit. 
If you want to test the overall behavior of your application, 
see the section about [Integration Tests](#integration-tests).

Suppose, for example, that you have an incredibly simple class called `Calculator` in 
the `src/Support/` directory of the app:

```php
namespace App\Support;

class Calculator
{
    public function add(int $a, int $b): int
    {
        return $a + $b;
    }
}
```

To test this, create a `CalculatorTest` file in the `tests/TestCase/Support`
directory of your application:

File: tests/TestCase/Support/CalculatorTest.php

```php
namespace App\Test\TestCase\Support;

use App\Support\Calculator;
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
module for unit tests. So, if you're testing a class in the `src/Support/` directory, 
put the test in the `tests/TestCase/Support/` directory.

Now run all tests:

```
composer test
```

## Mocking

When testing Slim applications, you may wish to "mock" certain aspects of your 
application, so they are not actually executed during a test. 
For example, when testing an HTTP endpoint that hits the database, 
you may wish to mock the repository method, so it's not actually accessing the database
during the test. 

Phpunit itself provides some useful methods for mocking out of the box.
But when you use a dependency injection container (PSR-11), then you also have to
set the mocked instances into the container. For this purpose I added
the `mock` helper method into the `MockTestTrait`.
This tiny helper primarily provides a convenience layer over the Phpunit MockObject, 
so you do not  have to manually make complicated mocking method calls. 

For example, if you want to mock the database, just use the `mock` method as follows:

```php
use App\Test\Traits\AppTestTrait;
use App\Domain\User\Repository\UserCreatorRepository;
use Selective\TestTrait\Traits\MockTestTrait;
// ...

class ExampleTest extends TestCase
{
    use AppTestTrait;
    
    public function test(): void
    {
        // Mock the required repository method
        $this->mock(UserCreatorRepository::class)
            ->method('insertUser')
            ->willReturn(1);
        // ...   
    }

}
```

**Warning: Try to avoid excessive mocking.**

Try to test and cover all the code you actually deploy.

The downside of mocking is that you are not testing and covering
the code that you actually deploy and run on your real system.
The phpunit code coverage can never be 100% when you mock your repositories.
and the test quality will never be as good as in real **integration tests**.

Another problem with mocking is that your tests become very large and complex, 
and the test needs to know all the implementation details that could change 
quickly during the next refactoring. Maintaining such mocked tests would 
become more difficult and expensive in the long run.

Mocking makes sense if you have external APIs that must not be touched when 
the test is running. So, for example, I would only mock HTTP requests for external APIs.

When you write tests for the Actions, better write **integration tests** that cover the Actions, 
the Services, and the Repositories all at once. Then your tests are easier 
to set up, and you cover all the code you actually deploy.

Conclusion: **Don't mock everything, it's an anti-pattern.**
If everything is mocked, are we really testing the production code? 
Don't hesitate to **not** mock!

## Integration Tests

Integration tests ensure that component collaborations work as expected. 

* Test a repository against the actual database
* Test an application service using a real controller action
* Test an HTTP endpoint against the real webservice

Assertions may test the HTTP API, or side effects such as database, 
filesystem, datetime, logging etc.

Depending on your needs, you can choose to run your test against 
an integration database (with fixtures) or only against a mocked repository.
Some people prefer to create repository interfaces and replace them with an empty 
implementation for testing. Of course, you can do that, 
but the technical effort for testing is much higher then.

## HTTP Tests

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
use App\Test\Traits\AppTestTrait;
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
        // Mock the repository result
        $this->mock(UserReaderRepository::class)
            ->method('getUserById')->willReturn($user);

        // Create request with method and url
        $request = $this->createRequest('GET', '/users/1');

        // Make request and fetch response
        $response = $this->app->handle($request);

        // Asserts
        $this->assertSame(200, $response->getStatusCode());
        $this->assertJsonData($expected, $response);
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

### Passing a query string

The [http_build_query](https://www.php.net/manual/en/function.http-build-query.php) can generate 
URL-encoded query strings. Example:

```php
$params = [
    'limit' => 10,
];

$url = sprintf('/users?%s', http_build_query($params));
// Result: /users?limit=10

$request = $this->createRequest('GET', $url);
```

## Database Testing

The `selective/test-traits` component also provides a variety of helpful tools to 
make it easier to test your database driven applications. 

The `DatabaseTestTrait` provides methods for all these stages of a database test:

* Import the database schema (table structure)
* Insert the fixtures (rows) required for the test.
* Execute the test
* Verify the state of the tables
* Cleanup the tables for each new test

Add the `DatabaseTestTrait` only to a phpunit test class 
where you want to write a database test:

```php

use App\Test\Traits\AppTestTrait;
use PHPUnit\Framework\TestCase;
use Selective\TestTrait\Traits\DatabaseTestTrait;
// ...

class UserCreateActionTest extends TestCase
{
    use AppTestTrait;
    use DatabaseTestTrait;
    // ...
}
```

Then invoke the `setUpDatabase` method and pass the full path to the sql schema file:

```php
protected function setUp(): void
{
    // ...
    
    $this->setUpDatabase(__DIR__ . '/../../resources/schema/schema.sql');
}
```

The `setUpDatabase` method installs the database schema into a phpunit specific test database.
The database trait fetches the `PDO::class` instance directly from the DI container. So make sure
that the DI container returns the connection from a testing-, and not from a development database.
You can do this by defining a different database name in an environment-specific configuration 
file or by "manually" placing a custom PDO instance in the DI container.

Example configuration file for phpunit: `config/local.testing.php`

```php
// Phpunit test database
$settings['db']['database'] = 'slim_skeleton_test';
```

### Database asserts

Assert a number of rows in a given table:

```php
$this->assertTableRowCount(1, 'users');
```

Assert the given row exists:

```php
$this->assertTableRowExists('users', 1);
```

Assert that the given row does not exist:

```php
$this->assertTableRowNotExists('users', 1);
```

Assert row values:

```php
$this->assertTableRow($expected, 'users', 1);
```

Assert a specific set of row values:

```php
$this->assertTableRow($expected, 'users', 1, ['email', 'url']);
```

```php
$this->assertTableRow($expected, 'users', 1, array_keys($expected));
```

Assert a specific value in a given table, row and field:

```php
$this->assertTableRowValue('1', 'users', 1, 'id');
```

Read single value from table by id:

```php
$password = $this->getTableRowById('users', 1)['password'];
```

### Test fixtures

Insert multiple fixtures at once:

```php
use App\Test\Fixture\UserFixture;

$this->insertFixtures([UserFixture::class]);
```

Insert manual fixtures:

```php
$this->insertFixture('tablename', $row);
```


## Conclusion

We have seen how to create all kinds of tests like unit- and integration tests with phpunit.

## Read more

* [The Practical Test Pyramid](https://martinfowler.com/articles/practical-test-pyramid.html)
* [Testing: Unit vs Integration vs Regression vs Acceptance](http://www.getlaura.com/testing-unit-vs-integration-vs-regression-vs-acceptance/)
* [The Differences Between Unit Testing, Integration Testing And Functional Testing](https://www.softwaretestinghelp.com/the-difference-between-unit-integration-and-functional-testing/)
* [A Testing Strategy for Hexagonal Applications](https://www.youtube.com/watch?v=LtbHAFsEu5g)
* <https://github.com/odan/slim4-tutorial>
* [Ask a question or report an issue](https://github.com/odan/slim4-tutorial/issues)
