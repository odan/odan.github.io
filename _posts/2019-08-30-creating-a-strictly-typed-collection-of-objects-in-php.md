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
* [Creating a data object](#creating-a-data-object)
* [Creating a collection class](#creating-a-collection-class)
* [Usage](#usage)

## Intro

[Generics](https://wiki.php.net/rfc/generic-arrays) are still a long way off and whether 
they'll ever come like that is questionable.

Did you know that we actually do not need generics, because PHP 7.4+ has everything to 
implement type-safe and future-proof collection classes.

We only need the features everyone should know since PHP 7: [Classes](https://www.php.net/manual/en/language.oop5.php),
 arrays and [type declarations](https://www.php.net/manual/en/functions.arguments.php#functions.arguments.type-declaration). 

Type declarations were also known as "type hints" in PHP 5. 
With type declarations, methods can require that parameters be of a certain type at the time of the invocation. 
If the specified value is of the wrong type, an error occurred.

## Creating a data object

First, we create a data object to store our data in-memory:

```php
<?php

final class User
{
}
```

**Note:** [Typed class properties](https://wiki.php.net/rfc/typed_properties_v2) have been 
added in PHP 7.4 and provide a major improvement to PHP's type system.

## Creating a collection class

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
    public function addUser(User $user): void
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
$userList->addUser(new User());
$userList->addUser(new User());
```

Iterating over the collection:

```php
foreach ($userList->all() as $user) {
    echo sprintf("ID: %s, Username: %s\n", $user->id, $user->username);
}
```

You could also use variadics and pass many User objects in constructor at once like:

```php
$userList = new UserList(new User(), new User());
```
