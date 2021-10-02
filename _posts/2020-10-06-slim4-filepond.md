---
title: Slim 4 - FilePond
layout: post
comments: true
published: true
description: 
keywords: php, filepond, slim-framework
---

## Table of contents

* [Requirements](#requirements)
* [Introduction](#introduction)
* [Installation](#installation)
* [Storage](#storage)
* [Actions](#actions)
* [Security](#security)
* [Usage](#usage)
* [Read more](#read-more)

## Requirements

* PHP 7.3+
* [A Slim 4 application](https://odan.github.io/2019/11/05/slim4-tutorial.html)

## Introduction

[FilePond](https://pqina.nl/filepond/) is a JavaScript library that brings silky smooth drag n’ drop file uploading.

This tutorial shows how to implement the most basic API endpoints of FilePond in your slim application.

## Installation

To create unique filenames we have to install a UUID generator. Run:

```
composer require symfony/polyfill-uuid
```

## Storage

Create a new `tmp/upload/` directory in your project root
which later acts as your temporary upload directory.

Create a new `storage/` directory in your project root
which acts as your storage directory.

Add this `.htaccess` file to the `storage/` directory to 
prevent unwanted access from the web due to misconfiguration.

`deny from all`

Make sure that the `tmp/upload/` and the `storage/` directory has write access.

## Actions

We need at least 3 routes for this minimal application.

* `GET /filepond` - To show the upload form
* `POST /filepond/process` - To handle image uploads
* `DELETE /filepond/revert` - To revert (delete) the uploaded images

### Index

First we are preparing a simple HTML page and the action for the upload form itself.

Create a new directory (if not exists): `templates/`

Create a new template file in `templates/filepond.html` and copy/paste this content:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width">
    <title>FilePond PHP Boilerplate Project</title>
    <link href="https://unpkg.com/filepond/dist/filepond.css" rel="stylesheet">
    <link href="https://unpkg.com/filepond-plugin-image-preview/dist/filepond-plugin-image-preview.css" rel="stylesheet">
    <style>form { max-width:24em; }</style>
</head>
<body>

<form action="filepond/process" method="post" enctype="multipart/form-data">
    <input type="file" name="filepond[]" multiple>
    <button type="submit">Submit</button>
</form>

<script src="https://unpkg.com/filepond/dist/filepond.js"></script>
<script src="https://unpkg.com/filepond-plugin-image-preview/dist/filepond-plugin-image-preview.js"></script>

<script>
    FilePond.registerPlugin(
        FilePondPluginImagePreview,
    );

    // Set default FilePond options
    FilePond.setOptions({
        // upload to this server end point
        server: {

            process:(fieldName, file, metadata, load, error, progress, abort, transfer, options) => {

                // fieldName is the name of the input field
                // file is the actual file object to send
                const formData = new FormData();
                formData.append(fieldName, file, file.name);

                const request = new XMLHttpRequest();
                request.open('POST', 'filepond/process');

                // Should call the progress method to update the progress to 100% before calling load
                // Setting computable to false switches the loading indicator to infinite mode
                request.upload.onprogress = (e) => {
                    progress(e.lengthComputable, e.loaded, e.total);
                };

                // Should call the load method when done and pass the returned server file id
                // this server file id is then used later on when reverting or restoring a file
                // so your server knows which file to return without exposing that info to the client
                request.onload = function () {
                    if (request.status >= 200 && request.status < 300) {
                        // the load method accepts either a string (id) or an object
                        load(request.responseText);
                    } else {
                        // Can call the error method if something is wrong, should exit after
                        error('oh no');
                    }
                };

                request.send(formData);
            },
            revert: 'filepond/revert',
            restore: 'filepond/restore?id=',
            fetch: 'filepond/fetch?data=',
            load: 'filepond/load',
            fetch: 'filepond/fetch'
        },
    });

    const pond = FilePond.create(document.querySelector('input[type="file"]'));
</script>

</body>
</html>
```

Then add this action class into: `src/Action/FilePondIndexAction.php`:

```php
<?php

namespace App\Action;

use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface;

final class FilePondIndexAction
{
    public function __invoke(
        ServerRequestInterface $request,
        ResponseInterface $response
    ): ResponseInterface {
        $template = __DIR__ . '/../../templates/filepond.html';
        $response->getBody()->write(file_get_contents($template));

        return $response;
    }
}
```

Add this route into your routing file, e.g. in `config/routes.php`:

```php
$app->get('/filepond', \App\Action\FilePondIndexAction::class);
```

### Process

To handle the [image upload](https://pqina.nl/filepond/docs/patterns/api/server/#process) we add a new action class.

Create this action class in `src/Action/FilePondProcessAction.php`:

```php
<?php

namespace App\Action;

use App\Support\FilenameFilter;
use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface;
use Psr\Http\Message\UploadedFileInterface;
use RuntimeException;
use Slim\Psr7\UploadedFile;

final class FilePondProcessAction
{
    private $tempDirectory = __DIR__ . '/../../tmp/upload';

    private $storageDirectory = __DIR__ . '/../../storage';

    /**
     * Process upload.
     *
     * @see https://pqina.nl/filepond/docs/patterns/api/server/#process
     *
     * @param ServerRequestInterface $request The request
     * @param ResponseInterface $response The response
     *
     * @return ResponseInterface The response
     */
    public function __invoke(
        ServerRequestInterface $request,
        ResponseInterface $response
    ): ResponseInterface {
        /** @var UploadedFile[] $uploadedFiles */
        $uploadedFiles = (array)($request->getUploadedFiles()['filepond'] ?? []);

        if ($uploadedFiles) {
            return $this->moveTemporaryUploadedFile($uploadedFiles, $response);
        }

        $submittedIds = (array)($request->getParsedBody()['filepond'] ?? []);
        if ($submittedIds) {
            return $this->storeUploadedFiles($submittedIds, $response);
        }

        return $response->withStatus(422);
    }

    /**
     * Saves file to unique location and returns unique location id.
     *
     * @param UploadedFile[] $uploadedFiles
     * @param ResponseInterface $response
     *
     * @return ResponseInterface
     */
    private function moveTemporaryUploadedFile(
        array $uploadedFiles,
        ResponseInterface $response
    ): ResponseInterface {
        $fileIdentifier = '';

        foreach ($uploadedFiles as $uploadedFile) {
            if ($uploadedFile->getError() !== UPLOAD_ERR_OK) {
                continue;
            }
            $fileIdentifier = $this->moveUploadedFile($this->tempDirectory, $uploadedFile);
        }

        // Server returns unique location id in text/plain response
        $response = $response->withHeader('Content-Type', 'text/plain');
        $response->getBody()->write($fileIdentifier);

        return $response;
    }

    /**
     * Uses the unique id to move the ids to its final location 
     * and remove the temp files.
     *
     * @param string[] $submittedIds
     * @param ResponseInterface $response
     *
     * @throws RuntimeException
     *
     * @return ResponseInterface
     */
    private function storeUploadedFiles(
        array $submittedIds,
        ResponseInterface $response
    ): ResponseInterface {
        foreach ($submittedIds as $submittedId) {
            // Save the file into the file storage
            $submittedId = FilenameFilter::createSafeFilename($submittedId);
            $sourceFile = sprintf('%s/%s', $this->tempDirectory, $submittedId);
            $targetFile = sprintf('%s/%s', $this->storageDirectory, $submittedId);

            if (!copy($sourceFile, $targetFile)) {
                throw new RuntimeException(
                    sprintf('Error moving uploaded file %s to the storage', $submittedId)
                );
            }

            if (!unlink($sourceFile)) {
                throw new RuntimeException(
                    sprintf('Error removing uploaded file %s', $submittedId)
                );
            }
        }

        // Server returns unique location id in text/plain response
        $response = $response->withHeader('Content-Type', 'text/plain');

        return $response->withStatus(201);
    }

    /**
     * Moves the uploaded file to the upload directory and assigns it a unique name
     * to avoid overwriting an existing uploaded file.
     *
     * @param string $directory The directory to which the file is moved
     * @param UploadedFileInterface $uploadedFile The file uploaded file to move
     *
     * @return string The filename of moved file
     */
    private function moveUploadedFile(
        string $directory,
        UploadedFileInterface $uploadedFile
    ): string {
        $extension = (string)pathinfo(
            $uploadedFile->getClientFilename(),
            PATHINFO_EXTENSION
        );

        // Create unique id for this file
        $filename = FilenameFilter::createSafeFilename(
            sprintf('%s.%s', (string)uuid_create(), $extension)
        );

        // Save the file into the storage
        $targetPath = sprintf('%s/%s', $directory, $filename);
        $uploadedFile->moveTo($targetPath);

        return $filename;
    }

}
```

Add this route into your routing file, e.g. in `config/routes.php`:

```php
$app->post('/filepond/process', \App\Action\FilePondProcessAction::class);
```

### Revert

To provide the [revert](https://pqina.nl/filepond/docs/patterns/api/server/#revert) functionality, 
add this new action class in `src/Action/FilePondRevertAction.php`:

```php
<?php

namespace App\Action;

use App\Support\FilenameFilter;
use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface;

final class FilePondRevertAction
{
    private $tempDirectory = __DIR__ . '/../../tmp/upload';

    /**
     * Revert upload.
     *
     * @see https://pqina.nl/filepond/docs/patterns/api/server/#revert
     *
     * @param ServerRequestInterface $request The request
     * @param ResponseInterface $response The response
     *
     * @return ResponseInterface The response
     */
    public function __invoke(
        ServerRequestInterface $request,
        ResponseInterface $response
    ): ResponseInterface {
        // The server uses the unique id to remove the file
        $filename = FilenameFilter::createSafeFilename((string)$request->getBody());

        if (!$filename) {
            return $response;
        }

        $fullPath = sprintf('%s/%s', $this->tempDirectory, $filename);

        if (file_exists($fullPath)) {
            unlink($fullPath);
        }

        return $response;
    }
}
```

Add this route into your routing file, e.g. in `config/routes.php`:

```php
$app->delete('/filepond/revert', \App\Action\FilePondRevertAction::class);
```

## Security

For security reasons all requested filenames must be "sanitized" to prevent
unwanted filesystem manipulations.

Create a class in `src/Support/FilenameFilter.php` and copy this content:

```php
<?php

namespace App\Support;

use UnexpectedValueException;

final class FilenameFilter
{
    /**
     * Makes file name safe to use.
     *
     * @param string $file The name of the file [not full path]
     * 
     * @throws UnexpectedValueException 
     *
     * @return  string The sanitised string
     */
    public static function createSafeFilename(string $file): string
    {
        // Reject funky whitspace paths.
        if (preg_match('#\p{C}+#u', $file)) {
            throw new UnexpectedValueException('Corrupted path detected');
        }
        // Remove any trailing dots, as those aren't ever valid file names.
        $file = trim($file, '.');

        return trim(preg_replace(['#(\.){2,}#', '#[^A-Za-z0-9\.\_\- ]#', '#^\.#'], '', $file));
    }
}
```

## Usage

Now, when you enter the website, e.g. `http://localhost/filepond`, the page should look like this:

![image](https://user-images.githubusercontent.com/781074/95257595-d4627800-0824-11eb-9fe8-a2e8bf32f1dd.png)

Then click "Browse" or use Drag and Drop to upload some images.

![image](https://user-images.githubusercontent.com/781074/95257827-302d0100-0825-11eb-9458-5b12e344fb58.png)

To revert (delete) the image, just click the undo button.

Now you are able to upload, preview and revert images uploads.
FilePond offers much more plugins, e.g. for image manipulation, but this is out of the scope of this article.  

## Read more

* [FilePond website](https://pqina.nl/filepond/)
* [FilePond documentation](https://pqina.nl/filepond/docs/)
* [FilePond API documentation](https://pqina.nl/filepond/docs/patterns/api/server/)
* [FilePond Github repository](https://github.com/pqina/filepond)
* [Slim forum](https://discourse.slimframework.com)
* [Ask a question or report an issue](https://github.com/odan/slim4-tutorial/issues)
