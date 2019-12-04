---
title: Download link to the latest PHPStan PHAR file
layout: post
comments: true
published: true
description: 
keywords: php, phpstan
---

With PHPStan 0.12, the phpstan/phpstan-shim composer package will not be updated anymore. 
[Read more](https://medium.com/@ondrejmirtes/phpstan-0-12-released-f1a88036535d)

But you still can download the latest phpstan PHAR file diretly from Github.
 
The url ist: <https://github.com/phpstan/phpstan-shim/raw/master/phpstan.phar>

## Composer

Add the following scripts to your `composer.json` file: 

```json
"scripts": {
    "install-phpstan": "php -r \"@mkdir('build'); copy('https://github.com/phpstan/phpstan-shim/raw/master/phpstan.phar', 'build/phpstan.phar');\"",
    "phpstan": "php build/phpstan.phar analyse src --level=max --no-progress",
},
```

**Installation**

```
composer install-phpstan
```

**Usage**

```
composer phpstan
```

## Apache Ant

Add this to your `build.xml` file:

```xml
<target name="phpstan" description="PHP Static Analysis Tool - discover bugs in your code without running it">
    <mkdir dir="${basedir}/build"/>
    <exec executable="php" searchpath="true" resolveexecutable="true" failonerror="true">
        <arg value="${basedir}/vendor/phpstan/phpstan-shim/phpstan.phar"/>
        <arg value="analyse"/>
        <arg value="-l"/>
        <arg value="max"/>
        <arg value="src"/>
        <arg value="tests"/>
        <arg value="--no-interaction"/>
        <arg value="--no-progress"/>
    </exec>
</target>
```

Then run `ant phpstan`.


