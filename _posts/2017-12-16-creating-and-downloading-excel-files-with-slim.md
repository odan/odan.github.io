---
title: Creating and downloading Excel files in Slim 3
layout: post
comments: true
published: true
description: 
keywords: 
---

Maybe you need the possibility to create excel files and download it automatically.

To be able to create Excel files I'm using [PhpSpreadsheet](https://github.com/PHPOffice/PhpSpreadsheet). 

Here is a tiny example how to create and download the created file directly from your server.

## Installation

```bash
composer require phpoffice/phpspreadsheet
```

## Action

Create a new route:

```php
<?php
use PhpOffice\PhpSpreadsheet\Spreadsheet;
use PhpOffice\PhpSpreadsheet\Writer\Xlsx;
use Slim\Http\Request;
use Slim\Http\Response;

$app->get('/excel', function (Request $request, Response $response) {
    $excel = new Spreadsheet();

    $sheet = $excel->setActiveSheetIndex(0);
    $cell = $sheet->getCell('A1');
    $cell->setValue('Test');
    $sheet->setSelectedCells('A1');
    $excel->setActiveSheetIndex(0);

    $excelWriter = new Xlsx($excel);

    $excelFileName = __DIR__ . '/file.xlsx';
    $excelWriter->save($excelFileName);

    // For Excel2007 and above .xlsx files   
    $response = $response->withHeader('Content-Type', 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet');
    $response = $response->withHeader('Content-Disposition', 'attachment; filename="file.xlsx"');

    $stream = fopen($excelFileName, 'r+');

    return $response->withBody(new \Slim\Http\Stream($stream));
});
```

Then open the url: http://localhost/excel and the download should start automatically.

## Downloading CSV files

Creating an CSV file is even simpler.

```php
use Slim\Http\Request;
use Slim\Http\Response;

$app->get('/csv', function (Request $request, Response $response) {
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
use Slim\Http\Request;
use Slim\Http\Response;

$app->get('/image', function (Request $request, Response $response) {
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
