---
title: Builder pattern with validation
layout: post
comments: true
published: false
description: 
keywords: 
---

Having lots of parameters in a constructor is some kind of an anti-pattern. But where is the limit? Maybe 6 or 5 or even 3? Especially a controller can have lots of parameters like Request, Response, Database (Model, Repository), View, Session (User and security context) and a Logger of course. Ups, suddenly we have 6 parameters and that looks bad. Instead of using telescoping constructor pattern, use builder pattern in combination with a simple validation trick. The build() method checks all required parameters before it creates a new object. All required parameters has to be set by a fluent interface. This makes is possible to create a new instance in a valid state without injecting a container.

## Usage

```php
<?php

$builder = new UserControllerBuilder();

// set required values
$builder->setRequest(new Request())->setResponse(new Response());

// set optional values
$builder->setDb(new Db())->setView(new View());

// build new object
$newController = $builder->build();

var_dump($newController); // object(UserController)
```

## Example

```php
<?php

interface BuilderInterface
{
    public function build();
}

abstract class Builder implements BuilderInterface {

    protected $properties = [];

    /**
     * Inject dependencies in properties.
     * 
     * @param mixed $object
     * @return mixed
     */
    protected function inject($object) {
        $class = get_class($object);
        foreach($this->properties as $name => $value) {
            $reflection = new ReflectionProperty($class, $name);
            $reflection->setAccessible(true);
            $reflection->setValue($object, $value);
        }
        return $object;
    }
}

class Request
{
}

class Response {

}

class Db {
    public function fetchAll($sql) {
        return [];
    }
}

class View {
    public function render($file, $data = []) {
        return 'html';
    }
}

class UserController {
    protected $request;
    protected $response;
    protected $db;
    protected $view;
}

class UserControllerBuilder extends Builder
{
    public function setRequest(Request $request) {
        $this->properties['request'] = $request;
        return $this;
    }

    public function setResponse(Response $response) {
        $this->properties['response'] = $response;
        return $this;
    }

    public function setView(View $view) {
        $this->properties['view'] = $view;
        return $this;
    }

    public function setDb(Db $db) {
        $this->properties['db'] = $db;
        return $this;
    }

    /**
     * Build.
     *
     * @return UserController
     * @throws Exception
     */
    public function build() {
        // Validation
        if(!isset($this->properties['request'])) {
            throw new Exception('Request is required');
        }
        if(!isset($this->properties['response'])) {
            throw new Exception('Response is required');
        }

        // Build the object
        $controller = new UserController();
        return $this->inject($controller);
    }
}
```

## Keywords

* PHP
* Dependency Injection
* Properties injection
* Container
* DI, DIC