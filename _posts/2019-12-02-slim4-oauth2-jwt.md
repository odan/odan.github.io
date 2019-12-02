---
title: Slim 4 - OAuth 2.0 and JSON Web Token (JWT) Setup.
layout: post
comments: true
published: false
description: 
keywords: php slim oauth jwt json authentication security
---

## Table of contents

* [Requirements](#requirements)
* [Introduction](#introduction)
* [Installation](#installation)
* [Usage](#usage)

## Requirements

* PHP 7.1+
* Composer
* [A Slim 4 application](https://odan.github.io/2019/11/05/slim4-tutorial.html)

## Introduction

This tutorial demonstrates how to implement the [OAuth 2.0](https://oauth.net/2/) authentication 
standard in combination with a [JSON Web Token](https://oauth.net/2/jwt/) (JWT).

Please note that a logout functionality with tokens is not feasible without 
giving up the stateless principle.

## Installation

[lcobucci/jwt](https://github.com/lcobucci/jwt) is a very good library to work 
with JSON Web Token (JWT) and JSON Web Signature based on RFC 7519.

The Package is available on [packagist](https://packagist.org/packages/lcobucci/jwt), 
you can install it using composer:

```
composer require lcobucci/jwt
```

## Configuration

Todo: private key setup

## Creating a token

## Validating a token

## Routing and middleware

