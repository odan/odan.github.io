---
title: Slim 4 - Mailer
layout: post
comments: true
published: true
description: 
keywords: php, slim, email, smtp, symfony, slim-framework
image: https://odan.github.io/assets/images/slim-logo-600x330.png
---

## Table of contents

* [Requirements](#requirements)
* [Introduction](#introduction)
* [Installation](#installation)
* [Configuration](#configuration)
* [Creating and Sending Messages](#creating-and-sending-messages)
* [Twig Integration](#twig-integration)
* [Error handling](#error-handling)
* [Read more](#read-more)

## Requirements

* PHP 7.2+
* [A Slim 4 application](https://odan.github.io/2019/11/05/slim4-tutorial.html)

## Introduction

This tutorial explains how to integrate and use the 
[Symfony Mailer](https://symfony.com/doc/current/mailer.html)
component for creating and sending emails.

## Installation

To install the mailer component, run:

```
composer require symfony/mailer
```

## Configuration

Emails are delivered via a "transport". For this tutorial we deliver emails over SMTP.

Only for demonstration purposes I use a free [Mailtrap](https://mailtrap.io/) account. 

In Mailtrap you can create a forwarding rule to your real inbox.
Mailtrap keeps a copy of your message in any case.

Add the email settings to your Slim settings array, e.g `config/settings.php`:

```php
// E-Mail settings
$settings['smtp'] = [
    // use 'null' for the null adapter
    'type' => 'smtp',
    'host' => 'smtp.mailtrap.io',
    'port' => '25',
    'username' => 'my-username',
    'password' => 'my-secret-password',
];
```

Autowire the mailer using `MailerInterface`:

```php
<?php

use Psr\Container\ContainerInterface;
use Symfony\Component\Mailer\Mailer;
use Symfony\Component\Mailer\MailerInterface;
use Symfony\Component\Mailer\Transport;
use Slim\App;
use Slim\Factory\AppFactory;

return [

    // ...
    
    // SMTP transport
    MailerInterface::class => function (ContainerInterface $container) {
        $settings = $container->get('settings')['smtp'];
        // or
        // $settings = $container->get('settings')['smtp'];
        
        // smtp://user:pass@smtp.example.com:25
        $dsn = sprintf(
            '%s://%s:%s@%s:%s',
            $settings['type'],
            $settings['username'],
            $settings['password'],
            $settings['host'],
            $settings['port']
        );

        return new Mailer(Transport::fromDsn($dsn));
    },
];
```

If you don't use the Configuration class, just use `settings` as container entry:

```php
$settings = $container->get('settings')['smtp'];
```

## Creating and Sending Messages

Let the DIC inject the `MailerInterface` object and create an new `Email` object:

```php
<?php

namespace App\Domain\User\Service;

use Symfony\Component\Mailer\MailerInterface;
use Symfony\Component\Mime\Email;

final class ContactMailer
{
    /**
     * @var MailerInterface
     */
    private $mailer;

    public function __construct(MailerInterface $mailer)
    {
        $this->mailer = $mailer;
    }

    public function sendEmail(array $formData): void
    {
        // Validate form data
        $this->validate($formData);

        // Send email
        $email = (new Email())
            ->from('hello@example.com')
            ->to('you@example.com')
            //->cc('cc@example.com')
            //->bcc('bcc@example.com')
            //->replyTo('john.doe@example.com')
            //->priority(Email::PRIORITY_HIGH)
            ->subject('Time for Symfony Mailer!')
            ->text('Sending emails is fun again!')
            ->html('<p>My HTML content</p>');

        $this->mailer->send($email);
    }

    private function validate(array $data): void
    {
        // ...
    }
}
```

## Twig Integration

The **Twig bridge** provides integration for Twig with various Symfony components.
To install the Twig Bridge, run:

```
composer require symfony/twig-bridge
```

To render Twig templates in emails just add a new container definition for 
`BodyRendererInterface:class` in `config/container.php`

```php
use Symfony\Component\Mime\BodyRendererInterface;
use Slim\Views\Twig;
use Psr\Container\ContainerInterface;
// ...

BodyRendererInterface::class => function(ContainerInterface $container)
{
    return new BodyRenderer($container->get(Twig::class)->getEnvironment());
},
```

To define the contents of your email with Twig, use the `TemplatedEmail` class (and not the `Email` class). 
This class extends the normal `Email` class but adds some new methods for Twig templates.

Then inject the `BodyRendererInterface` instance via constructor injection where you need it.

## Usage

```php
use Symfony\Bridge\Twig\Mime\TemplatedEmail;

$email = (new TemplatedEmail())
    ->from('from@example.com')
    ->to(new Address('to@example.com'))
    ->subject('Thanks for signing up!')

    // path of the Twig template to render
    ->htmlTemplate('emails/signup.html.twig')

    // pass variables (name => value) to the template
    ->context([
        'username' => 'foo',
    ])
;

 // Render the email twig template
$this->bodyRenderer->render($email);

// Send email
$this->mailer->send($email);
```

## Error handling

To catch all mailer errors, just add a try/catch block around the send method:

```php
try {
    $this->mailer->send($email);
} catch (Exception $exception) {
   // handle error here...
}
```

## Read more

* [Symfony Mailer documentation](https://symfony.com/doc/current/mailer.html)
* [Symfony Mailer on Github](https://github.com/symfony/mailer)
* [Ask a question or report an issue](https://github.com/odan/slim4-tutorial/issues)
