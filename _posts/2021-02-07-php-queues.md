---
title: PHP - RabbitMQ
layout: post
comments: true
published: false
description:
keywords: php, rabbitmq, queue, work, worker, queues
image: 
---

## Table of contents

* [Requirements](#requirements)
* [Introduction](#introduction)
* [Installation](#installation)
* [Usage](#usage)
* [Testing](#testing)
* [Conclusion](#conclusion)
* [Read more](#read-more)

## Requirements

* PHP 7.2+
* Windows

## Introduction

This guide describes how RabbitMQ can be installed and configured manually on Windows.

## Installation

RabbitMQ requires a 64-bit supported version of Erlang for Windows to be installed. 
Latest binary builds for Windows can be obtained from the Erlang/OTP page.
Erlang must be installed using an administrative account.

* [OTP 23.2 Windows 64-bit Binary File](https://www.erlang.org/downloads)

Once a supported version of Erlang is installed, download the RabbitMQ installer,
`rabbitmq-server-{version}.exe` and run it.
It is highly recommended that RabbitMQ is also installed as an administrative account.
The setup installs RabbitMQ as a Windows service and starts it using the default configuration.

* [Download RabbitMQ for windows](https://www.rabbitmq.com/install-windows.html#downloads)

On Windows, CLI tools have a `.bat` suffix compared to other platforms. 
For example, `rabbitmqctl` on Windows is invoked as `rabbitmqctl.bat`.

Start the "RabbitMQ Command Prompt" from the Windows start menu:

![image](https://user-images.githubusercontent.com/781074/107159581-5e684c00-6991-11eb-82ad-e1882c0eb97b.png)

This will run the commands from the `sbin` folder of the RabbitMQ installation path,
e.g. `C:\Program Files\RabbitMQ Server\rabbitmq_server-3.8.12-rc.1\sbin`

Now that we have RabbitMQ installed and started we want to install the management interface for the browser.
The [management plugin](https://www.rabbitmq.com/management.html) is not enabled by default, so you need to run the below command to enable it:

```
rabbitmq-plugins enable rabbitmq_management
```

The management interface runs on port 15672 by default, 
it is possible the server/network is blocking this port. 
You will need to check that the port is open.

Now open: **<http://localhost:15672/>** in your browser. You should see the login.

![image](https://user-images.githubusercontent.com/781074/107159718-2281b680-6992-11eb-9e26-57023d1be4a2.png)

Use `guest` as username and `guest` as password to login.

This is the web based management interface:

![image](https://user-images.githubusercontent.com/781074/107159865-fb77b480-6992-11eb-8377-d4cdf396b001.png)

At this point in time we have no queues, see `Queues`.

Next we have to install a RabbitMQ client for PHP.

RabbitMQ uses a protocol called AMQP by default. 
To be able to communicate with RabbitMQ you need a library that understands 
the same protocol as RabbitMQ. This tutorial covers [AMQP 0-9-1](https://www.rabbitmq.com/tutorials/amqp-concepts.html),
which is an open, general-purpose protocol for messaging. There are a number of clients for RabbitMQ 
in many languages. We'll use the `php-amqplib` in this tutorial, and Composer for dependency management.

Open your PHP project and run:

```
composer require php-amqplib/php-amqplib
```

Now that we have the php-amqplib library installed, we can write some code.

## Usage

Message queueing allows web servers to respond to requests quickly instead
of being forced to perform resource-heavy procedures on the spot that may
delay response time. Message queueing is also good when you want to distribute
a message to multiple consumers or to balance loads between workers.

The consumer takes a message off the queue and starts processing the PDF's, Reports or E-Mails.
At the same time, the producer is queueing up new messages.
The consumer can be on a totally different server than the producer or they
can be located on the same server. The request can be created in one
programming language and handled in another programming language.
The point is, the two applications will only communicate through the
messages they are sending to each other, which means the sender and
receiver have low coupling.

### Sending

Send a new message to a queue works like shown below:

```php
use PhpAmqpLib\Connection\AMQPStreamConnection;
use PhpAmqpLib\Message\AMQPMessage;

$connection = new AMQPStreamConnection('localhost', 5672, 'guest', 'guest');
$channel = $connection->channel();

$channel->queue_declare('hello', false, false, false, false);

$msg = new AMQPMessage('Hello World!');
$channel->basic_publish($msg, '', 'hello');

$channel->close();
$connection->close();
```

The example from above will push a `Hello World!` payload into a queue with the name `hello`.
When the queue name does not exist, it will be created. Instead of a plain string, it's
also possible to send a JSON or XML string as payload. Just make sure
that the consumer can decode it.

### Receiving

Depending on your requirements and server-setup the message receiver (consumer) 
has to run all the time to check for new messages from the queue. 

**Example**

```php
use PhpAmqpLib\Connection\AMQPStreamConnection;
use PhpAmqpLib\Message\AMQPMessage;

$connection = new AMQPStreamConnection('localhost', 5672, 'guest', 'guest');
$channel = $connection->channel();

$channel->queue_declare('hello', false, false, false, false);

$callback = function ($msg) {
    echo ' [x] Received ' . $msg->body . "\n";
};

$channel->basic_consume('hello', '', false, true, false, false, $callback);

while ($channel->is_consuming()) {
    $channel->wait();
}

$channel->close();
$connection->close();
```

The simplest solution would be to start the consumer script via crontab every minute.

The more accepted method is to keep your consumer running. 
There are tools like [Supervisor](http://supervisord.org/) and 
[Circus](https://circus.readthedocs.io/en/latest/) that can help you with that. 
If you can get your consumer to exit when there 
are no more messages, you could also use cron. This might cause 
a delay in consuming the messages. You can't react on messages instantly. 
Users might have to wait a minute before any task is started / mail is received.

Please take the following into account when running your consumer 
(or any PHP code for that matter) for a long time:

Try to avoid memory usage accumulation. Don't keep appending to arrays 
without ever clearing them. This means for instance that you **shouldn't**
use [FingersCrossedHandler](https://github.com/Seldaek/monolog/blob/main/src/Monolog/Handler/FingersCrossedHandler.php)
in Monolog since this keeps a buffer of log messages. Even when you are
careful, PHP might **leak memory**. Well it's, PHP... 

A mix of both would be to have a cronjob installed that restarts the workers every night, 
but in theory the consumers could run about a month before they run out of memory.

## Testing

The **[queue-interop](https://github.com/queue-interop/queue-interop)** project
tries to identify and standardize a common way for PHP programs to create, 
send, receive and read MQ messages to achieve interoperability.

For testing purpose it would be better to abstract away the real `php-amqplib`
implementation behind an Interface. With a memory queue you
would then be able to publish and receive messages without a real RabbitMQ server.   

## Conclusion

Installing a RabbitMQ server on Windows was not that complicated as it might sound.
You may know that PHP is not the best tool for long-running tasks. So be careful
not to create memory leaks and monitor (or restart) the consumer(s).
In some very special cases you might also consider other programming languages
to implement the consumers. It always depends on your specific requirements.
Don't forget: *Keep it simple.*

## Read more

* [The RabbitMQ  website](https://www.rabbitmq.com/)
* [RabbitMQ Tutorial for PHP](https://www.rabbitmq.com/tutorials/tutorial-one-php.html)