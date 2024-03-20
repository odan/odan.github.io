---
title: PHP - Using the PSR-20 Clock
layout: post
comments: true
published: true
description:
keywords: 
image: 
---

Handling time can be a tricky task, especially when it comes to testing 
and ensuring predictable results. PHP provides various functions like 
`time()`, `microtime()` and the `\DateTimeImmutable` class to fetch the 
current time, but the problem is when `\DateTimeImmutable` is hard-coded into logic, 
unit tests will not be reliable. This is where the `ClockInterface` comes into play.

## What is the ClockInterface?

The `ClockInterface` is a concept that aims to provide a standardized 
way to obtain time in PHP applications. It offers a solution not 
only for accessing the real system time but also for mocking time 
in scenarios where predictable results are required, such as during testing. 
By using the `ClockInterface`, developers can avoid the need to 
resort to PHP extensions or complex workarounds,
like redeclaring the `time()` function within a local namespace.

The clock interface defines the most basic operations to read the 
current time and date from the clock. 
It returns the current time as a `\DateTimeImmutable` object.

```php
<?php

namespace Psr\Clock;

use DateTimeImmutable;

interface ClockInterface
{
    public function now(): DateTimeImmutable;

}
```

The following implementation show how the `ClockInterface` can be 
used to fetch the current time from the system.

```php
<?php

namespace Example;

use DateTimeImmutable;
use Psr\Clock\ClockInterface;

final class SystemClock implements ClockInterface
{
    public function now(): DateTimeImmutable
    {
        return new DateTimeImmutable();
    }
}
```

The `FrozenClock` can be used to fetch create a "frozen" time instance 
for testing purposes.

```php
<?php

namespace Example;

use DateTimeImmutable;
use Psr\Clock\ClockInterface;

final class FrozenClock implements ClockInterface
{
    private DateTimeImmutable $now;

    public function __construct(DateTimeImmutable $now)
    {
        $this->now = $now;
    }

    public function now(): DateTimeImmutable
    {
        return clone $this->now;
    }
}
```

Of course, this was just an example, and in reality you could just
install a package that already provides such functionality, for example
the `symfony/clock` package.

## Why Bother with ClockInterface?

You might be wondering why we need yet another method for dealing 
with time in PHP applications. The key reason is **interoperability**. 
While there are existing libraries that tackle this issue, 
they often come with their own clock interfaces, creating a lack of standardization.

Popular libraries, such as Carbon and its fork Chronos, 
offer mocking capabilities through a static `setTestNow()` method. 
While this can be useful, it lacks isolation and requires 
calling the method again to stop mocking.

## Usage

It's easier to explain the usage of the usage with a real example.
So let's try out the `symfony/clock` package.

Open your console and navigate to your project's root directory, 
then run the following command:

```
composer require symfony/clock
```

The retrieve the current system time, you can use the 
`Symfony\Component\Clock\Clock` class as follows:

```php
use Symfony\Component\Clock\Clock;

$clock = new Clock();
$now = $clock->now();

// The current date and time
echo $now->format('Y-m-d H:i:s');
```

For testing purposes, you can use the `Symfony\Component\Clock\MockClock` class,
which will return always the same time.

```php
use Symfony\Component\Clock\MockClock;

$clock = new MockClock('2024-01-31 15:45:59');
$now = $clock->now();

// A fix value: 2024-01-31 15:45:59
echo $now->format('Y-m-d H:i:s');
```

## Using Dependency Injection

You can now use dependency injection to provide a `ClockInterface`
implementation to objects requiring access to the current time.

**Example**

```php
<?php

namespace MyNamespace;

use Psr\Clock\ClockInterface;

final class MyService
{
    private ClockInterface $clock;

    public function __construct(ClockInterface $clock)
    {
        $this->clock = $clock;
    }

    public function doSomething(): void
    {
        $now = $this->clock->now();
        $formattedTime = $now->format('Y-m-d H:i:s');

        echo "[$formattedTime] $message\n";
    }
}

```

In this class, the `ClockInterface` is injected through the constructor, 
allowing you to use it to get the current time.

## Defining Services in a DI Container

To use the `ClockInterface` within your class, you need to tell
the DI container what implementation should be used and injected.

Here's an example of how you can define the `ClockInterface::class` using PHP-DI:

```php
<?php

use Symfony\Component\Clock\Clock;
use Psr\Clock\ClockInterface;

return [
    ClockInterface::class => function () {
        return new Clock();
    },
];

```

Now, the DI container will inject the Symfony `Clock` instance into your application classes.

By defined different `ClockInterface` implementations within the DI container,
you can control whether your application uses the system clock or a 
mocked / frozen clock, which is useful for testing and other scenarios.

To replace the default implementation, you can use the `set` 
method of the DI container as follows:

```php
use Psr\Clock\ClockInterface;
use Symfony\Component\Clock\MockClock;

$now = '2024-01-31 00:00:00';
$timezone = 'UTC';

$container->set(ClockInterface::class, new MockClock($now, $timezone));
```

In practice, you would better put that code into a test trait for
better reusability.

```php
<?php

namespace App\Test\Traits;

use DateTimeImmutable;
use DateTimeZone;
use Psr\Clock\ClockInterface;
use Symfony\Component\Clock\MockClock;

/**
 * PSR-20 Test Clock.
 */
trait ClockTestTrait
{
    private function setTestNow(DateTimeImmutable|string $now = 'now', DateTimeZone|string $timezone = null): void
    {
        $this->container->set(ClockInterface::class, new MockClock($now, $timezone));
    }
}

```

Note: This example assumes that you have a `$this->container` member variable
in your PHPUnit test class with the applications DI container instance.

**Usage**

```php
use ClockTestTrait;
// ...

// Within the phpunit test method
$this->setTestNow('2014-02-01 14:45:30');
```

This approach allows you to decouple your code from specific clock implementations, 
making it more flexible, maintainable, and suitable for different use cases.

## Conclusion

In conclusion, the `ClockInterface` is a valuable addition, 
simplifying time handling, enabling easier testing, 
and promoting interoperability between packages.

It's a step towards more reliable and maintainable PHP applications 
that can accurately manage time-related tasks.

