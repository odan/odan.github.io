---
title: Architecture
layout: post
comments: false
published: false
description:
keywords: architecture, adr, action domain responder, architecture, php
image: https://odan.github.io/assets/images/slim-logo-600x330.png
---

## Table of contents

* [Action Domain Responder](#action-domain-responder-adr)
* [Action](#action)
* [Responder](#responder)
* [Domain](#domain)
  * [Services](#services)
  * [Application- and Domain Services](#application--and-domain-services)
  * [Infrastructure Service](#infrastructure-service)
  * [Services and Data](#services-and-data)
* [Naming convention](#naming-convention)
* [Directory structure](#directory-structure)
* [Best practices](#best-practices)
* [Repositories](#repositories)
* [Data Transfer Objects](#data-transfer-objects-dto)
* [Types and enums](#types-and-enums)
* [Read more](#read-more)

## Action Domain Responder (ADR)

There a lot of possible architecture styles that can be used with Slim.
In my opinion the **Action Domain Responder (ADR)** pattern fits very 
well to a wide range of modern web applications.

**ADR** is  a modern interpretation, and the successor of **MVC model 2**.

**ADR** is a user interface pattern specifically intended for server-side applications operating in an over-the-network, request/response environment.

The responsibilities in ADR are clearly separated as listed below:

* **[Action](#action):** Mediates between Domain and Responder
* **[Domain](#domain):** The core application with the business logic.
* **[Responder](#responder):** Presentation logic. The Responder builds the HTTP response.

Most of the work is getting done in the Domain layer, while the
Action and Responder layer remains very thin. 

The Action invokes the required Service method and pushes the result
(from the Service) to the Responder.
The "shared" business logic will be implemented within the Service class and not in the Action layer.
The Service fetches and combines all the data from the database using Repositories.
Then the Responder transforms the result from the Service into a specific representation (e.g. JSON).

The following diagram shows the dependencies between the layers:

![image](https://user-images.githubusercontent.com/781074/103690436-022a9a80-4f95-11eb-98e1-cb8a7061a230.png)

### Request and Response

In Slim the request passes through a [middleware](https://www.slimframework.com/docs/v4/concepts/middleware.html)
stack (in and out):

> `Request > Front controller > Routing > Middleware > Action > Middleware > Response`

Here is a quick overview of the request/response cycle:

![image](https://user-images.githubusercontent.com/781074/67461691-3c34a880-f63e-11e9-8266-2119ac98f639.png)

<div style="page-break-after: always; visibility: hidden"> 
\pagebreak 
</div>

The following image shows a more detailed HTTP request/response flow:

![image](https://user-images.githubusercontent.com/781074/59540964-b2dad000-8eff-11e9-89da-aa98e400bd88.png)

[Fullscreen](https://user-images.githubusercontent.com/781074/59540964-b2dad000-8eff-11e9-89da-aa98e400bd88.png)

<div style="page-break-after: always; visibility: hidden"> 
\pagebreak 
</div>

### Description

1. The user or the API client starts an HTTP request.
2. The [front controller](https://en.wikipedia.org/wiki/Front_controller) `public/index.php` handles all requests.
   Create a PSR-7 request instance from the server request.
3. Dispatch the request to the router.
4. The router uses the HTTP method, and the HTTP path to determine the appropriate action method.
5. The invoked controller action is responsible for:
    * Retrieving information from the request
    * Invoking the service and passing the parameters
    * Building the view data
    * Returning the response using a responder
6. The service is a use case handler and responsible for:
    * The business logic (validation, calculation, transaction handling, etc.)
    * Returning the result (optional)
7. The service can read ir write data to the database using a repository
8. The repository query handler creates a so called "use case optimal query" using a QueryBuilder
9. Execute the database query
10. Fetch the rows (result set) or the new primary key (ID) from the database
11. Map the row(s) to an object, or a list of data objects. Optional use a data mapper to create the objects.
12. The repository returns the result
13. Do more calculations with the fetched data. Do more complex operations. Optional, commit or rollback the transaction.
14. Return the result
15. Create the view data for the responder (optional)
16. Pass the view data to the responder
17. Let the responder render the view data into the specific representation like html or json and build the
    PSR-7 response with the ResponseFactory.
18. Return the response to the action method
19. Return the response to the router
20. Return the response to the front controller
21. The front controller emits the response using the SAPI Emitter
22. The emitter sends the HTTP headers and echos the HTTP body back to the client

## Action

Each **Single Action Controller** is represented by a dedicated class or closure.

The *Action* does only these things:

* collects input from the HTTP request (if needed)
* invokes the **Domain** with those inputs (if required) and retains the result
* builds an HTTP response (typically with the Domain invocation results).

All other logic, including all forms of input validation, error handling, and so on,
are therefore pushed out of the Action and into the **Domain**
(for domain logic concerns), or the response renderer (for presentation logic concerns).

**Note:** [Closures](https://www.php.net/manual/en/class.closure.php) (functions) as routing
handlers are quite "expensive", because PHP has to create all closures on each request.
The use of class names is more lightweight, faster and scales better for larger applications.

### Action example class

Returning a JSON response:

```php
<?php

namespace App\Action;

use App\Responder\Responder;
use Psr\Http\Message\ResponseInterface;
use Slim\Http\Response;
use Slim\Http\ServerRequest;

final class HomeAction
{
    /**
     * @var Responder
     */
    private $responder;
    
    public function __construct(Responder $responder)
    {
        $this->responder = $responder;
    }
    
    public function __invoke(ServerRequest $request, Response $response): ResponseInterface
    {
        return $this->responder->json(['success' => true]);
    }
}
```

Most people may think that this pattern is not suitable because it results in too many files. 
That this will result in more files is true, however these files are very small and focus on 
exactly one specific task. You get very specific classes with only one clearly defined responsibility 
(see SRP in SOLID). You should not worry too much about too many files, 
instead you should try to avoid too large files (fat controllers) 
with too many responsibilities where possible.

## Responder

According to [ADR](https://en.wikipedia.org/wiki/Action%E2%80%93domain%E2%80%93responder) there should be a **responder** for each action.
In most cases a generic responder (see [Responder.php](https://github.com/odan/slim4-skeleton/blob/master/src/Responder/Responder.php))
is good enough. Of course, you can add special responder classes and move the complete presentation logic there.
An extra responder class would make sense when [building an transformation layer](https://fractal.thephpleague.com/)
for complex (json or xml) data output. This helps to separate the presentation logic from the domain logic.

In the case of the tutorial application, the presentation work is so straightforward as
to not require a separate Responder for each action. A relaxed variation of a Responder
layer is perfectly suitable in this simple case, one where each Action uses a
different method on a common Responder.

Extracting the presentation work to a separate Responder,
so that response-building is completely removed from the Action, looks like this:

```php
<?php

namespace App\Responder;

use Psr\Http\Message\ResponseInterface;

final class Responder
{
    public function json(
        ResponseInterface $response,
        array $data = null
    ): ResponseInterface {
        $response = $response->withHeader('Content-Type', 'application/json');
        $response->getBody()->write((string)json_encode($data, JSON_THROW_ON_ERROR));

        return $response;
    }
    
    public function redirect(
        ResponseInterface $response,
        string $destination,
        array $queryParams = []
    ): ResponseInterface {
        if ($queryParams) {
            $destination = sprintf('%s?%s', $destination, http_build_query($queryParams));
        }

        return $response->withStatus(302)->withHeader('Location', $destination);
    }
    
    // ...
}
```

Now we can test the response-building work separately from the domain work.

Some notes: Putting all the response-building in a single class with multiple methods, 
especially for simple cases like this tutorial, is fine to start with. 
For ADR, is not strictly necessary to have one Responder for each Action.
What is necessary is to extract the response-building concerns out of the Action.

But as the presentation logic complexity increases (content-type negotiation? status headers? etc.), 
and as dependencies become different for each kind of response being built, you will want 
to have a Responder for each Action.

Alternatively, you might stick with a single Responder, but reduce its interface to a single method.

## Domain

The Domain layer is the core of the application.

The ADR pattern does not specify how to organize the Domain layer of the application.
You are completely free to choose an architecture style what makes sense to you and your 
specific requirements.

Nevertheless, I would like to take the opportunity to introduce a 
style that has worked very well for me in practice. 

This concept combines the best of a layered architecture and Domain Driven Design (DDD)
without the overwhelming complexity.

### Services

The word "service" is everywhere. Using the same word everywhere
for different things makes it hard to differentiate between them.

There are 3 different types of services:

* **Domain Services**: Encapsulates business logic that doesn't naturally fit within a domain object, 
  and are NOT typical CRUD operations – those would belong to a Repository.
* **Application Services**: Used by external consumers to talk to your system (think Web Services). 
  If consumers need access to CRUD operations, they would be exposed here.
* **Infrastructure Services**: Used to abstract technical concerns (e.g. Twig, Message Queuing, E-Mail provider, etc).

Keeping Domain Services along with Domain Objects is a good practice - 
they are all focused on domain logic.

Application Services will typically use both Domain Services and Repositories 
to deal with external requests.

### Application- and Domain Services

Here is the right place for complex **business logic** e.g. calculation, validation, transaction handling, file creation etc.
Business logic is a step up on complexity over CRUD (Create, Read, Update and Delete) operations.

An Application service can be invoked directly from an action handler, a service, the console (CLI) and 
of course from a test suite.

How to differentiate between **Application Services** and **Domain Services**?

Having that trouble myself, I have started using the term "Use Case" instead of "Application Service".
Further, I do only one thing per Use Case, instead of doing several things in an Application Service.

The question is to know exactly what goes in the Application Service
and non-application service. Where does user authorization go?
Where does validation go?
Are those application or non-application services?

This is where the idea of a Domain Service (cf. the DDD books by Vernon and Evans) comes into play.
They say those non-application services are called Domain Services.

> *If a non-application service is injected into other non-application services,
> should all this be moved out of the Application Service? If so, the Application Service
> don't really then contain anything as far as I can see.*

Whereas an Application Service presents a face to the world outside the Domain,
the Domain Service is internal to the Domain. As such, it's totally fine to have that
non-application Domain Service get injected into different Application Services
(and for Domain Services to be used by other Domain Services).

That also means eventually the Application Service may then contain almost nothing at all except
calls to coordinate different Domain Services. I think this is part of a natural progression:

* You used to put everything in a Controller method or Action class;
  then you extracted the domain logic into an Application Service.

* Next, different Application Services needed to do the same things over and over;
  you extracted the shared logic to different Domain Service classes.

* Finally, you realized that your Application Service could just use one or more
  Domain Service objects in a lot of cases.

Great, all non-trivial logic is down in the Domain now. The Application Service coordinates
between one or more of those Domain Services, then presents a Domain Payload back to the user interface.

### Infrastructure Service

An Infrastructure Services can be:

* Framework-specific code
* Implementations for boundary objects, e.g. the repository classes (communication with the database)
* Web controllers (actions), CLI, etc.

This means that from an ADR perspective an Infrastructure Service
does not fit conceptually into the ADR pattern because ADR
is a user interface pattern for server-side HTTP specific request/response operations.

The infrastructure is therefore like a layer around the core application.

![Onion Architecture](https://user-images.githubusercontent.com/781074/103920657-d84ab280-5111-11eb-8d02-83e352322897.png){ width=350px }

By separating domain from infrastructure code you automatically **increase testability**
because you can replace an **adapter** without affecting the **ports**. 
You can **postpone** the choice for database vendor, framework, query builder, mailer etc.
You can more easily **keep up** with the change rate of the framework-specific code or 
replace the framework altogether.

In the best case, externalize the infrastructure with proper interfaces
to ensure loose coupling between the application and the database.
It decomposes further the application core into several concentric rings using inversion of control.

The DI container is the perfect tool to separate the core application from infrastructure code.

### Services and Data

When people learn about OOP and object-oriented design they are often taught that objects are data with behavior.
There is the concept called "encapsulation" that says you should not expose the internals of your objects,
what you should do is you should have your objects expose enough operations so that the user of
the object can just ask it to do things you know. Tell don't ask if you will so.
This is the whole design idea about encapsulation and object-orientation.
If people are new to enterprise development they're just out of college or whatever you know learned object-oriented
design they've probably come out and they'll do design like this:

```php
class User
{
    public int $id;
    
    public string $username;
    
    public function createUser(string $username) {}
    
    public function read(int $id) {}
    
    public function update() {}
    
    public function delete(int $id) {}
}
```

They say I have users in my system and the user has an ID and a username.
The also want talk to their database to create, read, update and delete users.
Well, this is a well describes design-pattern, it's called "**Active-Record**".
But this is probably more of an **anti-pattern** because in the last decade we learned
that this is actually not a good way to design systems, even though this seems very object-oriented.
A user is an encapsulation of data about the user together with all the behavior that belongs to the user.
This seems like object-orientation. By principle there is nothing wrong with this, but the
problem is that you say, well I need to do things with users and later on you figure out I need
to do other things with users as well because I want to be able to send an email to an user.
So you would also add that feature to the user class:

```php
class User
{
    public int $id;
    
    public string $username;
    
    public function createUser(string $username) {}
    
    public function read(int $id) {}
    
    public function update() {}
    
    public function delete(int $id) {}
    
    public function sendEmail(Mail $message) {}
}
```

What often happens with this is that the more behavior you add to a class like this,
the more behavior it also attracts, so you end up with a couple of very, very big classes
and the class becomes less and less cohesive and we end up calling those big classes "**god classes**".
That is a well-known **anti-pattern**. So we learned the hard way that this leads to something
that is not very maintainable. So what should we do about it?

The (blue) DDD book from Eric Evans talks about various design pattern and how to think about objects.
Eric Evans makes a very good and interesting distinction in this book that I think is very valuable
that have served me very well doing object-oriented design in the last ten years.
Basically you can think about your objects as falling into these categories: Entities, Value Objects and Services.
Services are for behavior (business logic), while Value Objects and Entities contain only pure data.

I put the logic into Services and the plain data into tiny and domain specific objects (e.g. DTO's)
to **separate** data from behavior. This allows me to build and maintain non-trivial applications for many years. 

**Service example**

```php
$sourceAccount = new Account(100);
$destinationAccount = new Account(0);

$service = new AccountService();
$service->transfer(100, $sourceAccount, $destinationAccount);
```

### Naming convention

An indispensable part of every programming project is how you structure it, 
which involves organizing files and sources into directories, naming conventions, and similar. 
As your application grows, so does the need for structuring it in way that it is 
easy to manage and maintain.
In most cases, structure of an average PHP-based application is dictated or 
influenced by the framework that is being used, which is something I'm opposed to.
I would like to suggest a naming convention, and directory structure regarding application services
that works really well for me for small and very big enterprise applications at the same time. 

<div style="page-break-after: always; visibility: hidden"> 
\pagebreak 
</div>

### Directory structure

```
bin/              # command-line executables
config/           # configuration files
public/           # web server files, assets
resources/        # other resource files
src/              # PHP source code
templates/        # view and layout files
tests/            # test code
```

This is pretty much standard for applications, but also for libraries, whereas in their case, 
it is unlikely to find `public/` or `templates/` directories.

There is no `app/` directory, and there shouldn't be. 
Everything in addition to `src/` introduces unnecessary level of complexity.
I consider this directory structure a framework-agnostic, and based on it I've 
successfully managed to build different projects that were using different frameworks.
Now that we've busted all the myths about where PHP sources should be located,
let's consider options for organizing them inside `src/` directory,
because that's where the juice is.

**Separate domain and general-purpose code**

There's a great chance that besides domain-specific code, 
there will be need for creating some general, reusable, 
framework-like code inside `src/` directory, for example custom 
log handler (though it's most likely that Monolog already has it), 
cache facilitation, filters, and similar. 
That stuff is not your domain logic, it's not specific for your 
application and it's probably reusable between projects. 
What's more, it isn't supposed to be part of your application, 
but rather in a standalone library, and in a separate repository.

The domain code itself is isolated in a separate namespace `\App\Domain`.

```
src/
    Action/
    Console/
    Domain/
    Factory/
    Middleware/
    Responder/
    Utility/
```

**Grouping the Domain by Sub-Domain (Feature)**

The idea here is that when you are looking for the code that makes a feature, 
it is located in one place. This approach results in an intuitive directory 
structure that speaks for itself, whereas code is organized based on functional areas. 
Only by looking inside `src/Domain` directory, developer can immediately gain insight 
into what the application does

```
src/
    Domain/
        Subdomain1/
        Subdomain2/
        Subdomain3/
```

<div style="page-break-after: always; visibility: hidden"> 
\pagebreak 
</div>

**Grouping the Sub-Domain by Archetype**

When working on a certain feature I try to prevent switching between to many multiple directories.
For this reason I group the sub-domain (feature, sub-module) by Archetype like Service and Repository.
Theoretically you could also put the Action classes in a `Action` sub-directory here 
that belongs to this specific bounded context.

```
src/
    Domain/
        User/
            Data/             # DTO's
            Repository/       # Repositories
            Service/          # Application- and Domain Services
            Type/             # Enums
```

<div style="page-break-after: always; visibility: hidden"> 
\pagebreak 
</div>

## Best practices

Try to follow the SOLID, KISS, YAGNI and DRY principles whenever possible. Simplicity is the key.

Think of the [SRP](http://pragmaticcraftsman.com/2006/07/single-responsibility-principle/)
and give a service a "single responsibility". What changes for the same reason should be grouped together.

Please don't prefix all service classes with `*Service`.
A service class is not a "Manager" or "Utility" class. A service is a "Do-er".

A service class can have several methods as long as they serve a narrow purpose.
This also encourages you to name your classes more specifically. Instead of a "User" god-class,
you might have a `UserCreator` class with a few methods focusing on creating a user.

**Q:** Why would I change my UserCreator class?

**A:** Because I'm changing how I create a user and not because 
I'm changing how I assign a user to a task. Because that's being handled by the UserTaskAssignor class.

**Q:** Can I use a repository in multiple services OR 
can I use services in other services OR
can I use multiple services in an action class?

**A:** I think there is no real "hard rule" for this as long you try to follow this simple "best practice rules":

* A Single Action Controller should use only one Service and one Responder.
* A Single Action Controller contains only one public method, e.g. `__invoke`.
* A Service can use Repositories, but a Repository must never use a Service.
* A Service can use other Services.
* A Repository can be used by multiple Services, but the Repository should be in the same sub-domain (bounded context, module).
* A Service should contain only one public method. More than 3 public methods is a warn-signal for violating the SRP.
* A Service class name should not contain *Service as prefix, because it's not a "Manager" or God-class.
* A Service class should be responsible for only one thing and not more. See SRP.

<div style="page-break-after: always; visibility: hidden"> 
\pagebreak 
</div>

**What is the key to excellent software that satisfies users?**

*By Mario Cervera*

For me, it is not a programming language or a framework.
It is a deep understanding of the domain and how the system must work in the context of the users.
Engineering disciplines can help us.

* **Continuous Delivery.** When software is always in a releasable state, we can deliver it to users frequently to gain new knowledge.
* **Domain-Driven Design.** DDD allows us to build a domain model that is shared by the team, the business, and even the source code.
* **Clean Code.** The source code contains knowledge about the system and the domain. Clean code reflects this knowledge to us every time we read it.
* **Refactoring.** Refactoring allows us to keep knowledge up to date in the code.
* **Pair/Mob Programming.** Pair/mob programming spread knowledge among team members, contributing to build a shared understanding.
* **Test-Driven Development.** TDD makes us specify our current knowledge even before we write the code of the system.
* **Manage Priorities.** Decide about priorities you need knowledge about how important something is to your users. 
  If you haven't any knowledge, build something that generates data, insights, etc. *(by Simon Schubert)*

### Repositories

A [repository](https://designpatternsphp.readthedocs.io/en/latest/More/Repository/README.html)
abstracts the database access to the real database.
A repository improves code maintainability, testing and readability by separating **business logic**
from **data access logic** and provides centrally managed and consistent access rules for a data source.

There are two types of repositories: collection-oriented and persistence-oriented repositories.
In this case, we are talking about **persistence-oriented repositories**, since these are better
suited for processing large amounts of data.

Each public repository method represents a query. 
The return value represents the "result set" of a query and can be a primitive type, object or list (array) of them. 
Database transactions must be handled on a higher level (service) and not within a repository.

**Quick summary:**

* Communication with the database.
* Place for the data access (query) logic.
* Uses data mapper to create domain objects
* This is no place for the business logic. Use [services](#services) for the business logic.

<div style="page-break-after: always; visibility: hidden"> 
\pagebreak 
</div>

### Data Transfer Objects (DTO)

A DTO contains only pure **data**. There is no business or domain specific logic.
There is also no database access within a DTO.
A service fetches data from a repository and  the repository (or the service)
fills the DTO with data. A DTO can be used to transfer data inside or outside the domain.

**Example:**

```php
<?php

namespace App\Domain\Customer\Data;

use DateTimeImmutable;

final class CustomerData
{
    public string $name = '';
    
    public string $email = '';
    
    public DateTimeImmutable $dateOfBirth = null;
}
```

**Note:** Typed class properties have been added in PHP 7.4. [Read more](https://stitcher.io/blog/typed-properties-in-php-74)

### Types and enums

You should not use fixed strings and integer codes as values. Use class constants instead.

**Example:**

```php
<?php

final class LevelType
{
    public const LOW = 1;
    public const MEDIUM = 2;
    public const HIGH = 3;
}
```

<div style="page-break-after: always; visibility: hidden"> 
\pagebreak 
</div>

## Read more

This architecture was inspired by the following books and resources:

* [The Clean Architecture](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)
* [The Onion Architecture](https://jeffreypalermo.com/2008/07/the-onion-architecture-part-1/)
* [Action Domain Responder](https://github.com/pmjones/adr)
* [Domain-Driven Design](https://amzn.to/3cNq2jV) (The blue book)
* [Implementing Domain-Driven Design](https://amzn.to/2zrGrMm) (The red book)
* [Hexagonal Architecture](https://fideloper.com/hexagonal-architecture)
* [Alistair in the Hexagone](https://www.youtube.com/watch?v=th4AgBcrEHA)
* [Hexagonal Architecture demystified](https://madewithlove.be/hexagonal-architecture-demystified/)
* [Functional architecture](https://www.youtube.com/watch?v=US8QG9I1XW0&t=33m14s) (Video)
* [Object Design Style Guide](https://www.manning.com/books/object-design-style-guide?a_aid=object-design&a_bid=4e089b42)
* [Advanced Web Application Architecture](https://leanpub.com/web-application-architecture/) (Book)
* [Advanced Web Application Architecture](https://www.slideshare.net/matthiasnoback/advanced-web-application-architecture-full-stack-europe-2019) (Slides)
* [The Beauty of Single Action Controllers](https://driesvints.com/blog/the-beauty-of-single-action-controllers)
* [On structuring PHP projects](https://www.nikolaposa.in.rs/blog/2017/01/16/on-structuring-php-projects/)
* [Standard PHP package skeleton](https://github.com/php-pds/skeleton)
* [Services vs Objects](https://dontpaniclabs.com/blog/post/2017/10/12/services-vs-objects)
