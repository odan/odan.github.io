---
title: Creating your first Slim 3 Framework Application Part 2
layout: post
comments: true
published: true
description: 
keywords: php slim
---

The Slim Framework is a great micro framework for web application, RESTful API's and websites. 
This Tutorial shows how to create a very basic but flexible project for every use case.

## Table of contents

* [Requirements](#requirements)
* [Introduction](#introduction)
* [Single Action Controllers](#single-action-controllers)
* [Dependency injection](#dependency-injection)
* [Repositories](#repositories)
* [Business logic](#business-logic)
* [Validation](#validation)
* [Handling Ajax requests](#handling-ajax-requests)
* [Conclusion](#conclusion)

## Requirements

* PHP 7+
* MySQL 5.7+
* Slim Framework 3

## Introduction

In the [first chapter](https://odan.github.io/2017/11/30/creating-your-first-slim-framework-application.html) we created a 
very basic app template. Now it's time to extend this project and add more logic and a more scalable architecture.

## Single Action Controllers

In real life we have to deal with a lot more then only 3 or 5 routes (url endpoints). In bigger applications 100 and more 
routes are very common. Another point is how to manage and organize all the different routings in a modern and OOP way. 

However, one thing is clear: with the Slim callbacks we can't get any further here, 
because the routing definitions including their code would simply get too big. 

In addition, the code must be made ready for refactoring so that changes to the code can be made later. 

Last but not least controller classes should stick to the SRP principle and do only one thing and 
not several things at the same time.

In classic MVC frameworks, several action methods are normally implemented for each controller class. 
The problem is that each action has different dependencies, although only one action is called at a time. 
A conventional controller class therefore has too many dependencies that must be resolved unnecessarily by the application. 

Too much responsibility therefore violates the SRP principle.

For this reason, we reduce the responsibility of a controller class to a single task (route) by providing 
the controller class with exactly one public method for processing the request and response. 

Here is an example of a very simple single action controller:

File: `src/Action/PingAction.php`

```php
namespace App\Action;

use Psr\Http\Message\ResponseInterface;
use Slim\Http\Request;
use Slim\Http\Response;

final class PingAction
{
    public function __invoke(Request $request, Response $response): ResponseInterface
    {
        return $response->withJson(['success' => true]);
    }
}
```

To register a new action handler we add a routing definition in `routes.php` like this:

```php
$app->any('/ping', \App\Action\PingAction::class);
```

The advantage of this syntax is that our IDE now knows exactly which class is responsible for the 
route and provides us with full refactoring capabilities.

The static code analysis using `phpstan` will also benefit from this technique.

### Dependency injection

Ok, this is a bit of a controversial topic, because Slim doesn't recommend any particular way to implement it.

However, it is clear that all necessary dependencies must be injected via the constructor. 
It is important that in no case the container itself is given, but only the explicit dependencies.

By default the Slim framework will inject the container into the constructor of the Action class for us.

> Injecting a container inside something is an anti-pattern and must be avoided.

Of course, we try to avoid this mistake by injecting all dependencies explicitly.

With Pimple this is very easy and straightforward.

**Example**

Let's create a new action class for the first url route (`/`) that will render a twig template for us.

File: `src/Action/HomeIndexAction.php`

```php
<?php

namespace App\Action;

use Psr\Http\Message\ResponseInterface;
use Slim\Http\Request;
use Slim\Http\Response;
use Slim\Views\Twig;

final class HomeIndexAction implements ActionInterface
{
    /**
     * @var Twig
     */
    private $twig;

    /**
     * Constructor.
     *
     * @param Twig $twig
     */
    public function __construct(Twig $twig)
    {
        $this->twig = $twig;
    }

    /**
     * Action.
     *
     * @param Request $request the request
     * @param Response $response the response
     *
     * @return ResponseInterface the response
     */
    public function __invoke(Request $request, Response $response): ResponseInterface
    {
        $viewData = [
            'name' => 'World',
        ];

        return $this->twig->render($response, 'Home/home-index.twig', $viewData);
    }
}
```

Create a twig template file: `templates/Home/home-index.twig`

**Content:**

{% raw %}
```twig
Hello {{ name }}!
```
{% endraw %}

Then we map the url to the action class in `routes.php`:

```php
$app->get('/', \App\Action\HomeIndexAction::class);
```

To implement correct dependency injection, we explicitly inject the dependencies into the constructor.

In `container.php` add this definition:

```php
$container[\App\Action\HomeIndexAction::class] = function (Container $container) {
    $twig = $container->get(\Slim\Views\Twig::class);
    return new \App\Action\HomeIndexAction($twig);
};
```

> If you don't want to write this manually, you can use the [PHP-DI Bridge for Slim](https://github.com/PHP-DI/Slim-Bridge).

Ok you can see, this was very easy. If we open the index page we should see the the content: `Hello World!`.

## Repositories

The `Active Record Pattern` is an so called anti-pattern, because it voilates the Single Responsibility Principle of SOLID.

Instead we are implementing our persistend oriented Repositores as a [Data Mapper](https://designpatternsphp.readthedocs.io/en/latest/Structural/DataMapper/README.html), which follows the Single Responsibility Principle.

A repository improves code maintainability, testing and readability by separating `business logic` from `data access logic` and provides centrally managed and consistent access rules for a data source. Each public repository method represents a query. The return values represent the result set of a query and can be primitive/object or list (array) of them. Database transactions must be handled on a higher level (domain service) and not within a repository.

Quick summary:

* Communication with the database.
* Place for the data access logic (query logic).
* This is no place for the business logic! Use domain services for the complex business and domain logic.

### Example

A concrete repository which is responsible for user data could look like this:

Filename: `src/Domain/User/UserRepository.php`

```php
<?php

namespace App\Domain\User;

use DomainException;
use PDO;

final class UserRepository
{

    private $pdo;

    public function __construct(PDO $pdo)
    {
        $this->pdo = $pdo;
    }
    
    public function getUserById(int $userId): array
    {
        $statement = $this->pdo->prepare('SELECT * FROM users WHERE id = :userId');
        $statement->execute(['userId' => $userId]);
        $row = $statement->fetch();

        if (empty($row)) {
            throw new DomainException(sprintf('User not found: %s', $userId));
        }

        return $row ?: [];
    }
}
```

**Return types**

In some cases, it makes more sense to return only a simple array of data or objects. In other cases (like an insert), it makes more sense to simply return the last inserted ID as an integer. For deletions, it is recommended to return only a boolean value such as `true`.

**Method names**

Use the prefix `get` to indicate that this method must return a value, otherwise throw an exception.
If the select method can return an empty result set, the method name starts with `find` and doesn't throws an exception.

**Examples**

* findUserById(int $userId): ?User
* getUserById(int $userId): User
* findActiveUsers(): array  (array of Users, e.g. User[])
* insertUser(array $data): int (last inserted ID)
* deleteUser(int $userId): bool (success)
* updateUser(int $userId, array $data): bool (success)

## Business logic

The business logic (e.g. calulation, validation, file creation etc.) should be placed in service classes (domain service).

The domain service object is invoked directly by the controller layer (in our case the action method).

Normally, a service object reads or changes data from the database. 
For this reason, we must provide the service object with a `Repository`.
You could consider these two objects (service and repository) as bundled, because they form a kind of symbiosis. 

**Example**

Filename: `src/Domain/User/UserService.php`

```php
<?php

namespace App\Domain\User;

final class UserService
{
    /**
     * @var UserRepository
     */
    private $userRepository;

    /**
     * Constructor.
     *
     * @param UserRepository $userRepository The repository
     */
    public function __construct(UserRepository $userRepository)
    {
        $this->userRepository = $userRepository;
    }

    /**
     * Get user by ID.
     *
     * @param int $userId The user ID
     *
     * @return array The data
     */
    public function getUserById(int $userId): array
    {
        // do complex things here, like validation, logging, calculation, 
        // external api requests, caching, generate csv, excel, pdf files and so on.
        
        // ...
        
        $userRow = $this->userRepository->getUserById($userId)
        
        // ...
        
        return $userRow;
    }
}

```

## Validation

For security reasons, server-side validation is a MUST. Client-side validation is useful for the usability.

There are countless concepts and validation libraries to help you validate the potentially dangerous user input.

Slim does not have a validation functionality.

We have to decide for ourselves which is the best solution for our specific requirements. 

One of the most popular validation engines for PHP is [Respect\Validation](https://github.com/Respect/Validation).
You can try it and maybe it's a good solution for you.

Personally, I don't use a special validation library, because PHP itself offers more flexible and better possibilities for input validation. The only solution that **really works** for me in a large enterprise application was [Martin Fowler's validation concept](https://martinfowler.com/articles/replaceThrowWithNotification.html).

I have written a small library in PHP to collect all vallidation errors: [selective/validation](https://github.com/selective-php/validation).

If you like to read more about this topic, please contact me or write a comment.

## Handling Ajax requests

Today we have multiple different possiblilities to invoke an Ajax request from the client side like jQuery, Axios or the native `fetch` function. I still prefer jQuery because it's bulletproof and simply works in any browser.

### Client side

Invoking a server side action can be done in jQuery with the [ajax](http://api.jquery.com/jquery.ajax/) function:

Sample `POST` request:

```javascript
const data = {
    username: "max",
    email: "max@example.com"
};

$.ajax({
    url: 'users',
    type: "POST",
    contentType: 'application/json',
    dataType: 'json',
    data: JSON.stringify(data)
}).done(function (data) {
    alert('Success');
}).fail(function (xhr) {
    alert('Server error');
});
```

## Conclusion

Remember the relationships.

* Slim - As routing framework
* Single Action Controllers - To invoke the correct service method (domain)
* Dependency injection - For SOLID code and testability
* Service classes - To handle business logic
* Repositories - Data access logic
* Validation - User input handling
