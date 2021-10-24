---
title: Slim 4 - Webpack
layout: post
comments: true
published: true
description: 
keywords: slim-framework, slim4, php, webpack, assets, js, css
image: https://odan.github.io/assets/images/slim-logo-600x330.png
---

## Table of contents

* [Requirements](#requirements)
* [Introduction](#introduction)
* [Directory structure](#directory-structure)
* [Webpack setup](#webpack-setup)
* [Twig Webpack extension setup](#twig-webpack-extension-setup)
* [Creating assets](#creating-assets)
* [Compiling assets](#compiling-assets)
* [Useful tips](#useful-tips)
  * [Recompiling on change](#recompiling-on-change)
  * [jQuery Setup](#jquery-setup)
  * [Bootstrap Setup](#bootstrap-setup)
  * [Fontawesome Setup](#fontawesome-setup)
  * [SweetAlert2 Setup](#sweetalert2-setup)
  * [DataTables Setup](#datatables-setup)
  * [Babel Setup](#babel-setup)

## Requirements

* [Slim 4](https://www.slimframework.com/docs/v4/start/installation.html)
* [NPM](https://nodejs.org/en/download/)

## Introduction

If you've ever been confused and overwhelmed about getting started with Webpack and asset compilation, you should read further. 
In this tutorial we are using [Webpack](https://webpack.js.org) to bundle (compile, combine and minimize) web assets like JavaScript modules and CSS files.


## Directory structure

```
/                              the root of your project
/composer.json
/webpack.config.js             the main config file for Webpack
/templates/                    the twig templates and page specific assets
/templates/home/               
/templates/user/
/public/                       the webservers document root
/public/index.php              the front controller (application entry point)
/public/assets                 the compiled assets (webpack output)
/public/assets/manifest.json   the generated manifest file (required by the webpack twig extension)
```

## Webpack setup

Create a new `package.json` file at the root of your project. This file lists the packages your project depends on:

```json
{
    "name": "my-app",
    "version": "1.0.0",
    "license": "MIT",
    "private": true,
    "dependencies": {
        "foreach": "^2",
        "promise-polyfill": "8",
        "url-search-params-polyfill": "^8"
    },
    "devDependencies": {
        "autoprefixer": "^10",
        "clean-webpack-plugin": "^4",
        "css-loader": "^6",
        "css-minimizer-webpack-plugin": "^3",
        "file-loader": "^6",
        "mini-css-extract-plugin": "^2",
        "postcss-loader": "^6",
        "sass-loader": "^12",
        "style-loader": "^3",
        "terser-webpack-plugin": "^5",
        "url-loader": "^4",
        "webpack": "^5",
        "webpack-assets-manifest": "^5",
        "webpack-cli": "^4",
        "webpack-manifest-plugin": "^4"
    }
}

```

Create a new `webpack.config.js` file at the root of your project. This is the main config file for Webpack:

```js
const path = require('path');
const webpack = require('webpack');
const {CleanWebpackPlugin} = require('clean-webpack-plugin');
const {WebpackManifestPlugin} = require('webpack-manifest-plugin');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');
const CssMinimizerPlugin = require('css-minimizer-webpack-plugin');
const TerserJSPlugin = require('terser-webpack-plugin');

module.exports = {
    entry: {
        'home/home-index': './templates/home/home-index.js'
        // here you can add more entries for each page or global assets 
        // 'layout/layout': './templates/layout/layout.js'
    },
    output: {
        path: path.resolve(__dirname, 'public/assets'),
        publicPath: 'assets/',
    },
    optimization: {
        minimizer: [new TerserJSPlugin({}), new OptimizeCSSAssetsPlugin({})],
    },
    performance: {
        maxEntrypointSize: 1024000,
        maxAssetSize: 1024000
    },
    resolve: {
        extensions: ['.js']
    },
    module: {
        rules: [
            {
                test: /\.css$/,
                use: [
                    {
                        loader: MiniCssExtractPlugin.loader,
                    },
                    {
                        loader: "css-loader",
                        options: {
                            url: true,
                            esModule: false,
                        },
                    },
                ],
            },
        ],
    },
    plugins: [
        new CleanWebpackPlugin(),
        new ManifestPlugin(),
        new MiniCssExtractPlugin({
            ignoreOrder: false
        }),
    ],
    watchOptions: {
        ignored: ['./node_modules/']
    },
    mode: "development"
};

```

They key part is the `entry` object: This tells Webpack to load the `templates/home/home-index.js` file and follow all of the require / import statements. 
It will then package everything together and - thanks to the first `home/home-index` key - output final `home/home-index.js` and `home/home-index.css` files into the `public/assets/` directory. Later you can add more page-specific JavaScript or CSS to the `entry` object.

Note that Webpack uses the [TerserWebpackPlugin](https://webpack.js.org/plugins/terser-webpack-plugin/) to minify your JavaScript and the [OptimizeCSSAssetsPlugin](https://github.com/NMFR/optimize-css-assets-webpack-plugin) to minify your CSS files.

To install webpack and all dependencies, run:

```
npm install
```

### Twig Webpack extension setup

Now we install the Twig Webpack extension with composer:

```
composer require fullpipe/twig-webpack-extension
```

Register the WebpackExtension Twig extension within your `Twig::class` container definition: 

```php
use Psr\Container\ContainerInterface;
use Slim\App;
use Slim\Views\Twig;
// ...

return [
    // Application settings
    'settings' => function () {
        return require __DIR__ . '/settings.php';
    },

    // The Slim app factory
    App::class => function (ContainerInterface $container) {
        AppFactory::setContainer($container);

        return AppFactory::create();
    },

    // The Twig template engine
    Twig::class => function (ContainerInterface $container) {
        $settings = (array)$container->get('settings');
        $twigSettings = $settings['twig'];

        $twig = Twig::create($twigSettings['paths'], $twigSettings['options']);
        // ...

        // The path must be absolute.
        // e.g. /var/www/example.com/public
        $publicPath = (string)$settings['public'];

        // Add extensions
        $twig->addExtension(new \Fullpipe\TwigWebpackExtension\WebpackExtension(
            // The manifest file.
            $publicPath . '/assets/manifest.json',
            // The public path
            $publicPath
        ));

        return $twig;
    },

    // Add more container entries ...
];
```

* The first parameter (manifestFile) defines the location of your `manifest.json` file. You can also use `__PATH__` here.
* The second parameter (publicPathJs) defines the public path for the js files.
* The third parameter (publicPathCss) defines the public path for the css files.

It's recommended to create the Slim app instance within the container (and not somewhere "outside").
Just make sure that your bootstrap process also uses exactly this app instance from the container.

Example for `config/bootstrap.php`:

```php
<?php

use DI\ContainerBuilder;
use Slim\App;

require_once __DIR__ . '/../vendor/autoload.php';

$containerBuilder = new ContainerBuilder();

// Add container definitions
$containerBuilder->addDefinitions(__DIR__ . '/container.php');

// Build PHP-DI Container instance
$container = $containerBuilder->build();

// Create App instance
$app = $container->get(App::class);

// Register routes
// ...

// Register middleware
// ...

return $app;
```

## Creating assets

Next, create a new `templates/home/home-index.js` file with some basic JavaScript and import some CSS with `require`:

```js
require('./home-index.css');

alert('Hello World!');
```

Create the new `templates/home/home-index.css` file:

```css
body {
    background-color: lightgray;
}
```

Create a twig template file `templates/layout/layout.twig` with this content:

{% raw %}
```twig
<!DOCTYPE html>
<html>
    <head>
        <!-- ... -->
        
        {% block css %}{% endblock %}
    </head>
    <body>
        <!-- ... -->
        {% block content %}{% endblock %}

        {% block js %}{% endblock %}
    </body>
</html>
```
{% endraw %}

Create a twig template file `templates/home/home-index.twig` with this content:

{% raw %}
```twig
{% extends "layout/layout.twig" %}

{% block css %}
    {% webpack_entry_css 'home/home-index' %}
{% endblock %}

{% block js %}
    {% webpack_entry_js 'time/time-index' %}
{% endblock %}

{% block content %}

Welcome

{% endblock %}
```
{% endraw %}

Notice: The 'home/home-index' must match the first key of the `entry` item in webpack.config.js.

The `webpack_entry_css` and `webpack_entry_js` will fetch the url from the webpack `manifest.json` file and outputs the appropriate html link tags. For example:

```html
<link type="text/css" href="/assets/home/home-index.css" rel="stylesheet">
<script type="text/javascript" src="/assets/home/home-index.js"></script>  
```

## Compiling assets

To build the assets for development, run:

```
npx webpack --mode=development
```

or just:

```
npx webpack
```

To compile and minify the assets for production, run:

```
npx webpack --mode=production
```

**Congrats!** You now have three new files:

* `public/assets/home/home-index.js`  (holds all the JavaScript for your "home/home-index" entry)
* `public/assets/home/home-index.css` (holds all the CSS for your "home/home-index" entry)
* `public/assets/manifest.json` (holds all the entries and filenames)

## Useful tips

### Recompiling on change

Webpack can watch and recompile files whenever they change.

```
npx webpack --watch
```

To stop the webpack watch process, press `Ctrl+C`.

Read more:
* [Webpack Watch and WatchOptions](https://webpack.js.org/configuration/watch/)

### jQuery Setup

jQuery uses the global variable `global.jQuery` and the 
alias `global.$`. The problem ist that Webpack will wrap all 
modules within a closure function to protect the global scope. 
For this reason we have to bind the jQuery instance to the 
global scope manually.

To install jQuery, run:

```
npm install jquery
```

The browser must load jQuery before other jQuery plugins can be used.
For this reason, I would recommend bundling jQuery into a generally available asset file.

Add a new webpack entry in `webpack.config.js`:

```js
module.exports = {
    entry: {
        'layout/layout': './templates/layout/layout.js',
        // ...
    },
    // ...
};
```

Bind jQuery to the global scope in your webpack entry point `templates/layout/layout.js`:

```js
global.jQuery = require('jquery');
global.$ = global.jQuery;
```

Add the assets {% raw %}`{% webpack_entry_css 'layout/layout' %}`{% endraw %} and {% raw %}`{% webpack_entry_js 'layout/layout' %}`{% endraw %} to the Twig template `layout/layout.twig`:

{% raw %}
```twig
<!DOCTYPE html>
<html>
    <head>
        <!-- ... -->
        
        {% webpack_entry_css 'layout/layout' %}
        
        {% block css %}{% endblock %}
        
        {% webpack_entry_js 'layout/layout' %}
    </head>
    <body>
        <!-- ... -->
        {% block content %}{% endblock %}

        {% block js %}{% endblock %}
    </body>
</html>
```
{% endraw %}

### Bootstrap Setup

To install Bootstrap 5, run:

```
npm install bootstrap
```

Bootstrap depends on Popper, which is specified in the peerDependencies property. 
This means that you will have to make sure to add it to your package.json 
using this command:

```
npm install @popperjs/core
```

Import boostrap in a global available webpack entry point like: `templates/layout/layout.js`:

```js
global.bootstrap = require('bootstrap');
require('@popperjs/core');
require('bootstrap/dist/css/bootstrap.css');
```

### Fontawesome Setup

[Fontawesome](https://fontawesome.com/) is the world's most popular 
and easiest to use icon set.

To install Fontawesome, run:

```
npm install @fortawesome/fontawesome-free
```

We also need the file-loader for the webfonts:

```
npm install file-loader --save-dev
```

We want to copy the Fontawesome fonts automatically into the `assets` directory. 
The file-loader copies the font files, to the build directory.
To install the file-loader, run:

```
npm install file-loader --save-dev
```

Import Fontawesome in a global available webpack entry like: `templates/layout/layout.js`:

You can import all fontawesome icons...

```js
require('@fortawesome/fontawesome-free/css/all.min.css');
```

... or you can import only a specific set of icons:

```js
require('@fortawesome/fontawesome-free/css/fontawesome.css');
require('@fortawesome/fontawesome-free/css/v4-shims.css');
require('@fortawesome/fontawesome-free/css/regular.css');
require('@fortawesome/fontawesome-free/css/solid.css');
require('@fortawesome/fontawesome-free/css/brands.css');
```

**Note:** We don't install the js dependencies here, because it would blow up 
your js build to >1 MB of useless javascript. We only need the plain 
css and webfont files for fontawesome.

To copy the fonts into the `assets/webfonts/` directory, 
add this rule to your `webpack.config.js` file:

```js
 module: {
        rules: [
            // ...
            {
                test: /\.(ttf|eot|svg|woff|woff2)(\?[\s\S]+)?$/,
                include: path.resolve(__dirname, './node_modules/@fortawesome/fontawesome-free/webfonts'),
                use: {
                    loader: 'file-loader',
                    options: {
                        name: '[name].[ext]',
                        outputPath: 'webfonts',
                        publicPath: '../webfonts',
                    },
                }
            },
        ],
    },
```

### SweetAlert2 Setup

[SweetAlert2](https://sweetalert2.github.io/) is a beautiful, 
responsive, customizable and accessible replacement for JavaScript's popup boxes.

To install SweetAlert2, run:

```
npm install sweetalert2
```

Import the sweetalert2 module and bind `Swal` to the global scope:

```js
global.Swal = require('sweetalert2');
```

Swal for Bootstrap 5:

```js
global.sweetalert2 = require('sweetalert2');
global.Swal = global.sweetalert2.mixin({
    // Fix for Bootstrap 5
    scrollbarPadding: false,
    heightAuto: false,
});
```

Usage:

```js
Swal.fire(
  'Good job!',
  'You clicked the button!',
  'success'
);
```

### DataTables Setup

[DataTables.net](https://datatables.net/) is a very flexible table plug-in for jQuery.

You have to [setup jQuery for Webpack](#jquery-setup) first.

To install the DataTables Core library, run:

```
npm install datatables.net
npm install datatables.net-dt
```

To install the DataTables Bootstrap 5 styles, run:

```
npm install datatables.net-bs5
npm install datatables.net-responsive-bs5
npm install datatables.net-select-bs5
```

Add a new webpack entry in `webpack.config.js`:

```js
module.exports = {
    entry: {
        'layout/layout': './templates/layout/layout.js',
        'layout/datatables': './templates/layout/datatables.js', // <-- add this line
        'user/user-list': './templates/user/user-list.js', // <-- add this line
        // other pages ...
    },
    // ...
};
```

Import datatables in a global available webpack entry point like: 
`templates/layout/datatables.js`:

```js
// Register a jQuery Plugin
global.$.fn.DataTable = require('datatables.net-bs5');

// Datatables Bootstrap 5 styles
require('datatables.net-bs5/css/dataTables.bootstrap5.css');
```

Create a new Twig template: `templates/user/user-list.twig`:

{% raw %}
```twig
{% extends "layout/layout.twig" %}

{% block css %}
    {% webpack_entry_css 'layout/datatables' %}
{% endblock %}

{% block js %}
    {% webpack_entry_js 'layout/datatables' %}
    {% webpack_entry_js 'user/user-list' %}
{% endblock %}

{% block content %}
  <div id="content" class="container">
        <div class="row">
            <div class="col-md-12">
                <h1><i class="fas fa-user"></i> User list</h1>
                <hr>
                <table id="my-data-table" class="table table-striped table-bordered dt-responsive nowrap dataTable no-footer dtr-inline collapsed">
                    <thead>
                    <tr>
                        <th>Username</th>
                        <th>E-Mail</th>
                        <th>First name</th>
                        <th>Last name</th>
                    </tr>
                    <tfoot></tfoot>
                </table>
                <p></p>
            </div>
        </div>
{% endblock %}
```
{% endraw %}

Call this single function in `templates/user/user-list.js`:

```js
$(function() {
    $('#my-data-table').DataTable();
});
```

## Babel Setup

Installation:

```
npm install --save-dev babel-loader @babel/core @babel/preset-env webpack
```

Add this rule to your `webpack.config.js` file:

```js
 module: {
        rules: [
            // ...
            {
                test: /\.js$/,
                exclude: path.resolve('node_modules'),
                use: [{
                    loader: 'babel-loader',
                    options: {
                        presets: [
                            ['@babel/preset-env']
                        ]
                    }
                }]
            },
        ],
    },
```
