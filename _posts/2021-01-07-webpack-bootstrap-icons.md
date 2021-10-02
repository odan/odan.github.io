---
title: Webpack - Bootstrap Icons
layout: post
comments: true
published: true
description:
keywords: webpack, bootstrap, icons
image: https://icons.getbootstrap.com/assets/img/icons-hero.png
---

## Table of contents

* [Requirements](#requirements)
* [Introduction](#introduction)
* [Installation](#installation)
* [Usage](#usage)

## Requirements

* NPM

## Introduction

[Bootstrap Icons](https://icons.getbootstrap.com/) is a free, high quality,
open source icon library with nearly 1,200 icons.

I had some problems installing Bootstrap Icons with Webpack 5.
There was no information on the web, so I decided to post it here for people who might be interested.

## Installation

To get started, install via npm:

```
npm i bootstrap-icons
```

The `file-loader` resolves `import/require()` on a file into a url and emits 
the file into the output directory.

To install the `file-loader` run:

```
npm i file-loader --save-dev
```

Then in the `webpack.config.js` file, add a new module rule as follows:

```js
module: {
    rules: [
        {
            test: /\.woff(2)?(\?v=[0-9]\.[0-9]\.[0-9])?$/,
            include: path.resolve(__dirname, './node_modules/bootstrap-icons/font/fonts'),
            use: {
                loader: 'file-loader',
                options: {
                    name: '[name].[ext]',
                    outputPath: 'webfonts',
                    publicPath: '../webfonts',
                },
            }
        }
    ]
}
```

Optional change the `outputPath` and `publicPath`.

Then include the `bootstrap-icons.css` in your app js file:

```js
require('bootstrap-icons/font/bootstrap-icons.css');
```

Finally, build the assets:

```
npm run build
```

## Usage

Add HTML snippets to include Bootstrap Icons where desired.

```html
<i class="bi-alarm"></i>
```
