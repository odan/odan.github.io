---
title: Slim 4 - League Flysystem v2
layout: post
comments: true
published: true
description:
keywords: php, filesystem
image: https://odan.github.io/assets/images/slim-logo-600x330.png
---

## Table of contents

* [Requirements](#requirements)
* [Introduction](#introduction)
* [Installation](#installation)
* [Usage](#usage)
* [Conclusion](#conclusion)
* [Read more](#read-more)

## Requirements

* PHP 7.4+
* [A Slim 4 application](https://odan.github.io/2019/11/05/slim4-tutorial.html)

## Introduction

Handling files is an important part of any web application.
PHP provides several functions for creating, reading, uploading, and editing files.
There are also extra Composer packages for proprietary cloud-based file storage, 
such as Amazon S3.
When writing a test, you may need to simulate different types of filesystems
without "touching" the real filesystem of the disk, external FTPS server, cloud API, etc.
So you are dealing with many "filesystems" today, and maybe one day you 
will need to replace local storage with cloud-based storage without 
risking a complete rewrite of your application. 
Wouldn't it be nice to abstract the "filesystem" with a common 
interface, so you can replace the actual technology when needed?

The [League Flysystem package](https://github.com/thephpleague/flysystem) 
is a file storage library that provides one interface to interact with many 
types of filesystems. 
When you use Flysystem, you're not only protected from vendor lock-in, 
you'll also have a consistent experience for which ever storage is right for you.

Flysystem V2 is a full, from the ground up rewrite. It features simplifications of the interface, 
introduces a new error/exception handling strategy, and brings all the latest type-related 
features of PHP 7 to the filesystem. While the overall concept of Flysystem has not changed, 
V2 brings you a bunch of developer-experience improvements.

Now let's look at how to use Flysystem V2 in a Slim 4 application.

## Installation

To install Flysystem, run:

```
composer require league/flysystem
```

Make sure you have v2 installed:

```
composer show league/flysystem
```

## Configuration

Create a new directory in your project root directory: `storage/`

This directory will later act as the root-directory of the local filesystem adapter.

Insert the storage settings into your configuration file, e.g. `config/settings.php`;

```php
use League\Flysystem\Local\LocalFilesystemAdapter;
use League\Flysystem\UnixVisibility\PortableVisibilityConverter;

// ...

$settings['storage'] = [
    'adapter' => new LocalFilesystemAdapter(
        __DIR__ . '/../storage/',
        PortableVisibilityConverter::fromArray(
            [
                'file' => [
                    'public' => 0640,
                    'private' => 0604,
                ],
                'dir' => [
                    'public' => 0740,
                    'private' => 7604,
                ],
            ]
        ),
    ),
];
```

The [LocalFilesystemAdapter](https://flysystem.thephpleague.com/v2/docs/adapter/local/)
interacts with the local filesystem through Flysystem. In this example we also
add the unix permissions for new files and directories. You can change that if you want.
The LocalFilesystemAdapter uses the `LOCK_EX` operation by default to acquire an exclusive write lock.

When you run tests, you can define other adapters, for example the [Memory Filesystem Adapter](https://flysystem.thephpleague.com/v2/docs/adapter/in-memory/)
to run all file operations in-memory and not on the local filesystem.

The [SFTP Adapter](https://flysystem.thephpleague.com/v2/docs/adapter/sftp/) uses
Phpseclib to provide the same convenient interface for SFTP file transfers.

Read more: [Slim - SFTP Connection](2021-01-03-slim4-sftp.md)

## Container Setup

Insert a DI container definition for `Filesystem:class` in `config/container.php`:

```php
<?php

use League\Flysystem\Filesystem;
use Psr\Container\ContainerInterface;
// ...

return [

    // ...

    Filesystem::class => function (ContainerInterface $container) {
        $settings = $container->get('settings')['storage'];

        return new Filesystem($settings['adapter']);
    },

];

```

## Usage

To get the `Filesystem` instance via dependency injection, just declare it in
the constructor where you need it. Normally you should use the filesystem instance 
only within a domain or application service.

**Example**

```php
<?php

namespace App\Domain\Example\Service;

use League\Flysystem\Filesystem;

final class Example
{
    private Filesystem $filesystem;

    public function __construct(Filesystem $filesystem)
    {
        $this->filesystem = $filesystem;
    }

    public function foo(): void
    {
        $this->filesystem->write('test.txt', 'test');
    }
}
```

The `Filesystem::write` method creates a new file `test.txt` within the `storage/` directory
and stores the content `test` in the file.

```php
$this->filesystem->write('test.txt', 'test');
```

In vanilla PHP, the same line would be written as follows:

```php
if (@file_put_contents('storage/test.txt', 'test', LOCK_EX) === false) {
    throw new RuntimeException('Unable to write file: ' . error_get_last()['message']);
}
```

The difference is that Flysystem provides a very clean and simple OOP interface, 
better exception handling, a configured root-directory, 
and the possibility to replace the adapter for testing purposes.

## Conclusion

The Flysystem architecture is very flexible thanks to the adapter pattern.
In case you have special requirements, or your filesystem of choice is not available, 
you can always create your own adapter. Flysystem also provides its own mime-type detection.

## Read more

* [Flysystem Github repository](https://github.com/thephpleague/flysystem)
* [Flysystem V2 documentation](https://flysystem.thephpleague.com/v2/docs/)
* [Ask a question or report an issue](https://github.com/odan/slim4-tutorial/issues)
