---
title: API Design - Decoupling from ORM Dependencies
layout: post
comments: true
published: true
description:
keywords: 
image: 
---

In the world of software development, APIs are increasingly important for modern applications. APIs connect systems, enable data exchange, and provide smooth automation. 
In this blog post, I will address the critical topic of API design and why it's essential to avoid binding your API to an ORM (Object-Relational Mapping) entity model. 
Let's uncover the pitfalls and discover the path to more flexible, maintainable, and powerful APIs.

## What is an ORM?

![ORM](https://github.com/odan/odan.github.io/assets/781074/8c547092-42c0-4f5a-a3f7-a975c7e75012)

Object-Relational Mapping is a technique that allows developers to interact with a relational database through object-oriented programming. It tries to bridge the gap between the world of object-oriented code and the relational databases, allowing you to work with database records as if they were objects in your code. This simplifies interaction with the database and reduces the need for raw SQL queries.

## ORM to API Mapping

```php
namespace App\Entity;

use ApiPlatform\Metadata\ApiResource;
use Doctrine\ORM\Mapping as ORM;

#[ORM\Entity]
#[ApiResource]
class Product
{
    #[ORM\Id, ORM\Column, ORM\GeneratedValue]
    private ?int $id = null;

    #[ORM\Column] 
    #[Assert\NotBlank]
    public string $name = '';
}
```

For example, Doctrine and API Platform can automatically expose entities mapped as "API resources" through a REST API supporting CRUD operations. You can use attributes and YAML configuration files to expose your entities.

## CRUD-based API

![CRUD-based API](https://github.com/odan/odan.github.io/assets/781074/9de34d83-f88f-4dd7-a15b-d4eda9a5817e)

Binding your API tightly to ORM entity models often results in a CRUD-based API, where much of the logic is pushed onto the client-side. ORM entity models tend to closely mirror the database schema, focusing on data storage and retrieval. When your API is tightly bound to these models, it becomes primarily concerned with CRUD (Create, Read, Update, Delete) operations, reflecting the basic database operations.
## Lack of Separation of Concerns

![Lack of Separation of Concerns](https://github.com/odan/odan.github.io/assets/781074/93c59f99-6b4c-483c-8fb2-972ea6f24c01)

When you bind your API tightly to ORM entity models, you mix database logic with API logic, which can make your code harder to maintain.

## Increased Client Complexity

![Increased Client Complexity](https://github.com/odan/odan.github.io/assets/781074/e0e800ca-ed83-4dc4-af1b-9534fd317158)

APIs should not only provide data but also encapsulate business logic and application-specific rules. However, when your API is tightly coupled to ORM entity models, the logic for data manipulation often gets pushed to the client-side. Clients must perform complex operations by orchestrating multiple CRUD requests, leading to an anemic API with minimal business logic. This puts a burden on the client-side code, making it more complex and less maintainable.

## Inefficiency

![Inefficiency](https://github.com/odan/odan.github.io/assets/781074/be44c9f4-ad21-404b-a1b9-78daccff7930)

CRUD-based APIs may result in inefficient data transfer between the client and the server. Clients often need to make multiple requests to the API to perform a single logical operation, leading to increased network overhead and slower response times.

## Performance Issues

### N+1 problem

```php
use Doctrine\ORM\Mapping as ORM;

/**
 * @ORM\Entity
 * @ORM\Table(name="posts")
 */
class Post {
    /**
     * @ORM\ManyToOne(targetEntity="User", inversedBy="posts")
     * @ORM\JoinColumn(name="user_id", referencedColumnName="id")
     */
    private $user;

    //...
}
```

Inefficient ORMs can lead to performance issues that are difficult to optimize. For example, the N+1 problem is a common problem when using an ORM.

```php
$users = $entityManager->getRepository(User::class)->findAll();

foreach ($users as $user) {
    // N queries, one for each user
    $posts = $user->getPosts(); 

    foreach ($posts as $post) {
        // Process each post
    }
}
```

It can happen when an ORM retrieves a collection of entities and then makes additional queries for each entity to retrieve associated data. In this code, for each user, a separate query is executed to fetch their associated posts. This results in a large number of database queries and less performance.

## Fixing the N+1 Problem

![Fixing the N+1 Problem](https://github.com/odan/odan.github.io/assets/781074/2049582a-f9fa-41b0-901e-90a89e9d6297)


To fix the N+1 problem, developers can use eager loading to retrieve data in advance. With eager loading, the associated data is retrieved along with the parent object. This is more efficient when loading data, but the data is loaded whether it's used or not. It's essential to set up eager loading properly to avoid performance issues.

## Limited Reusability and Flexibility

![Limited Reusability and Flexibility](https://github.com/odan/odan.github.io/assets/781074/6eaab890-87a7-472d-ae06-2f7b1df4bd7d)

A CRUD-based API is less reusable because it exposes the underlying database structure directly. Changes in the database schema can significantly impact your API, leading to challenges when adapting to evolving requirements. This tight coupling makes it challenging to modify or extend the API without affecting client applications.

## Complex Queries

![Complex Queries](https://github.com/odan/odan.github.io/assets/781074/2e6e728c-419f-4227-b87f-d4b0b73c1fb4)

Handling complex database queries can become cumbersome when tightly bound to ORM models.

## Vendor Lock-in

![Vendor Lock-in](https://github.com/odan/odan.github.io/assets/781074/3bd811ea-b25a-4eac-8779-6dbc95502aee)

While ORM and proprietary platforms can be helpful at the beginning, they can eventually restrict your database options and create heavy reliance on third-party providers in the long term.

## Advantages of Separation

To overcome these limitations and create more robust and maintainable APIs, it's essential to abstract the API layer from the database layer and introduce a service layer that encapsulates business logic. This separation allows you to design APIs that provide meaningful, high-level operations, reducing the client's burden of handling low-level CRUD operations.

## Improved Modularity

Separation promotes cleaner, more maintainable code and better performance by minimizing unnecessary data transfers between the client and server. When different teams or developers work on different modules, they can optimize their components at the same time.

## Performance Optimization

Use-case specific query optimization can lead to lower overhead in database communication and data transformation, resulting in better performance.

## Caching Strategies

Separation helps in identifying which parts of the system are frequently accessed or require real-time data and which can be cached. Caching can significantly improve performance by reducing the load on critical components. The choice of caching strategy depends on your application's specific use cases and performance requirements.

## Multiple Data Sources

If needed, you may integrate other data sources, 
such as NoSQL databases or external APIs, when you have a separation between your API and database layer.

## How to Implement Separation

Implementing more separation involves organizing your software into different layers, such as:
- Application Layer: Controller classes to handle HTTP requests and responses.
- Domain Layer: Service classes for business logic.
- Persistence Layer: Repository classes for interacting with a database.

## Repository Pattern

```php
final class UserOtpSenderRepository
{
    public function __construct(private PDO $pdo)
    {
    }

    public function findUserByEmail(string $email): ?User
    {
        $sql = 'SELECT id,role FROM users WHERE email = :email';

        $statement = $this->pdo->prepare($sql);
        $statement->bindValue(':email', $email);
        $statement->execute();

        $row = $statement->fetch(PDO::FETCH_ASSOC);

        return $row ? new User($row): null;
    }
}
```

Repositories are responsible for handling data access operations, including reading, inserting, updating, and deleting data in the database. They provide a clean, domain-specific interface for accessing and managing data. This approach provides a clean separation between your application's business logic and data storage.

## Service Class

```php
final class UserOtpSender
{
    public function __construct(private UserOtpSenderRepository $repository)
    {
    }

    public function sendLoginOtp(string $email): void
    {
        // Validate input
        // ...

        // Find user
        $user = $this->repository->findUserByEmail($email);

        if ($user) {
            // Generate OTP, send email
            // ...
        }
    }
}
```

A Service class contains the application's business logic and orchestrates the execution of use-cases. It often relies on Repositories to interact with the database. This separation ensures that application rules and operations are maintained independently from the API or persistence layer.

## Action

```php
final class UserOtpAction
{
    public function __construct(private UserOtpSender $userOtpSender)
    {
    }

    public function __invoke(Request $request, Response $response): Response
    {
        // Get parameters from request
        $data = $request->getParsedBody();
        $email = $data['email'] ?? '';

        // Invoke use-case specific service
        $this->userOtpSender->sendLoginOtp($email);

        // Return response
        return $response->withStatus(201);
    }
}
```

A controller, like a single-action-controller, handles the HTTP request and response, and invokes a service method designed for this specific use case. This helps keep the code base well-organized and focused on a single task.

Depending on your framework, you may also need to define a route that maps the HTTP request to the corresponding Controller action.

## Conclusion

In conclusion, binding your API to an ORM entity model can lead to various pitfalls, while separating them brings a lot of benefits. I encourage you to adopt a decoupled approach for better API and software design.

## Read more

* [Video: API Design - Decoupling from ORM Dependencies](https://www.youtube.com/watch?v=2V79evfXlrA)
