---
title: PHPUnit - Mock global functions (like date, time) at runtime
layout: post
comments: true
published: false
description: 
keywords: 
---

## Example

```php
<?php

// Setup
// composer require --dev phpunit/phpunit
// composer require --dev php-mock/php-mock

namespace App\Model;

class ExampleModel
{

    public function doSomething()
    {
        $result = $this->myDate();
        $result .= $this->mySecondDate();
        return $result;
    }

    protected function myDate()
    {
        return date('Y-m-d H:i:s');
    }

    protected function mySecondDate()
    {
        return date('H:i:s');
    }

}

namespace App\Test;

use phpmock\MockBuilder;

/**
 * MethodMockTest
 */
class MethodMockTest extends \PHPUnit_Framework_TestCase
{

    public function testMockGlobalMethod()
    {
        $myFunction = function() {
            return 1417011228;
        };
        $builder = new MockBuilder();
        $builder->setNamespace(__NAMESPACE__)
                ->setName('time')
                ->setFunction($myFunction);

        $mock = $builder->build();

        // The mock is not enabled yet.
        $this->assertNotEquals(time(), 1417011228);

        $mock->enable();
        $this->assertEquals(time(), 1417011228);

        // The mock is disabled and PHP's built-in time() is called.
        $mock->disable();
        $this->assertNotEquals(time(), 1417011228);
    }

    public function testMockGlobalMethodInClass()
    {
        // Overwrite global date function
        $myFunction = function() {
            return 1;
        };
        $builder = new MockBuilder();
        $builder->setNamespace(dirname(\App\Model\ExampleModel::class))
                ->setName('date')
                ->setFunction($myFunction);

        $mock = $builder->build();
        $mock->enable();

        // Get stub
        $stub = $this->getMockBuilder(\App\Model\ExampleModel::class)
                // null: no methods will be replaced.
                ->setMethods(null)
                ->getMock();

        // Run test
        $this->assertEquals('11', $stub->doSomething());

        // Disable function mock
        $mock->disable();
    }

}
```