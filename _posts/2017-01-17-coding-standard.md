---
title: PHP, JS, CSS Coding Standards
layout: post
comments: true
published: false
description: 
keywords: 
---

## Filenames

* Filenames are case sensitive
* Allowed charset: a-zA-Z0-9-.
* Spaces are not allowed

## PHP

### Coding styles

Follow PSR-1 and PSR-2 Coding Standards.

* [PSR-1 Basic Coding Standard](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-1-basic-coding-standard.md)
* [PSR-2 Coding Style Guide](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-2-coding-style-guide.md)

## Filenames

* Extensions: .php
* Filename: PascalCase.php
* One class per file

### Class names

* Singular
* ClassName + "Controller"
* ModelName
*  "Abstract" + AbstractName
* InterfaceName + "Interface"

### Class method names

* camelCase
* Underscore is not allowed

### Global function names

* lowercase_under_score

### Method visibility

* Class methods must defined as public or protected

```php
public function getValue()
{
    // ...
}
```

### Variable names

* camelCase
* No data type prefix
* Arrays with a collection of data should be named plural
  
```php
<?php
$userName = 'admin';
$count = 100;
$amount = 123.45;
$i = 0;
$status = true;
$rows = array();
$customer = new Customer();
$db = $this->getDb();
```

### Special variable names

```php
<?php
$result            // Mixed return value
  
// Parts of file name and path
$splFile = new \SplFileInfo('/tmp/file.txt');

// Check if file or path exists
$existsPath = $splFile->getRealPath() !== false;

// Full path, filename and extension (e.g. /tmp/file.txt)
$file = $splFile->getRealPath() ?: $splFile->getPathname();

// Filename with extension (e.g. file.txt), or the last directory name
$fileName = $splFile->getFilename();

// Filename extension without dot (e.g. txt)
$extension = $splFile->getExtension();

// Filename without extension (e.g. file) or the last directory name
$baseFileName = pathinfo($splFile->getFilename(), PATHINFO_FILENAME);

// Full path to file (/tmp)
$filePath = pathinfo($splFile->getRealPath() ?: $splFile->getPathname(), PATHINFO_DIRNAME);

// Name of a single folder (e.g. tmp)
$directory = 'tmp';
```

### Array index names

* lower_snake_case
* singular by default
* plural if content is a list of multiple values

### Comments

* Use [PHPDoc](https://www.phpdoc.org/docs/latest/getting-started/your-first-set-of-documentation.html)

## HTML

* Google HTML&JS Style Guide: https://google.github.io/styleguide/htmlcssguide.html

### Filenames

* Extensions: .html.php or .twig
* Name: lower-kebab-case.html.php, my-fancy-page.twig

## JavaScript

### Filenames

* Extension: .js
* Use [jQuery style](http://stackoverflow.com/a/7273431): lower-kebab-case.js
* Examples: bootstrap.js, jquery.min.js, jquery-3.2.1.min.js, foo-library.bar.js

### Documentation

* Use [JSDoc](http://usejsdoc.org/) 
* [JSDoc Wiki](https://en.wikipedia.org/wiki/JSDoc)

### Variables

* camelCase
* No data type prefix
* Arrays index names should be plural
* You should use `const` in all cases except when the variable would be reassigned new values. At such cases use `let` instead.

```js
const userName = 'admin';
const total = 100;
const amount = 29.99;
const visible = true;
const config = {}
const rows = [];
const value = 'something';
```

### Element names

* Lowercase for object and array element names
* no prefix

```js
// good
let row = {};
row.id = 100;
row.username = 'admin';

// good
const row = {
    id: 100,
    username: 'admin'
};

// bad
var row = [];
row.Id = 100;
row.UserName = 'admin';
```

## CSS

### Filenames

* Extension: .css
* Format: lower-kebab-case.css
* Examples: bootstrap-theme.css, jquery-ui.min.css

### Names

* Class name: lower-kebab-case

```css
.my-super-class-name {}
```

* ID name: lower-kebab-case

```css
#email
#field-with-long-id
#my-modal
#anchor-name
```

## Vue.js

* Extension: .vue
* [Style Guide](https://vuejs.org/v2/style-guide/)
