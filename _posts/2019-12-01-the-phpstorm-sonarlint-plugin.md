---
title: PhpStorm - SonarLint Plugin
layout: post
comments: true
published: true
description:
keywords: PhpStorm php sonarlint phpstan
---

I'm sure you've heard of [phpstan](https://github.com/phpstan/phpstan), [psalm](https://github.com/vimeo/psalm), and the
[PhpStorm Php Inspections (EA Extended)](https://plugins.jetbrains.com/plugin/7622-php-inspections-ea-extended-) plugin. 
Maybe you're already using one of these great tools to scan your codebase for issues
and possible bugs. But this time I want to show you another very cool 
static code analyzer for PHP: 

The **[SonarLint Plugin](https://plugins.jetbrains.com/plugin/7973-sonarlint)**
for PhpStorm.

SonarLint is an IDE extension that helps you detect and fix quality issues as you 
write code. Like a spell checker, SonarLint squiggles flaws so they can be fixed 
before committing code. You can get it directly from the IntelliJ IDEA Plugin Repository, 
and it will then detect new bugs and quality issues as you code (PHP, Java, Kotlin, Ruby, 
JavaScript and Python).

If your project is analyzed on [SonarQube](https://www.sonarqube.org/) or on SonarCloud, SonarLint 
can connect to the server to retrieve the appropriate quality profiles 
and settings for that project.

## Installation

* Open PhpStorm > File > Settings > Plugins
* Type "SonarLint" to search for the plugin
* Click: Install
* Restart PhpStorm

## Configuration

* Open the `SonarLint` tab (to the right of the `Version Control` tab)
* Open the `Report` tab
* Click the `Configure SonarLint` button
* Open the `File Exclusions` tab and exclude the `vendor/` directory of your project.

## Usage

To start the code analysis...

* Open the `SonarLint` > `Report` tab
* Click the `Analyze all Project files` button 

As soon as the scan is completed, you should see the result of the code inspection:

![image](https://user-images.githubusercontent.com/781074/69918552-9d3c7100-1473-11ea-8336-786e6c20bdda.png)

