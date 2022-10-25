---
title: Collections in PHP
layout: post
comments: true
published: true 
description: 
keywords: php array generics collections
---

## Introduction

For many applications, you want to create and manage groups of related objects. Unfortunately, generics will never become part of the PHP programming language. For this reason I thought again about how at least a strictly typed list with specific data types or objects can be implemented, even without generics.

I already wrote a [blog post](https://odan.github.io/2019/08/30/creating-a-strictly-typed-collection-of-objects-in-php.html) about this topic a few years ago,
but this time I want to present an even simpler approach.

As goal, I have defined the following criteria.

* No dependencies (no library must be used)
* Write as less code as possible
* No fancy language features should be used
* It must be type safe
* The collection must be iterable using `foreach`
* It must be compatible with the [PHPStorm code completion](https://www.jetbrains.com/help/phpstorm/auto-completing-code.html) feature
* It must comply [PHPStan level 9](https://phpstan.org/user-guide/rule-levels)

## Implementing a String Collection

For the sake of simplicity, I want to start with `string` collection.

The idea is to create a class that is able to add new items to 
an internal array of strings. For this purpose, I add a method called `add`.

```php
<?php

class StringCollection
{
    /** @var string[] */
    private array $list = [];

    public function add(string $string): void
    {
        $this->list[] = $string;
    }
}
```

The next challenge is to read the list of strings using a [foreach](https://www.php.net/manual/en/control-structures.foreach.php) loop.


To achieve this, we need to implement a method that returns something
that can be iterated. Luckily, PHP itself provides the [IteratorAggregate ](https://www.php.net/manual/en/class.iteratoraggregate.php) interface for such an use case.

The implementation is quite simple, add the `IteratorAggregate` to your collection class and implement the `getIterator` method.

```php
<?php

use ArrayIterator;
use IteratorAggregate;
use Traversable;

class StringCollection implements IteratorAggregate
{
    /** @var string[] */
    private array $list = [];

    public function add(string $string): void
    {
        $this->list[] = $string;
    }

    public function getIterator(): Traversable
    {
        return new ArrayIterator($this->list);
    }
}
```

This `ArrayIterator` allows us iterating over arrays and objects.
In this case we are iterating over an array of strings.

The current implementation would already work, but PHPStan would now complain with the following error message:

```php
Class StringCollection implements generic interface 
IteratorAggregate but does not specify its types: TKey, TValue
You can turn this off by setting 
checkGenericClassInNonGenericObjectType: false in your phpstan.neon.
```

To fix this, we need to tell PHPStan which type of collection we are dealing with by adding a DocBlock to that class.

```
/**
 * @implements IteratorAggregate<int, string>
 */
 ```

This means that our collection returns a list of strings as value and an integer as key. The complete collection class, with a namespace, then looks like this:

```php
<?php

namespace App\Collection;

use ArrayIterator;
use IteratorAggregate;
use Traversable;

/**
 * @implements IteratorAggregate<int, string>
 */
class StringCollection implements IteratorAggregate
{
    /** @var string[] */
    private array $list = [];

    public function add(string $string): void
    {
        $this->list[] = $string;
    }

    public function getIterator(): Traversable
    {
        return new ArrayIterator($this->list);
    }
}
 ```

That's it, only 26 lines of code for the implementation of a strictly typed and iterable collection class.
 
**Usage**

The usage of the collection class is quite simple:

```php

$strings = new StringCollection();

$strings->add('foo');
$strings->add('var');

foreach ($strings as $value) {
    echo $value . "\n";
}
```

**Output**

```
foo
bar
```

## Implementing an Object Collection

That approach works with objects too.

Let's say you want to implement a collection for a class called `User`:

```php
<?php

class User
{
    public string $username;
    public string $email;
}
```

The Collection class can be implemented as follows:

```php
<?php

use ArrayIterator;
use IteratorAggregate;
use Traversable;

/**
 * @implements IteratorAggregate<int, User>
 */
class UserCollection implements IteratorAggregate
{
    /** @var User[] */
    private array $list = [];

    public function add(User $user): void
    {
        $this->list[] = $user;
    }

    public function getIterator(): Traversable
    {
        return new ArrayIterator($this->list);
    }
}
```

**Usage**

```php

$users = new UserCollection();

$users->add(new User());
$users->add(new User());

foreach ($users as $user) {
    // ...
}
```

Note that this is just an pseudo example. You need to fill the User
objects with real data before you add it to the collection. How you
pass the data into your objects depends on your specific implementation.

## Conclusion

With this simple but effective approach it is possible to fulfill the criteria mentioned above. I hope that this could inspire some people to use collection classes instead of arrays in the future.

## Read more

* [Why we can't have generics in PHP — Generics in PHP](https://www.youtube.com/watch?v=BN0L2MBkhNg)
* [What I would change about PHP](https://stitcher.io/blog/php-reimagined-part-2)