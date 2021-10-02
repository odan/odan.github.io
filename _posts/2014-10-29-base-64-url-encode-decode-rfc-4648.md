---
title: Base 64 url encode/decode (RFC 4648)
layout: post
comments: true
published: false
description: 
keywords: php
---

## Code

```php

// Base 64 Encoding with URL and Filename Safe Alphabet.
// Details RFC 4648: http://tools.ietf.org/html/rfc4648#section-5
// License: MIT
function base64url_encode(string $data): string
{
    return str_replace('=', '', strtr(base64_encode($data), '+/', '-_'));
}

function base64url_decode(string $base64): string
{
     $base64 .= str_repeat('=', (4 - (strlen($base64) % 4)));
     
     return base64_decode(strtr($base64, '-_', '+/'));
}
```

## Usage

```php
echo base64url_encode('abc123ÿäüö');
echo "\n";
echo base64url_decode('YWJjMTIzw7_DpMO8w7Y');
```
