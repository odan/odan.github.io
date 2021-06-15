---
title: Slim 4 - Spam Protection
layout: post
comments: true
published: true
description:
keywords: php, spam, protection
image: https://odan.github.io/assets/images/slim-logo-600x330.png
---

## Table of contents

* [Requirements](#requirements)
* [Introduction](#introduction)
* [Installation](#installation)
* [Designing a Spam Checker Class](#designing-a-spam-checker-class)
* [Configuration](#configuration)
* [Container Setup](#container-setup)
* [Checking Comments for Spam](#checking-comments-for-spam)
* [Testing and Mocking](#testing-and-mocking)
* [Conclusion](#conclusion)
* [Read more](#read-more)

## Requirements

* PHP 7.4+ or PHP 8.0+
* [A Slim 4 application](https://odan.github.io/2019/11/05/slim4-tutorial.html)
* A free Akismet account

## Introduction

Anyone can submit a feedback. Even robots, spammers, and more. We could add some
“captcha” to the form to somehow be protected from robots, or we can use some 
third-party APIs.

I have decided to use the free [Akismet](https://akismet.com) service to demonstrate 
how to call an API and how to make the call “out of band”.

## Signing up on Akismet

Sign-up for a free account on [akismet.com](https://akismet.com/signup/#personal) 
and get the Akismet API key.
Select the "Personal" account and change the price to `0/Year`.

## Installation

Instead of using a library that abstracts the Akismet API, we will do all the API calls directly. 
Doing the HTTP calls ourselves is more efficient.
To make API calls, use the Guzzle HTTP client component:

```
composer require guzzlehttp/guzzle
```

## Designing a Spam Checker Class

Create a new class named `SpamChecker` to wrap 
the logic of calling the Akismet API and interpreting its responses:

File: `src/Support/Akismet/SpamChecker.php`

```php
<?php

namespace App\Support\Akismet;

use GuzzleHttp\ClientInterface;
use RuntimeException;

final class SpamChecker
{
    public const NOT_SPAM = 0;

    public const MAYBE_SPAM = 1;

    public const BLATANT_SPAM = 2;

    private ClientInterface $client;

    public function __construct(ClientInterface $client)
    {
        $this->client = $client;
    }

    /**
     * Get spam score.
     *
     * @param SpamCheckerComment $comment The comment
     *
     * @throws RuntimeException
     *
     * @retrun int The spam score: 0: not spam, 1: maybe spam, 2: blatant spam
     */
    public function checkComment(SpamCheckerComment $comment): int
    {
        // Create context
        // https://akismet.com/development/api/#comment-check
        $context = [
            'blog' => $comment->blog,
            'comment_type' => $comment->type,
            'comment_author' => $comment->author,
            'comment_author_email' => $comment->authorEmail,
            'comment_content' => $comment->content,
            'comment_date_gmt' => $comment->date ? $comment->date->format('c') : null,
            'blog_lang' => $comment->language,
            'blog_charset' => $comment->charset,
            'user_ip' => $comment->userIp,
            'user_agent' => $comment->userAgent,
            'referrer' => $comment->referrer,
            'permalink' => $comment->permalink,
        ];

        $response = $this->client->request(
            'POST',
            'comment-check',
            [
                'form_params' => $context,
            ]
        );

        $headers = $response->getHeaders();
        if (($headers['X-akismet-pro-tip'][0] ?? '') === 'discard') {
            return static::BLATANT_SPAM;
        }

        $content = (string)$response->getBody();

        if (isset($headers['X-akismet-debug-help'][0])) {
            throw new RuntimeException(
                sprintf('Unable to check for spam: %s (%s).', $content, $headers['x-akismet-debug-help'][0])
            );
        }

        return $content === 'true' ? static::MAYBE_SPAM : static::NOT_SPAM;
    }
}

```

The HTTP client `request()` method submits a POST request to the Akismet endpoint
and passes an array of parameters.

The `checkComment()` method returns 3 values depending on the API call response:

* 2: if the comment is a “blatant spam”;
* 1: if the comment might be spam;
* 0: if the comment is not spam (ham).

We also need a DTO as parameter object for the spam checker.

File: `src/Support/Akismet/SpamCheckerComment.php`

```php
<?php

namespace App\Support\Akismet;

use DateTimeImmutable;

final class SpamCheckerComment
{
    public ?string $blog = null;
    public string $type = 'comment';
    public ?string $author = null;
    public ?string $authorEmail = null;
    public ?string $content = null;
    public ?DateTimeImmutable $date = null;
    public string $language = 'en';
    public string $charset = 'UTF-8';
    public bool $test = true;
    public ?string $userIp = null;
    public ?string $userAgent = null;
    public ?string $referrer = null;
    public ?string $permalink = null;
}

```

This DTO will be filled later with form data from the website.

## Configuration

All calls to Akismet are POST requests much like a web form would send. 
When making calls to Akismet, your key is used as a subdomain. 
So, if you had the key `123YourAPIKey`, you would make all API calls to
`https://123YourAPIKey.rest.akismet.com`.

Insert the settings into your configuration file, e.g. `config/settings.php`;

```php
$settings['akismet'] = [
    'base_uri' => 'https://%s.rest.akismet.com/1.1/',
    'api_key' => '123YourAPIKey',
];
```

**Security note:** In reality, you should not store the secret API-Key (`api_key`) here. Instead, the
secret should be stored in a server specific file for all secrets, e.g. in `env.php`.

## Container Setup

The SpamChecker class relies on an GuzzleClient as argument.

Insert a new DI container definition for `SpamChecker:class` in `config/container.php`.

```php
<?php

use App\Support\Akismet\SpamChecker;
use GuzzleHttp\Client;
use Psr\Container\ContainerInterface;

return [
    // ...

   SpamChecker::class => function (ContainerInterface $container) {
        $settings = $container->get('settings')['akismet'];
        $apiKey = $settings['api_key'];
        $baseUri = $settings['base_uri'];

        $client = new Client(
            [
                'base_uri' => sprintf($baseUri, $apiKey),
            ]
        );

        return new SpamChecker($client);
    },
];
```

## Checking Comments for Spam

One simple way to check for spam when a new comment is submitted is to call the spam 
checker before storing the data into the database:

```php
use App\Support\Akismet\SpamChecker;
use App\Support\Akismet\SpamCheckerComment;
use DateTimeImmutable;

// ...

$formData = (array)$request->getParsedBody();

$comment = new SpamCheckerComment();
$comment->blog = 'https://guestbook.example.com';
$comment->author = (string)$formData['author'];
$comment->authorEmail = (string)$formData['email'];
$comment->content = (string)$formData['content'];
$comment->date = new DateTimeImmutable('now');
$comment->language = 'en';
$comment->test = false;
$comment->authorEmail = (string)$formData['email'];
$comment->userIp = $request->getServerParams()['REMOTE_ADDR'] ?? null;
$comment->userAgent = $request->getHeaderLine('User-Agent');
$comment->referrer = $request->getHeaderLine('Referer');
$comment->permalink = (string)$request->getUri();

$score = $this->spamChecker->checkComment($comment);

if ($score !== SpamChecker::NOT_SPAM) {
    $response->getBody()->write('Spam detected!');

    return $response->withStatus(403);
}

// Save the data into the database
// ...

// Create response
return $response->withStatus(201);
```

To test if the spam filtering is working correctly, try inputting `viagra-test-123`
into the `author` field or `akismet-guaranteed-spam@example.com`
into the `email` field, and then submit the form.
With these magic words reserved for testing, Akismet must return a "spam" response.

## Testing and Mocking

For testing purpose you have to make sure that you don't hit the real HTTP endpoints.
When you test with phpunit you can inject a mocked Guzzle client into the SpamChecker
to simulate the response only in memory.

Guzzle provides a mock handler that can be used to fulfill HTTP requests with a response or 
exception by shifting return values of a queue.

Before each test you could define a response queue and add it to the mocked Guzzle client 
like this:

```php
use GuzzleHttp\Client;
use GuzzleHttp\Handler\MockHandler;
use GuzzleHttp\HandlerStack;
use GuzzleHttp\Psr7\Response;
use GuzzleHttp\Psr7\Request;
use GuzzleHttp\Exception\RequestException;

//...

public function testAction(): void
{
    // Create a mock and queue
    $responses = [
        new Response(200, [], 'true'),
    ];
    $client = new Client(['handler' => HandlerStack::create(new MockHandler($responses))]);

    $this->container->set(SpamChecker::class, new SpamChecker($client));

    $request = $this->createRequest('POST', '/blog/comments');
    $request = $request->withParsedBody(
        [
            'author' => 'viagra-test-123',
            'email' => 'akismet-guaranteed-spam@example.com',
            'content' => 'test comment'
        ]
    );
    $response = $this->app->handle($request);

    $this->assertStringContainsString('Spam detected!', (string)$response->getBody());
    $this->assertSame(403, $response->getStatusCode());
}
```

## Conclusion

Now you are able to check the comments for possible spam.
This general concept is testable without touching the real endpoints 
and can be applied to other HTTP API clients / SDKs as well.

## Read more

* [Documentation](https://akismet.com/development/api/#detailed-docs)
* [Ask a question or report an issue](https://github.com/odan/slim4-tutorial/issues)
