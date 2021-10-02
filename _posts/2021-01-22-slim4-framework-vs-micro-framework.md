---
title: Slim 4 - Framework vs. Microframework
layout: post
comments: true
published: true
description:
keywords: php, slim, framework, laravel, symfony
image: 
---

## Table of contents

* [Introduction](#introduction)
* [Microframework](#microframework)
* [Full-stack Framework](#full-stack-framework)
* [Slim](#slim)
* [Evaluating Slim](#evaluating-slim)
* [Community](#community)
* [Bundles, Packages and Plugins](#bundles-packages-and-plugins)
* [Standards](#standards)
* [Conclusion](#conclusion)

## Introduction

Over the last few years, I've noticed that there's a lot of confusion (and false expectations) around 
the Slim microframework. 
In the following chapters, I will try to explain the conceptual 
differences between full-stack frameworks and microframeworks.
You should not consider this comparison as "Slim vs. Laravel", 
but rather as framework vs. micro-framework in general.

## Microframework

A **microframework** is a term used to refer to minimalistic web application frameworks.
It is contrasted with full-stack frameworks.

According to Wikipedia a microframework typically...

> ... facilitates receiving an HTTP request, routing the HTTP request 
> to the appropriate controller, dispatching the controller, and returning an HTTP response. 

Microframeworks are often specifically designed for building the APIs for another service 
or application. For example, the Slim microframework is designed for Microservices development 
 and API development.

It **lacks most of the functionality** which is common to expect in a full-fledged web application framework, such as:

* Database abstraction
* Web template engine
* Input validation and input sanitation
* Accounts, authentication, authorization, roles etc...

## Full-stack Framework

Web **frameworks** provide a standard way to build and deploy web applications on the web.

Many web frameworks provide libraries for database access, templating frameworks, 
and session management, and they often promote code reuse.

Although they often target development of dynamic websites, 
they are also applicable for web APIs.

Laravel and Symfony belong to the group of full-stack frameworks.

To be fair, Symfony has been rebuilt in recent years to the point where Symfony Flex 
could almost considered a microframework approach as well.

## Slim

Slim is a microframework.

Although the idea behind Slim is relatively simple, it still requires a lot of explanation.

You might know this Unix philosophy:

> **Make each program do one thing well. To do a new job, build afresh rather 
> than complicate old programs by adding new "features".**

According to this philosophy the Slim microframeworks will only
do one task right: **Routing and dispatching**.

This sounds simple, but in reality this is not so easy as you might think.
Depending on your technical background it may take a long period of time
to understand and successfully use this tool.

What’s the difference between **routing** and **dispatching**?

**Routing** is the process of taking a **URI path** and decomposing 
it into parameters to determine which module, controller, 
and action of that controller should receive the request.
Routing occurs only once: when the request is initially received and before 
the first controller is dispatched.

**Dispatching** is the process of taking the request object, extracting the module name, 
controller name, action name, and optional parameters contained in it, and then 
instantiating a controller and calling an action of that controller.

In Slim all this happens when you add a route as follows:

```php
use Psr\Http\Message\ResponseInterface as Response;
use Psr\Http\Message\ServerRequestInterface as Request;
use Slim\Factory\AppFactory;

$app = AppFactory::create();

$app->get('/', function (Request $request, Response $response) {
    $response->getBody()->write('Hello, World!');
    
    return $response;
});

$app->run();
```

## Evaluating Slim

Besides the technical requirements, developers should also be "fit" enough for Slim.
Here is a list of resources you should already be familiar with before evaluating Slim.

**Minimum Skillset for Slim:**

* [PHP 7+](https://en.wikipedia.org/wiki/PHP)
    * [OOP](https://www.php.net/manual/en/language.oop5.php)
    * [Object Interfaces](https://www.php.net/manual/en/language.oop5.interfaces.php)  
    * [Namespaces](https://www.php.net/manual/en/language.namespaces.php)
    * [Factory pattern](https://phptherightway.com/pages/Design-Patterns.html#factory)
* [HTTP Basics](https://www3.ntu.edu.sg/home/ehchua/programming/webprogramming/HTTP_Basics.html)
  * [HTTP Request and Response](https://developer.mozilla.org/en-US/docs/Web/HTTP/Messages)
  * [URL - Uniform Resource Locator](https://en.wikipedia.org/wiki/URL)
* [Linux File System Basics](https://www.dummies.com/computers/operating-systems/linux/linux-file-system-basics/)
  * [The Linux file system differs from the Windows file system](https://www.howtogeek.com/137096/6-ways-the-linux-file-system-is-different-from-the-windows-file-system/)
* [Apache Module mod_rewrite](https://httpd.apache.org/docs/2.4/mod/mod_rewrite.html)
* [Dependency management with Composer](https://getcomposer.org/doc/00-intro.md)
* [The PSR-4 Autoloader](https://www.php-fig.org/psr/psr-4/)
* [PSR-7: HTTP message interfaces](https://www.php-fig.org/psr/psr-7/)
* [PSR-11: Container interface](https://www.php-fig.org/psr/psr-11/)
* [PSR-15: HTTP Server Request Handlers](https://www.php-fig.org/psr/psr-15/)
* [PSR-17: HTTP Factories](https://www.php-fig.org/psr/psr-17/)
* [Middleware](https://www.slimframework.com/docs/v4/concepts/middleware.html)
* [Dependency injection](https://en.wikipedia.org/wiki/Dependency_injection)
* [Front controller](https://phptherightway.com/pages/Design-Patterns.html#front-controller)
* [Model-View-Controller (MVC)](https://phptherightway.com/pages/Design-Patterns.html#front-controller)  
  * [MVC Model 2](https://de.wikipedia.org/wiki/Model_2)
* [PHPUnit](https://en.wikipedia.org/wiki/PHPUnit)
* [Debugging with Xdebug](https://en.wikipedia.org/wiki/Xdebug)
* [F12 - Developer Toolbar](https://developers.google.com/web/tools/chrome-devtools)

If you find something you feel not familiar with, please learn it **before** you try Slim.
But that was just the beginning. You should also have a stable
knowledge of security, design patterns and all [SOLID](https://en.wikipedia.org/wiki/SOLID) principles to build secure and maintainable software.

Read more: [Architecture](https://odan.github.io/learn-php/#architecture)

**Is Slim the right tool for me? When should I consider Slim and when not?**

You might consider a microframework like Slim if you:

* meet the "Minimum Skillset for Slim" list (see above)
* need performance, efficiency, full control and flexibility
* feel experienced enough, and you know what you are doing
* really value standard interfaces like PSR-7 and PSR-15
* want to build long-lasting software
* want to prevent vendor lock-in
* have special requirements and want exchange components
* want to follow industry standards and best practices (like dependency injection)
* want to build a "SOLID" software
* stay up to date with web development and security related topics
* know how to build and maintain complex software
* feel familiar with enterprise architecture concepts and patterns

You might consider a full-stack framework if you:

* cannot fulfill all the points in the above list
* need support from a big community.  
* don't care about vendor lock-in
* accept the risk of upgrading to the next framework version 
* don't care about high performance response times (SEO)
* prefer to use bad and slow OOP/SQL abstractions like an ([N+1](https://stackoverflow.com/a/97253/1461181)) ORM  
* prefer framework specific "bundles" or "plugins" over pure composer packages
* need tons of online resources like tutorials, videos, courses and certifications
* want to write something into your CV
* and your team need a rigid structure in order to function
* don't have the time or want to take the risk to decide which component to use
* don't want to learn something new, like the PSR interfaces and middleware concept
* don't feel safe enough in terms of web security
* prefer to install a bunch of IDE plugins to make your framework usable

## Community

Compared to Laravel and Symfony, the Slim community is actually very small.

Good support is very important, especially for beginners.
The Laravel and Symfony community is very big and willing to help as fast as possible. 
They also provide tons of tutorials, videos, online courses and certificates.

Especially beginners struggle to understand Slim and think it's "simple" and
so it must easy for them. This is not true. **Slim is simple but not easy.**
Slim cannot be learned in an "afternoon". Also PHP and development in general 
cannot be learned in an afternoon. This leads to a strange situation, 
because most support requests don't concern Slim itself.
In the last few years, most of these support issues are about teaching 
people how to configure their webserver, especially Apache with `mod_rewrite`.
The second most common questions are about basic PHP knowledge such as classes, 
namespaces, and autoloading (PSR-4). What I try to say is: Slim is not for beginners.

## Bundles, Packages and Plugins

A general-purpose framework can integrate other plugins or bundles based on a fixed structure. 

In Slim, however, such an ecosystem cannot emerge because this “plugin mechanism” is missing. 
In Slim however each individual component can be integrated with the help of “Composer” 
and some container definitions. This is more complex, but also more independent and flexible.

## Standards

The [PHP-FIG](https://www.php-fig.org/) is a group of experts trying to find ways for us to work better together.
For this reason, this group has also developed various specifications for frameworks
such as PSR-7, PSR-15 etc. Unfortunately, the big frameworks like Laravel and
Symfony are not yet ready to implement these standards, probably to strengthen
their own ecosystem (a.k.a. vendor lock-in).

On the other hand, Slim (and Laminas) provides full support for most standard [PSR 
interfaces](https://www.php-fig.org/psr/) to ensure very good interoperability between other components. 
This ensures that you are able to replace the HTTP message or container component 
"relatively" easily without risking too much dependency on a particular vendor.

## Conclusion

I think that Slim does exactly what a microframework should do: routing and dispatching. 
The DI container is optional, but recommended. If that's still too much or too magical for you,
you can take a look at [nikic/FastRoute](https://github.com/nikic/FastRoute).

Everything is relative, even complexity. 
Everything in life has its advantages and disadvantages. We have to find the right balance. 
Everyone must decide for themselves what suits them better and what does not. 
Luckily, we have the freedom to decide this for ourselves.
