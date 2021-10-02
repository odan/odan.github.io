---
title: Creating and downloading Excel files in Slim 3
layout: post
comments: true
published: true
description: 
keywords: 
---

I would like to show you how to create Excel files and download them automatically.

To create Excel files I'm using [PhpSpreadsheet](https://github.com/PHPOffice/PhpSpreadsheet). 

## Installation

To install PhpSpreadsheet run:

```bash
composer require phpoffice/phpspreadsheet
```

## Action

Create a new route:

```php
<?php

use PhpOffice\PhpSpreadsheet\Shared\File;
use PhpOffice\PhpSpreadsheet\Spreadsheet;
use PhpOffice\PhpSpreadsheet\Writer\Xlsx;
use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface;

$app->get('/excel', function (ServerRequestInterface $request, ResponseInterface $response) {
    $excel = new Spreadsheet();

    $sheet = $excel->setActiveSheetIndex(0);
    $cell = $sheet->getCell('A1');
    $cell->setValue('Test');
    $sheet->setSelectedCells('A1');
    $excel->setActiveSheetIndex(0);

    $excelWriter = new Xlsx($excel);

    // We have to create a real temp file here because the
    // save() method doesn't support in-memory streams.
    $tempFile = tempnam(File::sysGetTempDir(), 'phpxltmp');
    $tempFile = $tempFile ?: __DIR__ . '/temp.xlsx';
    $excelWriter->save($tempFile);

    // For Excel2007 and above .xlsx files   
    $response = $response->withHeader('Content-Type', 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet');
    $response = $response->withHeader('Content-Disposition', 'attachment; filename="file.xlsx"');

    $stream = fopen($tempFile, 'r+');

    return $response->withBody(new \Slim\Http\Stream($stream));
});
```

If you want to return the entire content at once, you could use this workaround:

```php
$stream = fopen($tempFile, 'r+');

$response->getBody()->write(fread($stream, (int)fstat($stream)['size']));

return $response;
```

Then open the url: http://localhost/excel and the download should start automatically.

## Downloading CSV files

Creating an CSV file is much simpler.

```php
use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface;

$app->get('/csv', function (ServerRequestInterface $request, ResponseInterface $response) {
    $list = array(
        array('aaa', 'bbb', 'ccc', 'dddd'),
        array('123', '456', '789'),
        array('"aaa"', '"bbb"')
    );

    $stream = fopen('php://memory', 'w+');

    foreach ($list as $fields) {
        fputcsv($stream, $fields, ';');
    }
    
    rewind($stream);

    $response = $response->withHeader('Content-Type', 'text/csv');
    $response = $response->withHeader('Content-Disposition', 'attachment; filename="file.csv"');

    return $response->withBody(new \Slim\Http\Stream($stream));
});
```

## Generating images

Example:

```php
use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface;

$app->get('/image', function (ServerRequestInterface $request, ResponseInterface $response) {
    $image = imagecreate(200, 80);
    imagecolorallocate($image, 255, 255, 255);
    $textColor = imagecolorallocate($image, 113, 158, 64);
    $lineColor = imagecolorallocate($image, 170, 170, 170);
    imagestring($image, 8, 35, 25, 'slim framework', $textColor);
    imagesetthickness($image, 2);
    imageline($image, 35, 45, 160, 45, $lineColor);

    ob_start();
    imagepng($image);
    $image = ob_get_clean();

    $response->write($image);

    return $response->withHeader('Content-Type', 'image/png');
});
```
