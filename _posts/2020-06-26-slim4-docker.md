---
title: Using Docker for Development
layout: post
comments: true
published: false
description: 
keywords: php, docker, slim
---

## Table of contents

* [Requirements](#requirements)
* [Introduction](#introduction)
* [Installation](#installation)
* [Configuration](#configuration)

## Requirements


## Introduction

Dev and prod containers should be similar as much as possible, it saves a lot of debugging
Having prod and dev very similar requires less effort in developing the dev images

Xdebug within dev container, but not in prod container.
Run two separate build targets for the container, 
then restart the container based on `docker-compose.override.yml`

How to set the vars on the machine instead? Just docker-compose.

## Installation

## Configuration

## Read more

* <https://twitter.com/goetas_asmir/status/1276389929443090432>