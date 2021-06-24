---
title: PHP - Queues with RabbitMQ
layout: post
comments: false
published: false
description:
keywords: php, rabbitmq, queue, work, worker, queues
image:
---

## Table of contents

* [Requirements](#requirements)
* [Introduction](#introduction)
* [Queues](#queues)
* [RabbitMQ Setup](#rabbitmq-setup)  
  * [RabbitMQ Setup on Linux](#rabbitmq-setup-on-linux)
  * [RabbitMQ Setup on Windows](#rabbitmq-setup-on-windows)
  * [RabbitMQ Management Interface Setup](#rabbitmq-management-interface-setup)
* [Installation](#installation)
* [Event Dispatcher](#event-dispatcher)
* [Configuration](#configuration)
* [DI Container Setup](#di-container-setup)
* [Usage](#usage)
  * [Sending](#sending)
  * [Receiving](#receiving)
* [Testing](#testing)
* [Deploying to Production](#deploying-to-production)
* [Supervisor Configuration](#supervisor-configuration)
* [Conclusion](#conclusion)
* [Read more](#read-more)

## Requirements

* PHP 7.2+
* The PHP `ext-sockets` extension

## Introduction

[RabbitMQ](https://www.rabbitmq.com/) is one of the most popular and deployed open source message broker.
RabbitMQ runs on many operating systems and cloud environments, 
and provides a wide range of developer tools for most popular languages (such as PHP).
As a message-oriented middleware, RabbitMQ can be used to implement the 
Advanced Message Queuing Protocol (AMQP) on all modern operating systems.

## Queues

Message queueing allows web servers to respond to requests quickly instead of being forced to perform resource-heavy
procedures on the spot that may delay response time. Message queueing is also good when you want to distribute a message
to multiple consumers or to balance loads between workers.

The **consumer** takes a **message** off the **queue** and starts processing the reports or E-Mails.
At the same time, the **producer** is queueing up **new messages**.

The consumer can be on a totally different server than the producer or they can be
located on the same server. The request can be created in one programming language and handled in another programming
language. The point is, the two applications will only communicate through the messages they are sending to each other,
which means the sender and receiver have low coupling.

## RabbitMQ Setup

For this demo you need at least a running RabbitMQ instance.
This section describes how RabbitMQ can be installed and configured.

### RabbitMQ Setup on Linux

Please follow the official installation guide:

* [Downloading and Installing RabbitMQ](https://www.rabbitmq.com/download.html)
* [Install RabbitMQ on Ubuntu](https://www.vultr.com/docs/how-to-install-rabbitmq-on-ubuntu-16-04-47)

### RabbitMQ Setup on Windows

Memcached works on most Linux and BSD like systems. There is no official support for windows builds.

RabbitMQ requires a 64-bit supported version of Erlang for Windows to be installed. 
The latest binary builds for Windows can be obtained from the Erlang/OTP page. 
Erlang must be installed using an administrative account.

* [OTP 23.2 Windows 64-bit Binary File](https://www.erlang.org/downloads)

Once a supported version of Erlang is installed, download the RabbitMQ installer,
`rabbitmq-server-{version}.exe` and run it. It is highly recommended that RabbitMQ is also installed as an
administrative account. The setup installs RabbitMQ as a Windows service and starts it using the default configuration.

* [Download RabbitMQ for windows](https://www.rabbitmq.com/install-windows.html#downloads)

On Windows, CLI tools have a `.bat` suffix compared to other platforms. For example, `rabbitmqctl` on Windows is invoked
as `rabbitmqctl.bat`.

Start the "RabbitMQ Command Prompt" from the Windows start menu:

![image](https://user-images.githubusercontent.com/781074/107159581-5e684c00-6991-11eb-82ad-e1882c0eb97b.png)

This will run the commands from the `sbin` folder of the RabbitMQ installation path,
e.g. `C:\Program Files\RabbitMQ Server\rabbitmq_server-3.8.12-rc.1\sbin`

### RabbitMQ Management Interface Setup

Now that we have RabbitMQ installed and started we want to install the management interface for the browser.
The [management plugin](https://www.rabbitmq.com/management.html) is not enabled by default, so you need to run the
below command to enable it:

```
rabbitmq-plugins enable rabbitmq_management
```

The management interface runs on port 15672 by default, it is possible the server/network is blocking this port. You
will need to check that the port is open.

Now open: **<http://localhost:15672/>** in your browser. You should see the login.

![image](https://user-images.githubusercontent.com/781074/107159718-2281b680-6992-11eb-9e26-57023d1be4a2.png)

Use `guest` as username and `guest` as password to login.

This is the web based management interface:

![image](https://user-images.githubusercontent.com/781074/107159865-fb77b480-6992-11eb-8377-d4cdf396b001.png)

At this point in time we have no queues, see `Queues`.

## Installation

RabbitMQ uses a protocol called AMQP by default. 
To be able to communicate with RabbitMQ you need a library that understands the same protocol as RabbitMQ. 
This tutorial covers [AMQP 0-9-1](https://www.rabbitmq.com/tutorials/amqp-concepts.html), 
which is an open, general-purpose protocol for messaging. 
There are a number of clients for RabbitMQ in many languages. 

Next we have to install a RabbitMQ client for PHP. We'll use the `php-amqplib` in this tutorial. 
Open your project and run:

```
composer require php-amqplib/php-amqplib
```

Now that we have the php-amqplib library installed, we can write some code.

For the event queue dispatcher we also need the 
[PSR-14: Event Dispatcher](https://www.php-fig.org/psr/psr-14/) interfaces.

```
composer require psr/event-dispatcher
```

## Event Dispatcher

In the next step we will create a very lightweight dispatcher
so that the events can be dispatched from an event dispatcher instance.
The event dispatcher follows the PSR-14 guidelines for implementation.

First we extend an interface from `EventDispatcherInterface` to have a unique identifier
for the DI container.

Filename: `src/Queue/Dispatcher/MessageDispatcherInterface.php`.

```php
<?php

namespace App\Queue\Dispatcher;

use Psr\EventDispatcher\EventDispatcherInterface;

interface MessageDispatcherInterface extends EventDispatcherInterface
{
}
```

Next we create a RabbitMQ implementation of that `EventDispatcherInterface`.

Filename: `src/Queue/Dispatcher/RabbitMessageDispatcher.php`

<div style="page-break-after: always; visibility: hidden"> 
\pagebreak 
</div>

```php
<?php

namespace App\Queue\Dispatcher;

use JsonException;
use PhpAmqpLib\Channel\AMQPChannel;
use PhpAmqpLib\Message\AMQPMessage;

final class RabbitMessageDispatcher implements MessageDispatcherInterface
{
    private AMQPChannel $channel;

    public function __construct(AMQPChannel $channel)
    {
        $this->channel = $channel;
    }

    /**
     * Dispatches the given message.
     *
     * @param object $event The message
     *
     * @throws JsonException
     *
     * @return object The message
     */
    public function dispatch(object $event)
    {
        $properties = [
            'content_type' => 'application/json',
            'delivery_mode' => AMQPMessage::DELIVERY_MODE_PERSISTENT,
        ];

        $body = [
            'class_name' => get_class($event),
            'payload' => $event,
        ];

        $message = new AMQPMessage(json_encode($body, JSON_THROW_ON_ERROR), $properties);

        // Optional: Map class type to exchange and routing key
        // if($event instanceof MailMessageEvent) { ... }

        // Default
        $exchange = '';
        $routingKey = 'events';

        // Push to queue
        $this->channel->basic_publish($message, $exchange, $routingKey);

        return $event;
    }
}

```

The `RabbitMessageDispatcher` from above will push an object into a queue as a JSON string.
Just make sure that the consumer can decode it.
When the queue name (e.g. events) does not exist, it will be created.
Please note: This is just a very basic example. If you have another routing keys or exchange key you can change the implementation to your specific needs.

## Configuration

Insert the settings into your configuration file, e.g. `config/settings.php`;

```php
$settings['queue'] = [
    'driver' => \App\Queue\Dispatcher\RabbitMessageDispatcher::class,
    // For testing with phpunit
    //'driver' => \App\Queue\Dispatcher\NullMessageDispatcher::class,
    'config' => [
        'host' => '127.0.0.1',
        'port' => 5672,
        'vhost' => '/',
        'username' => 'guest',
        'password' => 'guest',
    ],
];
```

For performance reason you may better use the IP address of the host.

## DI Container Setup

Add the following DI container definition in `config/container.php` for the RabbitMQ
connection and channel.

```php
<?php

use App\Queue\Dispatcher\MessageDispatcherInterface;
use PhpAmqpLib\Channel\AMQPChannel;
use PhpAmqpLib\Connection\AMQPStreamConnection;
use Psr\Container\ContainerInterface;
// ...

return [
    // ...
    
    AMQPStreamConnection::class => function (ContainerInterface $container) {
        $queue = $container->get('settings')['queue'];
    
        return new AMQPStreamConnection(
            $queue['host'],
            $queue['port'],
            $queue['username'],
            $queue['password'],
            $queue['vhost']
        );
    },
    
    AMQPChannel::class => function (ContainerInterface $container) {
        $connection = $container->get(AMQPStreamConnection::class);
    
        $channel = $connection->channel();
    
        // Declare queues here...
        $channel->queue_declare('events', false, false, false, false);
    
        return $channel;
    },
    
    MessageDispatcherInterface::class => function (ContainerInterface $container) {
        $queue = $container->get('settings')['queue'];

        return $container->get($queue['driver']);
    },
];
```

## Usage

### Sending

Events can be dispatched from the message dispatcher instance.
Sending a new message to a queue works like shown below:

```php
<?php

use App\Queue\Dispatcher\MessageDispatcherInterface;
use App\Queue\Message\RegisteredUserNotification;
use App\Domain\User\Message\RegisteredUserMessage;

final class Example
{
    private MessageDispatcherInterface $messageDispatcher;

    public function __construct(MessageDispatcherInterface $messageDispatcher)
    {
        $this->messageDispatcher = $messageDispatcher;
    }

    public function demo()
    {
        $this->messageDispatcher->dispatch(new RegisteredUserMessage(123));
    }
}
```

The event class is just a simple DTO with public properties for the payload. Example:

```php
<?php

namespace App\Domain\User\Message;

final class RegisteredUserMessage
{
    public string $userId;

    public function __construct(int $userId)
    {
        $this->userId = $userId;
    }
}
```

<div style="page-break-after: always; visibility: hidden"> 
\pagebreak 
</div>

### Receiving

Depending on your requirements and server-setup the message receiver (consumer)
has to run all the time to check for new messages from the queue.

Filename: `bin/consumer.php`

```php
<?php

use PhpAmqpLib\Connection\AMQPStreamConnection;
use PhpAmqpLib\Message\AMQPMessage;
use Psr\Container\ContainerInterface;

require_once __DIR__ . '/../vendor/autoload.php';

/** @var ContainerInterface $container */
$container = (require __DIR__ . '/../config/bootstrap.php')->getContainer();

$connection = $container->get(AMQPStreamConnection::class);

$channel = $connection->channel();

// Limit the worker to only process messages from specific queues
$channel->queue_declare('events', false, false, false, false);

$callback = function (AMQPMessage $msg) {
    echo ' [x] Received ' . $msg->body . "\n";
};

$channel->basic_consume('events', '', false, true, false, false, $callback);

while ($channel->is_consuming()) {
    $channel->wait();
}

$channel->close();
$connection->close();
```

To start the consumer, run: `php bin/consumer.php`

As soon as your application dispatches a message the consumer 
will print it to the screen:

```
[x] Received {"class_name":"RegisteredUserNotification","payload":{"userId":"123"}}
```

## Testing

For testing purposes it would be better to abstract away the real `php-amqplib`
implementation behind an Interface. With a memory queue you would then be able to 
publish and receive messages without a real RabbitMQ server.

For testing purposes you may not really want to test against a real RabbitMQ server.
So when you run your phpunit tests, you can use the `NullMessageDispatcher` instead
This a special transport implementation, kind of stub. 
It does not send nor receive anything.

Create a new file `src/Queue/Dispatcher/NullMessageDispatcher.php` and copy this code:

```php
<?php

namespace App\Queue\Dispatcher;

final class NullMessageDispatcher implements MessageDispatcherInterface
{
    /**
     * Dispatches the given message.
     *
     * @param object $event The message
     *
     * @return object The message
     */
    public function dispatch(object $event)
    {
        return $event;
    }
}
```

Then change the transport driver in your test environment specific config file to:

```php
'driver' => \App\Queue\Dispatcher\NullMessageDispatcher::class,
```

Example:

```php
$settings['queue'] = [
    'driver' => \App\Queue\Dispatcher\NullMessageDispatcher::class,
    'config' => [],
];
```

## Deploying to Production

On production, there are a few important things to think about.

**Use Supervisor to keep your worker(s) running**

You’ll want one or more “workers” running at all times.
To do that, use a process control system like [Supervisor](http://supervisord.org/).

**Don’t Let Workers Run Forever**

Please take the following into account when running your consumer
(or any PHP code for that matter) for a long time:

Try to avoid memory usage accumulation.
Don't keep appending to arrays without ever clearing them.
This means, for instance, that you **shouldn't**
use the [FingersCrossedHandler](https://github.com/Seldaek/monolog/blob/main/src/Monolog/Handler/FingersCrossedHandler.php)
in Monolog since this keeps a buffer of log messages. Even when you are careful,
PHP might **leak memory**.

**Restart Workers on Deploy**

Each time you deploy, you’ll need to restart all your worker processes so that they see the newly deployed code.
To do this, run `supervisorctl restart consumer_01` on deploy.
Then, Supervisor will create new worker processes.
A mix of both would be to have a cronjob installed that restarts the workers every night, but in theory the consumers
could run about a month before they run out of memory.

## Supervisor Configuration

Supervisor is a great tool to guarantee that your worker process(es) is "always" running.
You can install it on Ubuntu, for example, via: 

`sudo apt-get install supervisor`

Supervisor configuration files typically live in a `/etc/supervisor/conf.d` directory. 
For example, you can create a new `messenger-worker.conf` file there to make sure that 
1 instances of `consume.php` are running at all times:

```
;/etc/supervisor/conf.d/messenger-worker.conf
[program:consumer]
command=php /path/to/your/app/bin/consumer.php
user=ubuntu
numprocs=1
startsecs=0
autostart=true
autorestart=true
process_name=%(program_name)s_%(process_num)02d
```

Change the `user` to the Unix user on your server.
Note that each worker needs a unique consumer name to avoid the same message being handled by multiple workers.

Next, tell Supervisor to read your config and start your workers:

```
sudo supervisorctl reread
sudo supervisorctl update
sudo supervisorctl start consumer:*
```

See the [Supervisor docs](http://supervisord.org/) for more details.

## Conclusion

As you can see, you can implement a very lightweight queue with relatively little 
effort and few dependencies, which can also be easily tested with phpunit. 
Since the topic is very complex and extensive, 
I could of course only show here the most important aspects. 
The shown approach is not only limited to RabbitMQ, 
but can also be replaced by other queue solutions by implementing your own `MessageEventDispatcher`. 
If this is still not enough for you, you can also have a look at 
the [symfony/messager](https://symfony.com/doc/current/messenger.html) component.
It always depends on your specific requirements. Don't forget: *Keep it simple.*

## Read more

* [The RabbitMQ  website](https://www.rabbitmq.com/)
* [RabbitMQ Tutorial for PHP](https://www.rabbitmq.com/tutorials/tutorial-one-php.html)
* [Ask a question or report an issue](https://github.com/odan/slim4-tutorial/issues)