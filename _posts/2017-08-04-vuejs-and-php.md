---
title: VueJs and PHP
layout: post
comments: true
published: false
description: 
keywords: 
---

## Setup

* Download the development version from: <https://vuejs.org/js/vue.js> and the production version: <https://vuejs.org/js/vue.min.js>
* Copy this files into the `public/js` folder

## First test

* First we have a basic HTML boilerplate file. Create a test file: public/index.php
* A div with an ID of `#app` will contain our app.
* In the div, we have an h1 and a reference to the message piece of data.
* We instantiate a new Vue object and tell it with `el` that our app is the `#app` div.
* Finally, we provide the required data with the data object. In this case, we will provide the data for the message.

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Demo</title>
    <script src="js/vue.min.js"></script>
</head>
<body>
<div id="app">
    <h1>{{ message }}</h1>
</div>

<script>
    new Vue({
        el: "#app",
        data: {
            message: 'Hello Vue!'
        }
    });
</script>
</body>
</html>
```

You should see now: Hello Vue!

## Routing

Creating a single-page application (SPA) with Vue.js + vue-router is very easy. With Vue.js we already compose our application with components. When we add a the vue-router to the mix, all we need to do is map our components to the routes and tell the Vue router to render them. Here is a simple example:

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Demo</title>
    <script src="https://unpkg.com/vue/dist/vue.js"></script>
    <script src="https://unpkg.com/vue-router/dist/vue-router.js"></script>
</head>
<body>
<div id="app">
    <h1>Hello App!</h1>
    <p>
        <!-- use router-link component for navigation. -->
        <!-- specify the link by passing the `to` prop. -->
        <!-- `<router-link>` will be rendered as an `<a>` tag by default -->
        <router-link to="/foo">Go to Foo</router-link>
        <router-link to="/bar">Go to Bar</router-link>
        <router-link to="/user/1234">Go to user</router-link>
    </p>
    <!-- route outlet -->
    <!-- component matched by the route will render here -->
    <router-view></router-view>
</div>

<script>
    // 0. If using a module system (e.g. via vue-cli), import Vue and VueRouter and then call `Vue.use(VueRouter)`.

    // 1. Define route components.
    // These can be imported from other files
    const Foo = { template: '<div>Foo</div>' };
    const Bar = { template: '<div>Bar</div>' };
    const User = {
        props: ['id'],
        template: '<div>User {{ id }}</div>'
    };
    const NotFoundComponent = { template: '<div>Not found</div>' };

    // 2. Define some routes
    // Each route should map to a component. The "component" can
    // either be an actual component constructor created via
    // `Vue.extend()`, or just a component options object.
    // We'll talk about nested routes later.
    const routes = [
        { path: '/foo', component: Foo },
        { path: '/bar', component: Bar },
        { path: '/user/:id', component: User, props: true },
        { path: '*', component: NotFoundComponent }
    ];

    // 3. Create the router instance and pass the `routes` option
    // You can pass in additional options here, but let's
    // keep it simple for now.
    const router = new VueRouter({
        mode: 'history',
        routes: routes
    });

    //Vue.use(require('vue-chartist'));

    // 4. Create and mount the root instance.
    // Make sure to inject the router with the router option to make the
    // whole app router-aware.
    const app = new Vue({
        router: router
    }).$mount('#app');

    // Now the app has started!
</script>
</body>
</html>
```

* Open the console: `cmd`
* Change into to public directory: `cd c:\xampp\htdocs\example.com\public`
* Start the php webserver: `php -S localhost:8080`
* Open the url: <http://localhost:8080>

More details: <https://router.vuejs.org/en/essentials/getting-started.html>

## Package management

If you want to integrate and update a new VueJs package, you need `webpack` as module bundler. Its main purpose is to bundle JavaScript files for use in a browser, but it is also able to transform, bundle any resources or assetes.

### Installation

* Install node.js (with npm): <https://nodejs.org/en/download/>
* Install webpack: <https://webpack.js.org/guides/getting-started/>
* Create a Vue.js project: <https://skyronic.com/blog/vue-project-scratch>

### Minimal web app

```
<script src="js/app.js"></script>
<div id="app"></div>
```

### Read more

* <https://www.smashingmagazine.com/2018/02/jquery-vue-javascript/>