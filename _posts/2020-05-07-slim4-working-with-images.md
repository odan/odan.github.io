---
title: Slim 4 - Working with images
layout: post
comments: true
published: true
description: 
keywords: php slim files images png jpg gif
---

## Table of contents

* [Requirements](#requirements)
* [Introduction](#introduction)
* [Installation](#installation)
* [Integration in Slim](#integration-in-slim)
* [Creating & Sending Images](#creating--sending-images)
* [Creating images GD and Image functions](#creating-images-gd-and-image-functions)

## Requirements

* PHP 7.2+
* [A Slim 4 application](https://odan.github.io/2019/11/05/slim4-tutorial.html)
* GD Library **or** Imagick PHP extension

## Introduction

This tutorial explains how to create images and stream that files
as download or inline image to the client (browser).

## Installation

My favorite library for image manipulation is [Intervention Image](http://image.intervention.io/).

The best way to install Intervention Image is quickly and easily with Composer.

To install the most recent version, run the following command:

```
composer require intervention/image
```

## Integration in Slim

Intervention uses static method for the configuration, which is not so good 
for the container setup, but lucculy we can configure and autowire the `` class.

Add the email settings to your Slim settings array, e.g `config/settings.php`:

```php
$settings['image_manager'] = [
    // configure image driver (gd by default)
    'driver' => 'gd',
];
```

Autowire the image manager using the `ImageManager` class:

```php
<?php

use Intervention\Image\ImageManager;
use Psr\Container\ContainerInterface;
use Selective\Config\Configuration;

return [

    // ...
    
    ImageManager::class => function (ContainerInterface $container) {
        return new ImageManager($container->get(Configuration::class)->getArray('image_manager'));
    },
];
```

## Creating & Sending Images

Let the `ImageManager` instance, create the image and send it to the client:

```php
<?php

namespace App\Action;

use Intervention\Image\ImageManager;
use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface;
use Slim\Psr7\Factory\StreamFactory;

final class ImageExampleAction
{
    /**
     * @var ImageManager
     */
    private $imageManager;

    public function __construct(ImageManager $imageManager)
    {
        $this->imageManager = $imageManager;
    }

    public function __invoke(ServerRequestInterface $request, ResponseInterface $response): ResponseInterface
    {
        // Create image in memory
        $image = $this->imageManager->canvas(800, 600, '#719e40');

        // Sends HTTP response with current image in given format (PNG) and quality (100%)
        $data = $image->encode('png', 100)->getEncoded();

        // Set content-type
        $mime = finfo_buffer(finfo_open(FILEINFO_MIME_TYPE), $data);
        $response = $response->withHeader('Content-Type', $mime);

        // Output image as stream
        $streamFactory = new StreamFactory();
        $stream = $streamFactory->createStream($data);

        return $response->withBody($stream);
    }
}
```

**Image formats**

The possible image formats depend on the choosen driver (GD or Imagick) 
and your local configuration. By default Intervention Image currently supports 
the following major formats:

* <http://image.intervention.io/getting_started/formats>

## Sending Image Files

If you want to stream an existing file the just use the `make($filename)` method instead:

```php
$image = $this->imageManager->make('public/foo.png');
```

## Creating images GD and Image functions

If you want to use the native [GD and image function](https://www.php.net/manual/en/ref.image.php)
from PHP, then you don't have to install the Intervention Image component. The trick here is that we
fetch the raw stream from the [imagepng](https://www.php.net/manual/en/function.imagepng.php)
or [imagejpeg](https://www.php.net/manual/en/function.imagejpeg.php) function and
put that data into the PSR-7 response object. Here is an example:

```php
<?php

namespace App\Action;

use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface;
use Slim\Psr7\Factory\StreamFactory;

final class ImageCreateAction
{
    public function __invoke(ServerRequestInterface $request, ResponseInterface $response): ResponseInterface
    {
        // Create image in memmory
        $image = imagecreate(200, 80);
        imagecolorallocate($image, 255, 255, 255);
        $textColor = imagecolorallocate($image, 113, 158, 64);
        $lineColor = imagecolorallocate($image, 170, 170, 170);
        imagestring($image, 8, 35, 25, 'slim framework', $textColor);
        imagesetthickness($image, 2);
        imageline($image, 35, 45, 160, 45, $lineColor);

        // Fetch the raw image stream
        ob_start();
        imagepng($image);
        $data = ob_get_clean();
        imagedestroy($image);

        // Detect the correct content-type, e.g. image/png
        $mime = finfo_buffer(finfo_open(FILEINFO_MIME_TYPE), $data);
        $response = $response->withHeader('Content-Type', $mime);

        // Output image as stream
        return $response->withBody((new StreamFactory())->createStream($data));
    }
}

```


[Donate](../../../donate.html)
