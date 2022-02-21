---
title: PHP - Best Practices 2019
layout: post
comments: true
published: true
description: 
keywords: php
---

I think it's time for a fresh and simplified blog post about the latest PHP Best Practices in 2019 / 2020. 
A lot has changed within the last time I have written about this topic in [2018](https://odan.github.io/2018/04/05/php-best-practices.html).

## Principles and rules

Here are the most important rules:

* [KISS](https://odan.github.io/learn-php/#kiss) - Keep it simple, stupid
* [YAGNI](https://odan.github.io/learn-php/#yagni) - You Arent Gonna Need It
* [DRY](https://odan.github.io/learn-php/#dry) - Don't repeat yourself
* [SOLID](https://odan.github.io/learn-php/#solid) - The First 5 Principles of Object Oriented Design
* [The Boy Scout Rule](https://deviq.com/boy-scout-rule/) - Leave your code better than you found it

## Coding styles

* Code must follow [PSR-1](https://www.php-fig.org/psr/psr-1/) 
and [PSR-12](https://www.php-fig.org/psr/psr-12/). *PSR-2 is deprecated.*
* Add full [PSR-5](https://github.com/php-fig/fig-standards/blob/master/proposed/phpdoc.md) DocBlocks for all classes and all methods.

## Naming

* Use english names only (class names, method names, comments, variables names, database table and field names, etc...).
* Use class suffixes / prefixes according to [PSR Naming Conventions](https://www.php-fig.org/bylaws/psr-naming-conventions/).
* Follow industry best practices for your directory and file structure: 
  * [thephpleague/skeleton](https://github.com/thephpleague/skeleton) - A skeleton repository for packages
  * [pds/skeleton](https://github.com/php-pds/skeleton) - Names for your root-level directories

## Common rules

* All methods must have [type declaration](https://www.php.net/manual/en/migration70.new-features.php) and return type declaration.
* Methods without return statement must declared with `void` as their return type.
* Class properties must have [typed properties](https://wiki.php.net/rfc/typed_properties_v2) (PHP 7.4+).
* Don't mix data types for parameters and return types, except for `nullable`.
* Don't `extend` classes or create `abstract` classes for the sake of "code reuse", except for traits with test code only.
* Create `final` classes by default, except you have to mock it (e.g. repositories).
* Create `private` methods by default. Don't create `protected` methods.
* All method parameters must be used.

## Dependency injection

* Use [composition over inheritance](https://en.wikipedia.org/wiki/Composition_over_inheritance).
* Declare all class dependencies only within the constructor.
* Don't inject the container (PSR-11). The service locator is an anti-pattern.
* A constructor can accept only dependencies as object.
* Scalar data types (string, int, float, array) are not allowed for the constructor. Pass them as parameter object.

## Tools

Use a static code analyzer to detect bugs and errors. For example:

* [phpstan](https://github.com/phpstan/phpstan)
* [Php Inspections EA Extended](https://plugins.jetbrains.com/plugin/7622-php-inspections-ea-extended-)
* [SonarLint Plugin](https://odan.github.io/2019/12/01/the-phpstorm-sonarlint-plugin.html)

## Read more

* [Object Design Style Guide](https://www.manning.com/books/object-design-style-guide?a_aid=object-design&a_bid=4e089b42)
* [Hexagonal Architecture](https://odan.github.io/learn-php/#hexagonal-architecture)
* <http://bestpractices.thecodingmachine.com/>
* <https://odan.github.io/learn-php/>