---
title: Signing Github commits using GPG
date: 2018-03-13
layout: post
comments: true
published: false
description: 
keywords: 
---

Github offers a nice feature to verify your commits. 

## Requirements

* Windows
* Git
* Gpg4win

## Installation

* Download and install [Gpg4win](https://www.gpg4win.org/)

## Generating a new GPG key

If you don't have an existing GPG key, you can generate a new GPG key to use for signing commits and tags.

* Open the console `cmd` and enter

```cmd
> gpg --full-generate-key
```

At the prompt, specify the kind of key you want, or press Enter to accept the default **RSA and RSA**.

@todo

## Associating your key with your GitHub account

Paste the key here:

https://github.com/settings/keys


## PHPStorm

@todo

## Result

Your commits will show as verified within a pull request on GitHub.

![image](https://user-images.githubusercontent.com/781074/37349116-fce920ea-26d5-11e8-94ad-5ec85f517318.png)