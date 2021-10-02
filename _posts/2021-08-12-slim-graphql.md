---
title: Slim 4 - GraphQL
layout: post
comments: true
published: true
description:
keywords: php, slim, graphql
image: https://odan.github.io/assets/images/slim-logo-600x330.png
---

## Table of contents

* [Introduction](#introduction)
* [Requirements](#requirements)
* [Installation](#installation)
* [Hello World](#hello-world)  
* [Conclusion](#conclusion)
* [Read more](#read-more)

## Introduction

GraphQL is a modern way to build HTTP APIs consumed by the web and mobile clients. It is intended to be an alternative
to REST and SOAP APIs.

GraphQL itself is a specification designed by Facebook engineers. Various implementations of this specification were
written in different languages and environments.

The Graph API is the primary way to get data into and out of a platform. It's an HTTP-based API that apps can use to
programmatically query data, post new entries, upload things, and perform a wide variety of other tasks.

## Requirements

This documentation assumes your familiarity with GraphQL concepts. If it is not the case - first learn about
[GraphQL on the official website](https://graphql.org/learn/).

Other requirements:

* PHP 7.4+
* [A Slim 4 application](https://odan.github.io/2019/11/05/slim4-tutorial.html)

## Installation

For this Slim framework tutorial I will use the most popular
[graphql-php](https://webonyx.github.io/graphql-php/) component.
graphql-php is a feature-complete implementation of GraphQL specification in PHP.
This library is a thin layer around your existing data layer and business logic. It does not dictate how these layers
are implemented or which storage engines are used. Instead, it provides tools for creating rich APIs for your existing
application.

Use Composer to install the graphql-php component:

```
composer require webonyx/graphql-php
```

### Install Tools (optional)

For development and testing purposes you may install a GraphQL tool like
[Postman](https://learning.postman.com/docs/sending-requests/supported-api-frameworks/graphql/).

I know what you say now. Many people think of Postman as an advanced REST client. Beyond REST, Postman is a tool that
handles any calls sent over HTTP. This means that you can use Postman to interact with protocol-agnostic APIs - such as
SOAP and **GraphQL**, which can both utilize HTTP, just like REST.

Learn
more: [Postman - Querying with GraphQL](https://learning.postman.com/docs/sending-requests/supported-api-frameworks/graphql/)

## Hello World

GraphQL is composed of nodes, edges, and fields. Typically you use nodes to get data about a specific object, use edges
to get collections of objects on a single object, and use fields to get data about a single object or each object in a
collection.

Let's create a new GraphQL "endpoint" that will be capable to process the following simple query:

```graphql
query {
  echo(message: "Hello World")
}
```

In Slim you could create and endpoint that handles the "Hello World" query as follows:

Add this (empty) route that acts as our GraphQL endpoint.

```php
use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface;

// ...
$app->post('/hello', function (ServerRequestInterface $request, ResponseInterface $response) {
 // put your code here...
});
```

To handle the "echo" query we create an `GraphQL\Type\Definition\ObjectType` instance
and pass the configuration with the `echo` field, the return type,
the arguments and the resolver callback function.

```php
<?php
use GraphQL\Type\Definition\ObjectType;
use GraphQL\Type\Definition\Type;
use GraphQL\Type\Schema;

$queryType = new ObjectType(
    [
        'name' => 'Query',
        'fields' => [
            'echo' => [
                'type' => Type::string(),
                'args' => [
                    'message' => Type::nonNull(Type::string()),
                ],
                'resolve' => function ($rootValue, $args) {
                    return $rootValue['prefix'] . $args['message'];
                }
            ],
        ],
    ]
);

$schema = new Schema(['query' => $queryType]);
```

To make the following code work, you have to ensure that the
Slim BodyParsingMiddleware has been added to the middleware stack:

```php
// config/middleware.php
$app->addBodyParsingMiddleware();
```

Next, we have to fetch the parsed (json) body from the request
object to get the query and the optional parameter variables.

```php
$input = $request->getParsedBody();
$query = $input['query'];
$variables = $input['variables'] ?? null;
```

If you do not add the BodyParsingMiddleware to the middleware stack, 
you need to decode the request manually as described below:

```php
$input = (array)json_decode((string)$request->getBody(), true);
```

To execute the actual query we pass all we have to the static 
`GraphQL::executeQuery` method:

```php
use GraphQL\GraphQL;
// ...

try {
    $rootValue = ['prefix' => 'You said: '];
    $result = GraphQL::executeQuery($schema, $query, $rootValue, null, $variables);
    $output = $result->toArray();
} catch (Exception $exception) {
    $output = [
        'error' => [
            [
                'message' => $exception->getMessage()
            ]
        ]
    ];
}
```

The final step is to put this output into our PSR-7 response object:

```php
$response = $response->withHeader('Content-Type', 'application/json');
$response->getBody()->write(json_encode($output));

return $response;
```

Ok fine, let's open Postman and see if it works.

Change the HTTP method to `POST` and enter the url of your endpoint.

Change the schema to `GrapgQL` and paste this query into the "Query" textfield:

```graphql
query {
  echo(message: "Hello World")
}
```

Click the "Send" button to submit the request. 

The result should look like this:

![image](https://user-images.githubusercontent.com/781074/129356196-7f1ec6a5-de04-4971-a3fb-0b7558a8d6d6.png)

## Security

Please note that GraphQL is susceptible to the following attack vectors (among others):

* The single endpoint provides access to all information from the website, 
  so we could have private data unintentionally exposed.
* The queries can be very complex and may overwhelm the web and database servers.
* The same mutation can be executed multiple times in a single query, 
  and multiple queries can be executed together in a single request, 
  allowing attackers to attempt gaining access to the back-end by providing 
  many combinations of user/passwords.

These attacks can happen with GraphQL, and not with REST, 
because GraphQL is more powerful than REST. 
While in REST the query is defined in advance and stored in the server, 
in GraphQL it is provided on runtime by the client (unless using persisted queries).

## Conclusion

Of course, this article shows only a very short sample of what is possible.
Next, for instance, you could continue to generate your dynamic database queries, etc.
However, I only wanted to simplify the first step for you and give you the 
opportunity to develop it further.

**Opinion:** I don't think you need GraphQL as long as your REST API meets your requirements.
For security and privacy reasons, the most business logic and (database) queries
should be defined, created and executed by the server anyway (and not by the client).

Just to be sure: I'm not advocating to not use GraphQL, but please use GraphQL responsibly.
GraphQL is powerful, which means it is dangerous.
If you use GraphQL after all, you must be absolutely sure that you know what you are doing.

## Read more

* <https://webonyx.github.io/graphql-php/>
* [Why GraphQL should not be in WordPress core](https://graphql-api.com/blog/why-graphql-should-not-be-in-wordpress-core/)
* [Damn GraphQL - Defending and Attacking APIs - Dolev Farhi](https://www.youtube.com/watch?v=EVRf708-zq4)  
* [Ask a question or report an issue](https://github.com/odan/slim4-tutorial/issues)
 