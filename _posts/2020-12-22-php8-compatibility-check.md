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
composer require phpcompatibility/php-compatibility @develop --dev
```

Add a custom [script](https://getcomposer.org/doc/articles/scripts.md#writing-custom-commands) in the correct place in your `composer.json` file. 
It should be added in the "scripts" section as follows:


```json
"scripts": {
    "sniffer:php8": "phpcs -p ./src --standard=vendor/phpcompatibility/php-compatibility/PHPCompatibility --runtime-set testVersion 8.2"
}
```

### Usage

To check your codebase for PHP 8.2 compatibility issues execute the script as follows:

```
composer sniffer:php8
```

**Example Output**

![image](https://user-images.githubusercontent.com/781074/102933727-181f6200-44a3-11eb-8028-a08d32d08ba0.png)

