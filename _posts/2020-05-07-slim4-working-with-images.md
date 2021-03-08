---
title: Slim 4 - Images
layout: post
comments: true
published: true
description: 
keywords: php, slim, files, images, png, jpg, gif, slim-framework
---

## Table of contents

* [Requirements](#requirements)
* [Introduction](#introduction)
* [Installation](#installation)
* [Integration in Slim](#integration-in-slim)
* [Creating & Sending Images](#creating--sending-images)
* [Creating images using the GD and Image functions](#creating-images-using-the-gd-and-image-functions)

## Requirements

* PHP 7.2+
* [A Slim 4 application](https://odan.github.io/2019/11/05/slim4-tutorial.html)
* GD Library **or** Imagick PHP extension

## Introduction

To display **static** images in the browser you can simply store 
your image files in a public accessible directory, e.g. in `public/images`
and link them in HTML using the **[img](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/img)**
element.

For example:

```html
<img src="images/logo.png" width="120">
```

However, this tutorial explains how to generate or convert **dynamic** images 
and stream these files to the client as a download or inline image.

This technique can be useful if your client uploads some photos, and you want to display them
in a smaller size. It can also be used to display dynamic charts, etc. The use cases are not limited.

## Installation

My favorite library for image manipulation is [Intervention Image](http://image.intervention.io/).

The best way to install Intervention Image is quickly and easily with Composer.

To install the most recent version, run the following command:

```
composer require intervention/image
```

## Integration in Slim

Add the image manager settings to your Slim settings array, e.g in `config/settings.php`:

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

return [

    // ...
    
    ImageManager::class => function (ContainerInterface $container) {
        return new ImageManager($container->get('settings')['image_manager']);
    },
];
```

## Creating & Sending Images

This single action controller shows how to create an image (in memory) and send it to the client:

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

    public function __invoke(
        ServerRequestInterface $request,
        ResponseInterface $response
    ): ResponseInterface {
        // Create image in memory
        $image = $this->imageManager->canvas(800, 600, '#719e40');

        // Encode image into given format (PNG) and quality (100%)
        $data = $image->encode('png', 100)->getEncoded();

        // Detect and set the correct content-type, e.g. image/png
        $mime = finfo_buffer(finfo_open(FILEINFO_MIME_TYPE), $data);
        $response = $response->withHeader('Content-Type', $mime);

        // Output image as stream
        return $response->withBody((new StreamFactory())->createStream($data));
    }
}
```

**Image formats**

By default, Intervention Image supports the following major formats:

* <http://image.intervention.io/getting_started/formats>

## Sending Image Files

If you want to stream an existing file the just use the `make($filename)` method instead:

```php
$image = $this->imageManager->make('public/foo.png');
```

## Creating images using the GD and Image functions

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
    public function __invoke(
        ServerRequestInterface $request,
        ResponseInterface $response
    ): ResponseInterface {
        // Create image in memory
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

        // Detect and set the correct content-type, e.g. image/png
        $mime = finfo_buffer(finfo_open(FILEINFO_MIME_TYPE), $data);
        $response = $response->withHeader('Content-Type', $mime);

        // Output image as stream
        return $response->withBody((new StreamFactory())->createStream($data));
    }
}

```
