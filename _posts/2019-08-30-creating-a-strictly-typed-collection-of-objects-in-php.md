---
title: Creating a strictly typed collection of objects in PHP 
layout: post
comments: true
published: true 
description: 
keywords: php array generics
---

## Table of contents

* [Intro](#intro)
* [Creating a collection class](#creating-a-collection-class)
* [Usage](#usage)

## Intro

A type-checked array is an array that ensures that all values of an array are an instance of the
same [data type](https://www.php.net/manual/en/language.types.intro.php) (class).

Until today it is not possible directly with arrays and I think this is a missing feature in PHP.

There is an interesting RFC about [Generic arrays](https://wiki.php.net/rfc/generic-arrays), but whether and when this
feature will be added is still unclear.

However, you can also implement it with the existing language features.

## Creating a collection class

First, we create a class we want to add to the collection:

```php
<?php

final class User
{
}
```

Now we create a collection class to collect and retrieve our "array" of user objects.

Example:

```php
final class UserList
{
    /**
     * @var User[] The users
     */
    private $list;

    /**
     * The constructor.
     * 
     * @param User ...$user The users
     */
    public function __construct(User ...$user) 
    {
        $this->list = $user;
    }
    
    /**
     * Add user to list.
     *
     * @param User $user The user
     *
     * @return void
     */
    public function add(User $user): void
    {
        $this->list[] = $user;
    }

    /**
     * Get all users.
     *
     * @return User[] The users
     */
    public function all(): array
    {
        return $this->list;
    }
}
```

## Usage

```php
// Create a new (and empty) collection object
$userList = new UserList();

// Add some new User objects to the collection
$userList->add(new User());
$userList->add(new User());
```

Iterating over the collection:

```php
foreach ($userList->all() as $user) {
    // ...
}
```

You could also use variadics and pass many User objects in constructor at once like:

```php
$userList = new UserList(new User(), new User());
```

Passing an array of Users to the class constructor:

```php
$userList = new UserList(...[new User(), new User()]);
```

[Demo](https://3v4l.org/JNY1Z)