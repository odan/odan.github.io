---
title: phpunit - Modify a method/function at runtime
layout: post
comments: true
published: false
description: 
keywords: php
---

## Example

```php
<?php

namespace App\Test;

class SomeClass
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

/**
 * ExampleTest
 */
class ExampleTest extends \PHPUnit_Framework_TestCase
{

    public function testStub()
    {
        $stub = $this->getMockBuilder(SomeClass::class)
            ->setMethods(array('myDate', 'mySecondDate'))
            ->getMock();

        // Always return fix value for myDate method
        $stub->method('myDate')
            ->willReturn('foo');

        // Always return value from callback function for mySecondDate method.
        $stub->method('mySecondDate')
            ->willReturnCallback(array($this, 'mySecondDateCallback'));

        $this->assertEquals('foo123', $stub->doSomething());
    }

    public function mySecondDateCallback()
    {
        // my custom code
        return '123';
    }
}
```