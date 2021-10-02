---
title: Slim 4 - API documentation with Swagger
layout: post
comments: true
published: true
description: 
keywords: slim, php, swagger, openapi, api, docs, documentation
---

## Table of contents

* [Requirements](#requirements)
* [Introduction](#introduction)
* [Installation](#installation)
* [Template](#template)
* [Route](#Route)

## Requirements

* PHP 7.2+
* Composer (dev environment)
* [A Slim 4 application](https://odan.github.io/2019/11/05/slim4-tutorial.html)
* [Twig for Slim](https://odan.github.io/2020/04/17/slim4-twig-templates.html)

## Introduction

This article explains how to expose swagger-ui inside your Slim project 
through a route (eg. /docs), without the need for node.

Just add a reference to your swagger Yaml or JSON specification, 
and enjoy swagger-ui in all it's glory.

*Preview:*

![screely-1591996310595](https://user-images.githubusercontent.com/781074/84546695-2d505280-ad02-11ea-9ce9-93c609db656f.png)

## Installation

To parse the OpenAPI Yaml file, we have to install a good Yaml parser:

```
composer require symfony/yaml
```

For demonstration purpose download this example file, or just use your own specification:

* [petstore.yaml](https://raw.githubusercontent.com/OAI/OpenAPI-Specification/master/examples/v3.0/petstore.yaml)

Save the yaml file in the `resources/docs/` directory. If not exists, create it.

## Template

Create a new template file: `templates/docs/swagger.twig` and copy/paste this content:

{% raw %}
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>API Specification - Swagger UI</title>
    <link rel="stylesheet" type="text/css"
          href="https://cdnjs.cloudflare.com/ajax/libs/swagger-ui/3.26.1/swagger-ui.css">
</head>
<body>
<div id="swagger-ui"></div>
<script src="https://cdnjs.cloudflare.com/ajax/libs/swagger-ui/3.26.1/swagger-ui-bundle.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/swagger-ui/3.26.1/swagger-ui-standalone-preset.js"></script>
<script>
    window.onload = function () {
        const ui = SwaggerUIBundle({
            spec: {{ spec|raw }},
            dom_id: '#swagger-ui',
            deepLinking: true,
            supportedSubmitMethods: [],
            presets: [
                SwaggerUIBundle.presets.apis,
            ],
            plugins: [
                SwaggerUIBundle.plugins.DownloadUrl
            ],
        })
        window.ui = ui
    }
</script>
</body>
</html>
```
{% endraw %}

## Route

Create a new action class: `src/Action/Docs/SwaggerUiAction.php`

```php
<?php

namespace App\Action\Docs;

use App\Responder\Responder;
use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface;
use Slim\Views\Twig;
use Symfony\Component\Yaml\Yaml;

final class SwaggerUiAction
{
    /**
     * @var Twig
     */
    private $twig;

    public function __construct(Twig $twig)
    {
        $this->twig = $twig;
    }

    /**
     * Action.
     *
     * @param ServerRequestInterface $request The request
     * @param ResponseInterface $response The response
     *
     * @return ResponseInterface The response
     */
    public function __invoke(
        ServerRequestInterface $request, 
        ResponseInterface $response
    ): ResponseInterface {
        // Path to the yaml file
        $yamlFile = __DIR__ . '/../../../resources/docs/petstore.yaml';

        $viewData = [
            'spec' =>json_encode(Yaml::parseFile($yamlFile)),
        ];

        return $this->twig->render($response, 'docs/swagger.twig', $viewData);
    }
}
```

Add a new route:

```php
$app->get('/docs/v1', \App\Action\Docs\SwaggerUiAction::class);
```

Then navigate to `http://localhost/docs/v1` and you should see 
a pretty Swagger UI documentation.
