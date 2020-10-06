---
title: Slim 4 - Respect\Validation
layout: post
comments: false
published: false
description: 
keywords: php validaton slim
---

## Table of contents

* [Requirements](#requirements)
* [Introduction](#introduction)
* [Installation](#installation)
* [Usage](#usage)
* [Read more](#read-more)

## Requirements

* PHP 7.3+
* [A Slim 4 application](https://odan.github.io/2019/11/05/slim4-tutorial.html)
* [Twig for Slim](https://odan.github.io/2020/04/17/slim4-twig-templates.html)

## Introduction

Respect\Validation is a quite popular and flexible validation library for PHP.
It works as standalone libary and without much additional dependencies.

The fluent builder makes it very easy to validate even nested nested objects and arrays.

## Installation

To install Respect\Validation, run:

```
composer require respect/validation
```

## Usage

A lot of magic is going on when you use the static method of the validator.
I prefer less magic and more rubust and faster code. For this reason I try to avoid
static methods and create a new instance of the Validator class as follows:

```php
use Respect\Validation\Validator as v;

$validator = new v();

//...
```

## Read more

* [The Respect\Validation Github repository](https://github.com/Respect/Validation)
* [The Respect\Validation documentation](https://respect-validation.readthedocs.io/en/latest/)
* [What makes Respect\Validation awesome?](https://www.reddit.com/r/PHP/comments/1telis/respectvalidation_the_most_awesome_validation/ce7hvcs/?utm_source=reddit&utm_medium=web2x&context=3)
* [Slim forum](https://discourse.slimframework.com)
* [Feedback and support](https://github.com/odan/slim4-tutorial/issues)
