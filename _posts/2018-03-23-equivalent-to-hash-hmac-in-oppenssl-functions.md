---
title: Equivalent to hash_hmac in oppenssl_* functions
layout: post
comments: false
published: false
description: 
keywords: openssl, php, hmac, sha256
---

This function uses the openssl_* extension and would be an equivalent for: 

```php
hash_hmac('sha256', 'data', 'key');
```

## Code

```php
function openssl_hmac($algo, $data, $key, $raw_output = false)
{
    $algo = strtolower($algo);
    $pack = 'H' . strlen(openssl_digest('test', $algo));
    $size = 64;
    $opad = str_repeat(chr(0x5C), $size);
    $ipad = str_repeat(chr(0x36), $size);

    if (strlen($key) > $size) {
        $key = str_pad(pack($pack, $algo($key)), $size, chr(0x00));
    } else {
        $key = str_pad($key, $size, chr(0x00));
    }

    for ($i = 0; $i < strlen($key) - 1; $i++) {
        $opad[$i] = $opad[$i] ^ $key[$i];
        $ipad[$i] = $ipad[$i] ^ $key[$i];
    }

    $output = openssl_digest($opad . pack($pack, openssl_digest($ipad . $data, $algo)), $algo);

    return ($raw_output) ? pack($pack, $output) : $output;
}
```

## Usage

```php
echo openssl_hmac('sha256', 'data', 'key', false);
```

Result: 

```
5031fe3d989c6d1537a013fa6e739da23463fdaec3b70137d828e36ace221bd0
```

The result is the same as when using the `hash_hmac` function:

```
echo hash_hmac('sha256', 'data', 'key');
```

Result:

```
5031fe3d989c6d1537a013fa6e739da23463fdaec3b70137d828e36ace221bd0
```