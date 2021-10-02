---
title: Remove directory recursively in PHP
layout: post
comments: true
published: false
description: 
keywords: php, vfsStream
---

> Update: Better use the `remove` method of the 
> [symfony/filesystem](https://symfony.com/doc/current/components/filesystem.html#remove) component.

The code:

```php
/**
 * Remove directory recursively.
 * This function is compatible with vfsStream.
 *
 * @param string $path The path
 * @return bool True on success or false on failure.
 */
function rrmdir($path) {
    $iterator = new DirectoryIterator($path);
    
    foreach ($iterator as $fileInfo) {
        if ($fileInfo->isDot() || !$fileInfo->isDir()) {
            continue;
        }
        rrmdir($fileInfo->getPathname());
    }
    $files = new FilesystemIterator($path);
    
    /* @var SplFileInfo $file */
    foreach ($files as $file) {
        unlink($file->getPathname());
    }
    
    return rmdir($path);
}
```

### Usage

```php
rrmdir('/my/path');
```

### Testing

[vfsStream](https://github.com/mikey179/vfsStream/wiki) is a stream wrapper for a virtual file 
system that may be helpful in unit tests to mock the real file system. 
It can be used with any unit test framework, like PHPUnit or SimpleTest.

```php
<?php
use org\bovigo\vfs\vfsStream;

//...

$root = vfsStream::setup('root');
vfsStream::newDirectory('tmp/dir1')->at($this->root);
vfsStream::newDirectory('tmp/dir2')->at($this->root);
vfsStream::newDirectory('tmp/dir2/dir2-1')->at($this->root);

$tmpPath = vfsStream::url('root/tmp');
rrmdir($tmpPath);

$this->assertFalse(is_dir($tmpPath));
$this->assertFalse(file_exists($tmpPath));
```

### Recursively copy a directory

```php
function copyr($src, $dst)
{
    if (is_dir($src)) {
        if (!file_exists($dst)) {
            mkdir($dst, 0777, true);
        }
        foreach (scandir($src) as $file) {
            if ($file == '.' || $file == '..') {
                continue;
            }
            copyr("$src/$file", "$dst/$file");
        }
    } elseif (is_file($src)) {
        copy($src, $dst);
    } else {
        throw new RuntimeException("Cannot copy $src (unknown file type)");
    }
}
```

### List all files recursively

```php
// change this
$path = __DIR__; 
$directory = new RecursiveDirectoryIterator($path, FilesystemIterator::SKIP_DOTS);
foreach (new RecursiveIteratorIterator($directory, RecursiveIteratorIterator::SELF_FIRST) as $item) {
    if(!$item->isFile()) {
        continue;
    }
    echo $item->getRealPath() . "\n";
}
```


