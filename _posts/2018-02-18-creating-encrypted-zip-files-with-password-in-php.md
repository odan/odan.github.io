---
title: Creating encrypted ZIP files with password in PHP
layout: post
comments: true
published: false
description: 
keywords: 
---

You need at least PHP 7.2 to encrypt ZIP files with a password.

```php
<?php

$zip = new ZipArchive();

$zipFile = __DIR__ . '/output.zip';
if (file_exists($zipFile)) {
    unlink($zipFile);
}

$zipStatus = $zip->open($zipFile, ZipArchive::CREATE);
if ($zipStatus !== true) {
    throw new RuntimeException(sprintf('Failed to create zip archive. (Status code: %s)', $zipStatus));
}

$password = 'top-secret';
if (!$zip->setPassword($password)) {
    throw new RuntimeException('Set password failed');
}

// compress file
$fileName = __DIR__ . '/test.pdf';
$baseName = basename($fileName);
if (!$zip->addFile($fileName, $baseName)) {
    throw new RuntimeException(sprintf('Add file failed: %s', $fileName));
}

// encrypt the file with AES-256
if (!$zip->setEncryptionName($baseName, ZipArchive::EM_AES_256)) {
    throw new RuntimeException(sprintf('Set encryption failed: %s', $baseName));
}

$zip->close();
```