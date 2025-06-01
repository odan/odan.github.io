---
title: PHP - A Lightweight Dev Setup for Windows
layout: post
comments: true
published: true
description: 
keywords: php, slim, micro-framework, framework
---

I recently decided to remove my previous blog post about PHP development 
with WSL2. In theory, the approach looked promising, 
but in practice it proved to be impractical and too clunky.
Tools like Xdebug were difficult to get up and running reliably,
and the whole setup was too complex.

To keep things simple I put together a tiny 
Apache + PHP + MySQL setup script that runs natively on Windows
without the overhead of containers.

You can find it here: <https://github.com/odan/php-dev-box>

Maybe in the next blog post I'll take a look 
at scoop.sh (package manager) and nssm (non-sucking service manager) 
as another solution for PHP development.

If you have any good suggestions on how to install and use 
PHP with XDebug and PHPStorm, please write it into the comments.
