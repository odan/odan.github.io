---
title: Initializing Vue.js data from the server side
layout: post
comments: true
published: false
description: 
keywords: vuejs, ssr, php, twig
---

Inspired by the blog post 
[Server-side apps with client-side rendering](https://reinink.ca/articles/server-side-apps-with-client-side-rendering) 
from `@reinink` I wrote this article.

With the help of server-side rendered initial data we get more speed, a better SEO score and of course 
more happy users. 
This demo doesn't make use of a special Data Store like Vuex, because I want to keep it lean and simple.

The idea is that we generate the initial data for Vue on the server, and then we put this 
data into the `Vue.prototype` property. When creating the Vue.js instance, 
we will fetch the initial data and merge it with the existing `data` property. 
This works so fast that you won't have to wait any longer or see any flickering. 

### Requirements

In this example I use the PHP Twig Template Engine from Symfony on the server side. 

* PHP
* Twig
* Vue.js

### Rendering the data on the server side

We start with the server side.

Normally, the data on the server is fetched directly from the database. 

For the sake of simplicity, I use a fixed array of data in this example.

Content of file: `HomeIndexAction.php`

```php
<?php

namespace App\Action;

use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface;

class HomeIndexAction extends BaseAction
{

    public function __invoke(
        ServerRequestInterface $request,
        ResponseInterface $response
    ): ResponseInterface {
        $user = [
            'id' => 1234,
            'username' => 'Max',
            'email' => 'max@example.com',
        ];

        $viewData = [
            'data' => [
                'user' => $user,
            ]
        ];

        return $this->render($response, 'Home/home-index.twig', $viewData);
    }
}
```

### Preparing the Twig template engine

For the output of base64 encoded data we have to add a new Twig filter 
for the `base64_encode` functionality. 

Please add this filter to the Twig environment: 

```php
$twig->addFilter(new \Twig_Filter('base64_encode', function ($string) {
    return base64_encode($string);
}));
```

### Preparing the templates

Let's let us add a `app` element into a Twig template:

Content of file: `home-index.twig`

{% raw %}
```twig
{% block content %}
    <div id="app"></div>
{% endblock %}
```
{% endraw %}

Then create a vue file with a `template` tag for the html template
and a `script` tag to create a new Vue instance. 

Content of file: home-index.vue

{% raw %}
```vue
<template id="user-template">
    <div id="content" class="container">
        <div class="row">
            <div class="col-md-12">
                User-ID: {{ user.id }}<br>
                Username: {{ user.username }}<br>
            </div>
        </div>
    </div>
</template>

<script>
    new Vue({
        el: "#app",
        template: '#user-template',
        data: {
            user: {},
        },
        mounted: function() {
             Object.assign(this, this.initData);
        }
    });
</script>
```
{% endraw %}

### Loading the data on the client side

Lets take a closer look at the `mounted` event:

```javascript
mounted: function() {
     Object.assign(this, this.initData);
}
```
The `mounted` even will be called after the instance has been mounted.
Then `Object.assign` will merge the values of `initData` to the Vue.js instance 
`data` property. 

Optionally, it would be possible to retrieve further data via Ajax here if required.

### Encoding the data to JSON

On the client side we have to fill the `Vue.prototype` with the initial data. 

To [protect the script against XSS](https://github.com/dotboris/vuejs-serverside-template-xss) 
we are encoding the data to JSON and then to base64.

On the client side we are decoding the base64 string to a json string, 
and then we parse the json string into an object.

{% raw %}
```twig
<div id="app"></div>
<script>
    Vue.prototype.initData = JSON.parse(atob('{{ data|json_encode|base64_encode }}'));
</script>
```
{% endraw %}

The rendered result should then look like this:

```html
<div id="app"></div>
<script>
    Vue.prototype.initData = JSON.parse(atob('eyJ1c2VyIjp7ImlkIjo5ODk3LCJ1c2V...=='));
</script>
```

In the next step we are embedding the vue file. 

The Twig `source` function returns the content of a vue file without rendering it. 

This is important because Twig and Vue use the same template syntax with`{{ }}`.

{% raw %}
```twig
{% block content %}

    <div id="app"></div>

    <script>
        Vue.prototype.initData = JSON.parse(atob('{{ data|json_encode|base64_encode }}'));
    </script>

    {{ source('Home/home-index.vue') }}

{% endblock %}
```
{% endraw %}

Note: This is just an example template, in real live this Twig template 
would be embeded in another layout template.

### Finished

Now Vue.js can start without waiting for the first Ajax response, 
and the "First Time to Interactive" is reduced to a minimum 
without any additional library.

Good luck :-).

![image](https://user-images.githubusercontent.com/781074/52744727-c459ad00-2fdd-11e9-90e4-d40cd40badb8.png)

