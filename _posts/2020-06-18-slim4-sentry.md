---
title: Slim 4 - Sentry
layout: post
comments: true
published: true
description: 
keywords: php sentry error logging
---

## Table of contents

* [Requirements](#requirements)
* [Introduction](#introduction)
* [Installation](#installation)
* [Configuration](#configuration)
* [Usage](#usage)

## Requirements

* PHP 7.2+
* [A Slim 4 application](https://odan.github.io/2019/11/05/slim4-tutorial.html)
* A [Sentry account](https://sentry.io/signup/)

## Introduction

What is [Sentry](https://sentry.io/)? Sentry is an error tracking and monitoring tool that 
aggregates errors across your stack in real time. 

## Installation

Install the Sentry PHP SDK:

```
composer require sentry/sdk:2.1.0
```

## Configuration

To capture all errors, even the one during the startup of your application, 
you should initialize the Sentry PHP SDK as soon as possible.

To configure the SDK add this line with the public data source name (DSN) 
into your bootstrap file, e.g. in `config/bootstrap.php` 
directly after the composer autoloader:

```php
<?php

//...

require_once __DIR__ . '/../vendor/autoload.php';

Sentry\init(['dsn' => 'https://<key>@<organization>.ingest.sentry.io/<project>']);
```

## Usage

Once you have Sentry integrated into your project, you probably want to verify that 
everything is working as expected before deploying it.

One way to verify your setup is by intentionally sending an event that breaks your application.

You can throw an exception in your PHP application:

```php
throw new \Exception('My first Sentry error!');
```

Then login to your Sentry account and you should see all details of the error:

![image](https://user-images.githubusercontent.com/781074/85070056-70ed0580-b1b5-11ea-8537-ca6851154192.png)

Now that you’ve got basic reporting set up, you’ll want to explore adding additional context to your data.

**Read more:** [Next Steps](https://docs.sentry.io/error-reporting/quickstart/?platform=php#next-steps)
