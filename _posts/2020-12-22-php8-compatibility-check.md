---
title: PHP 8 Compatibility Check
layout: post
comments: true
published: true
keywords: php, php8, codesniffer
---

To check whether your code is compatible with PHP 8 or not, you may find this tool very useful.

The [PHP Compatibility Coding Standard for PHP CodeSniffer](https://github.com/PHPCompatibility/PHPCompatibility)
contains a set of sniffs for [PHP CodeSniffer](https://github.com/squizlabs/PHP_CodeSniffer) 
that checks for PHP cross-version compatibility. 
It will allow you to analyze your code for compatibility with higher and lower versions of PHP.

### Installation

If not already installed, install PHP CodeSniffer first:

```
composer require squizlabs/php_codesniffer --dev
```

Then install the "PHPCompatibility" rules using this command:

```
composer require phpcompatibility/php-compatibility --dev
```

In order to run the PHPCompatibility checker as native Composer command,
add the following [script](https://getcomposer.org/doc/articles/scripts.md#writing-custom-commands)
to your `composer.json` file:

```json
"sniffer:php8": "phpcs -p ./src --standard=vendor/phpcompatibility/php-compatibility/PHPCompatibility --runtime-set testVersion 8.0"
```

### Usage

To check your codebase for PHP 8 compatibility issues execute the script as follows:

```
composer sniffer:php8
```

Example output:

![image](https://user-images.githubusercontent.com/781074/102933727-181f6200-44a3-11eb-8028-a08d32d08ba0.png)

## Read more

* [Ask a question or report an issue](https://github.com/odan/slim4-tutorial/issues)
