---
title: Bringing "Actions" to your Slim application.
layout: post
comments: true
published: false
description: 
keywords: 
---

Did you know that the "classic" controller class with its different methods violates the SRP principle? 
The reason is that your controller no longer only cares about one action, 
but usually several actions. As a result, you need to inject dependencies over the constructor 
that are not always needed. This example shows an example of the problem:

```php
<?php

namespace App\Controller;

use App\Service\Product\ProductEditService;
use App\Service\Product\ProductIndexService;
use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface;

class ProductController extends AbstractController
{
    private $productIndexService;
    private $productEditService;

    public function __construct(
        ProductIndexService $productIndexService, 
        ProductEditService $productEditService)
    {
        $this->productIndexService = $productIndexService;
        $this->productEditService = $productEditService;
    }

    public function indexAction(ServerRequestInterface $request, ResponseInterface $response): ResponseInterface
    {
        $viewData = $this->productIndexService->calculateThings();

        return $this->render($response, 'Product/product-index.twig', $viewData);
    }

    public function editAction(ServerRequestInterface $request, ResponseInterface $response): ResponseInterface
    {
        $viewData = $this->productEditService->calculateThings();

        return $this->render($response, 'Product/product-edit.twig', $viewData);
    }
}
```

The above example shows that you have to pass (and resolve) two service classes, 
although only one action method will be called.

The aim of good architecture should be to achieve low cohesion. 

In order to reduce the coupling to a minimum, we therefore apply a simple principle.

The controller is not a class, but a layer. Within the controller layer, 
action classes should represent the specific actions. 

In other words, each action class has only one public method for handling the request. 
In this way, we ensure that each action class only declares the dependencies that are 
really needed, thus reducing the coupling.

I use the following naming convention for action classes: `<Page><Action>Action`

For example: 
  
  * Route: `GET /`, Class: HomeIndexAction
  * Route: `GET /about`, Class: AboutIndexAction
  * Route: `GET /users/123`, Class: UserEditAction
  * Route: `POST /contact`, Class: ContactSubmitAction

This is the content of the new file: ProductIndexAction.php for the route `/products`:

```php
<?php

namespace App\Action;

use App\Service\Product\ProductService;
use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface;

class ProductIndexAction extends AbstractAction
{
    private $productIndexService;

    public function __construct(ProductService $productIndexService)
    {
        $this->productIndexService = $productIndexService;
    }

    public function __invoke(
        ServerRequestInterface $request,
        ResponseInterface $response
    ): ResponseInterface {
        $viewData = $this->productIndexService->calculateThings();
        
        return $this->render($response, 'Product/product-index.twig', $viewData);
    }
}
```

This is the content of the new file: ProductEditAction.php

```php
<?php

namespace App\Action;

use App\Service\Product\ProductEditService;
use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface;

class ProductEditAction extends AbstractAction
{
    private $productEditService;

    public function __construct(ProductEditService $productEditService)
    {
        $this->productEditService = $productEditService;
    }

    public function __invoke(
        ServerRequestInterface $request,
        ResponseInterface $response
    ): ResponseInterface {
        $viewData = $this->productEditService->calculateThings();
        
        return $this->render($response, 'Product/product-edit.twig', $viewData);
    }
}
```

By reducing it to a single method per class, it is possible to reduce the dependencies 
and responsibilities of the class to a minimum. 

Next, I'll point out another advantage: Refactoring!

In the Slim Framework, you can use the syntax of class-based handlers: "class:method" 
can be defined. Here is an example route for the start page:

```php
$app->get('/product', 'App\Controller\ProductController:indexAction');
$app->get('/product/{id}', 'App\Controller\ProductController:editAction');
```

The problem is that the string `App\Controller\ProductController:indexAction` is just 
a string for your IDE and not a resolvable class. Therefore, when renaming classes, 
these strings must also be adjusted manually. Even the static code analysis tools 
can't protect you from your own mistakes.

Now, a nice example of the new action class:

```php
$app->get('/product', \App\Action\ProductIndexAction::class);
$app->get('/product/{id}', \App\Action\ProductEditAction::class);
```

Immediately your IDE knows what class you mean by that. In return for your efforts, 
you will benefit from the Refactor functionality. Also, the static code analysis system 
(e.g. phpstan) can now find bugs before they are executed. 

Integration test writing is simplified and coupling is reduced to a minimum. 
The SRP principle will be happy :-)


Keywords: php, slim framework, mvc, adr, controller