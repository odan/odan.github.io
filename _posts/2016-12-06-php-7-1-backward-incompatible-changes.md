---
title: PHP 7.1 Backward incompatible changes
layout: post
comments: true
published: false
description: 
keywords: 
---

## Operators
* <https://3v4l.org/kT6oL>
* <https://3v4l.org/1gJXa>
* <https://3v4l.org/RhCD1> / Division by zero depends on quotes in PHP 7
* <https://3v4l.org/EDcdY>

## Throw on passing too few function arguments
* <https://3v4l.org/8vRgI>

## Type casting of parameters
* <https://3v4l.org/is7tI>
* <https://3v4l.org/miT0X>
* <https://3v4l.org/VmWXa>
* <https://3v4l.org/qLv4E>

## Forbid dynamic calls to scope introspection functions
* <https://3v4l.org/AKRmm>

## Invalid class, interface, and trait names
* <https://3v4l.org/OjPma>
* <https://3v4l.org/QspG9>

## Disallow the ASCII delete control character in identifiers
* <https://3v4l.org/hoCMo>

## call_user_func() handling of reference arguments
* <https://3v4l.org/jH9GS>

## The empty index operator is not supported for strings anymore
* <https://3v4l.org/BaDqT>

## Removed ini directives
* <https://3v4l.org/mNce4>

## Array

* <https://3v4l.org/GcjRj> (array_walk + array_replace_recursive)
* <https://3v4l.org/K7bRf> / array_filter
* <https://3v4l.org/0BEEU> / key not exists!
* <https://3v4l.org/frbUc> / Array ordering
* <https://3v4l.org/2BivO> / print_r and var_dump of bool in array
* <https://3v4l.org/N5hXC>
* <https://3v4l.org/8VnKV >

## JSON encoding and decoding

* <https://3v4l.org/V0a4N>
* <https://3v4l.org/EQhFK>
* <https://3v4l.org/sJmL2>
* <https://3v4l.org/9EhJi>
* <https://3v4l.org/13Srm>
* <https://3v4l.org/vJWtM>
* <https://3v4l.org/ieOf9>
* <https://3v4l.org/aHt34>
* <https://3v4l.org/Tg6GB>

## [RFC](https://wiki.php.net/rfc/this_var): Fix inconsistent behavior of $this variable.

* <https://3v4l.org/seJHB>
* <https://3v4l.org/tWm1d>
* <https://3v4l.org/41PE7>
* <https://3v4l.org/ta4X5>
* <https://3v4l.org/RNDYn>
* <https://3v4l.org/rTT70>
* <https://3v4l.org/5IDt8>
* <https://3v4l.org/XNJSj>
* <https://3v4l.org/WWYAR>
* <https://3v4l.org/ibEDP>
* <https://3v4l.org/60SEn>

## Cast to float from string
* <https://3v4l.org/FPsXI>

## __clone
* <https://3v4l.org/J93ma>

## Session
* <https://3v4l.org/KcZIk>
* <https://3v4l.org/LM1ui>
* <https://3v4l.org/MbNuS>

## SimpleXML
* <https://3v4l.org/VDa0j>

## SQLite3
* <https://3v4l.org/GH75O>
* <https://3v4l.org/I4TpT>
* <https://3v4l.org/Vgtfc>

## Return value if var modified in finally
* <https://3v4l.org/MiLC0>

## Exceptions
* <https://3v4l.org/1WiBm>
* <https://3v4l.org/cNAQr>

## Previous property undefined in Exception after deserialization
* <https://3v4l.org/mWqHm>

## Nested try/finally blocks losing return value
* <https://3v4l.org/70VKP>

## Date time
DateTime constructor incorporates microseconds.

* <https://3v4l.org/tYK4A> (DateTime)
* <https://3v4l.org/P0GWf> (date_create)
* <https://3v4l.org/jGlVm> (microtime(true))
* <https://3v4l.org/D7bcM>
* <https://3v4l.org/aHuXd>
* <https://3v4l.org/vGsdM>
* <https://3v4l.org/e7EAp>
* <https://3v4l.org/XirOr>
* <https://3v4l.org/SopVG>
* <https://3v4l.org/P0GWf>
* <https://3v4l.org/6P8dS>
* <https://3v4l.org/ZCoaY>
* <https://3v4l.org/6lfO5>
* <https://3v4l.org/XQOQD>
* <https://3v4l.org/tgB0n>
* <https://3v4l.org/4GVsG>
* <https://3v4l.org/jf8qh> / date_diff
* <https://3v4l.org/HUAPL> / bug in date_parse_from_format
* <http://3v4l.org/RkMGa> / boolean fields are stored as integers.
* <https://3v4l.org/iTeSL> / DateTimeZone
* <https://3v4l.org/SJjVX> / DateInterval

## Apples + Oranges
* <https://3v4l.org/gDcfJ >
* <https://3v4l.org/5d4hR >

## Filter
* <https://3v4l.org/IagpD>
* <https://3v4l.org/qstrB>
* <https://3v4l.org/cEPvJ>

## iconv
* <https://3v4l.org/W2OSA>

## Intl
* <https://3v4l.org/IXhmu>
* <https://3v4l.org/jgpId>

## is_nan
* <https://3v4l.org/0Jeva>

## Class/Object related
* <https://3v4l.org/2qnbb>
* <https://3v4l.org/nkjOA>
* <https://3v4l.org/0gFa6>
* <https://3v4l.org/SqWNZ> / concatenation in member variable definition
* <https://3v4l.org/TKeLB> / Use After Free unserialize() PoC (serialize, unserialize)

## Reflection
* <https://3v4l.org/XoV5u>

# Strings
* <https://3v4l.org/jcDIo> / substr_count
* <https://3v4l.org/TOVhF> / mb_strlen

# General Issues
* <https://3v4l.org/AScS5> / unserialize
* <https://3v4l.org/aq3UZ> / bug in parse_url
* <https://3v4l.org/L0b92> / bug in parse_url
* <https://3v4l.org/GqOOI> / bug in parse_url

## Deprecated features in PHP 7.0.x

* <https://3v4l.org/YdX0V>
* <https://3v4l.org/5HCmW>
* <https://3v4l.org/H6C62>

## Strange fail after casting float to int (all versions)

* <https://3v4l.org/JdeUl>

## Funny things

* <https://3v4l.org/lUrBL>
* <https://3v4l.org/TAv2v>

## Source

* <http://php.net/manual/en/migration71.incompatible.php>