---
title: ASP.NET - Single Action Controllers
layout: post
comments: true
published: true
description: 
keywords: aspnet, csharp, controller, MediatR
---

*Exploring Controller Strategies*

*Simplicity in ASP.NET Core: Single Action Controllers vs. MediatR*

When building modern web applications with ASP.NET Core, 
the simplicity and efficiency of your architecture can be a 
key determinant of both maintainability and performance. 
One of the popular libraries often employed to facilitate 
the implementation of the Mediator pattern is [MediatR](https://github.com/jbogard/MediatR). 

While MediatR is a robust and highly useful library for handling 
complex scenarios involving many interdependent services 
and handlers, it might not be necessary when using single action controllers. 

The concept of **[single action controllers](https://driesvints.com/blog/the-beauty-of-single-action-controllers/)** represents an approach 
to handling HTTP requests. Unlike traditional controllers that manage 
**multiple actions (endpoints)**, each single action controller focuses 
on **executing a singular operation per endpoint**. 

**Simplicity and Directness**

Unlike traditional controllers that handle **multiple actions (endpoints)**, 
each single action controller is handling **only a single endpoint** operation.
You don't need any additional package or any configuration 
to implement single action controllers, because this is just
an architectural choice and everything is already there.

Single action controllers are designed with simplicity in mind. 
Each controller handles one specific action, making the flow of 
data and control straightforward and easy to follow. 
MediatR introduces an **additional layer of abstraction** that can obscure this simplicity.

Without MediatR, a request flows directly from the controller to the service, 
and then to the database or other dependencies. This directness is 
beneficial for understanding and maintaining the code. 

Adding MediatR means introducing extra steps: the controller sends 
a request to MediatR, which then dispatches it to the appropriate handler. 
For simple single action controllers, this added complexity is often unnecessary.

By directly calling the service layer from the controller, 
you can achieve lower latency and improve the overall performance of your application.

This is one of my pain points using MediatR: Debugging issues.

When it comes to maintainability, having fewer moving parts generally 
makes life easier. Debugging issues in a simple, direct flow from 
the controller to the service is more straightforward than tracing 
through multiple layers of abstraction. With MediatR, you might 
need to jump through additional hoops to understand the path 
a request takes, especially if your handlers are complex 
or involve multiple steps.

Unit testing in a system without MediatR is simpler. 
Single action controllers allow you to replace dependencies with 
alternate implementations as needed, facilitating isolation testing 
without the need to accommodate a mediation layer. In contrast, 
using MediatR introduces an additional abstraction that requires 
careful consideration in test setups, potentially complicating the testing process.

## Example of a Single Action Controller

To illustrate the concept of a single action controller, 
let’s look at a simple example in an ASP.NET Core application. 

**A single action controller is a controller that contains only one action method.**

This approach can be particularly useful in microservices or 
when you want to adhere to the Single Responsibility Principle (SRP).

```cs
using Microsoft.AspNetCore.Mvc;

namespace MyApi.Controllers;

[ApiController]
[Route("/api/ping")]
public class PingGetController : ControllerBase
{
    [HttpGet]
    public IActionResult Invoke()
    {
        return Ok("Hello, World!");
    }
}
```

In this example, the `HomeController` is a single action controller 
that handles one endpoint: `GET /api/ping`. 
This controller has only one (public) action method, `Invoke`, 
which returns a simple message.

Note that the name of the method `Invoke` is just an example.
You may also use other generic name such as `Handle` if you prefer.

**Benefits of Single Action Controllers**

* Each controller has a single responsibility, 
making the codebase easier to understand and maintain.

* New developers can quickly grasp the purpose of each controller and endpoint.

* By limiting controllers to a single action, you enforce a clean separation of concerns. 
Each controller focuses on handling one specific request type.

* This separation aligns with the Single Responsibility Principle (SRP), 
which states that a class should have only one reason to change.

* Unit tests can be more straightforward because each controller has a singular focus.

* Debugging is simplified because the flow of data is direct and less convoluted.

* In large applications or microservices, managing one controller per endpoint 
can lead to a more modular architecture. Each module (controller) can be developed,
tested, and deployed independently.

**Naming Controller Classes**

For applications with many endpoints, this approach can lead to a large number of controllers. 
Proper organization and naming conventions are essential.

Single action controllers can be named based on the specific action they perform. 
The naming should be clear and descriptive to reflect their single responsibility. 
Here are some naming conventions and examples for single action controllers:

### Example: Path + HTTP-Method

This convention includes the route (url path), and the used HTTP method.

**Get User**

* Route: `GET api/users/{id}`
* Controller Name: `UserGetController`

**Create User**

* Route: `POST api/users`
* Controller Name: `UserPostController`

**Update User**

* Route: `PUT api/users/{id}`
* Controller Name: `UserPutController` 

**Delete User**

* Route: `DELETE api/users/{id}`
* Controller Name: `UserDeleteController` 

This approach ensures that each controller is dedicated to a specific action, making it easier to understand, test, and manage.

## A More Complex Scenario with Dependency Injection

In this example, we will add service classes that handle the business logic for each operation. Each controller will inject the appropriate service using dependency injection and invoke its methods to perform the required actions.

```cs
using Microsoft.AspNetCore.Mvc;

namespace MyApi.Controllers;

[ApiController]
[Route("/api/v1/users")]
public class UserGetController : ControllerBase
{
    private readonly UserReader _userReader;

    public UserGetController(UserReader userReader)
    {
        _userReader = getUserService;
    }

    [HttpGet("{id}")]
    public IActionResult Invoke(int id)
    {
        var user = _userReader.FindUser(id);
        if (user == null)
        {
            return NotFound();
        }

        return Ok(user);
    }
}
```

## Conclusion

MediatR is an excellent tool for complex applications where 
decoupling and managing dependencies between various parts of the system is necessary. 
However, the benefits of MediatR may not outweigh the costs. 
By keeping your architecture simple, direct, and free from unnecessary abstractions,
you can enhance the maintainability, performance, and clarity of your code.

For many scenarios single action controllers is a more straightforward
approach that also can be the most efficient and effective choice.

