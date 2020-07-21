---
title: Replacing mcrypt_encrypt with openssl_encrypt
layout: post
comments: true
published: false
description: 
keywords: php, mcrypt, openssl, encryption
---

**Attention:** This article is from 2017, some information may be out of date.

> Note: This might help too:
> <https://packagist.org/packages/phpseclib/mcrypt_compat>

When you switch from PHP 5.x to 7.1, you may get the following error message:

> Function mcrypt_encrypt() is deprecated

Now it is the right time to refactor this old function :-)

```php
function encrypt(string $data, string $key, string $method): string
{
    $ivSize = openssl_cipher_iv_length($method);
    $iv = openssl_random_pseudo_bytes($ivSize);

    $encrypted = openssl_encrypt($data, $method, $key, OPENSSL_RAW_DATA, $iv);
    
    // For storage/transmission, we simply concatenate the IV and cipher text
    $encrypted = base64_encode($iv . $encrypted);

    return $encrypted;
}

function decrypt(string $data, string $key, string $method): string
{
    $data = base64_decode($data);
    $ivSize = openssl_cipher_iv_length($method);
    $iv = substr($data, 0, $ivSize);
    $data = openssl_decrypt(substr($data, $ivSize), $method, $key, OPENSSL_RAW_DATA, $iv);

    return $data;
}
```

## Usage

```php
$data = 'plain text or binary data';

// ECB encrypts each block of data independently and 
// the same plaintext block will result in the same ciphertext block.
//$method = 'AES-256-ECB';

// CBC has an IV and thus needs randomness every time a message is encrypted
$method = 'AES-256-CBC';

// simple password hash
$password = 'secret-password-as-string';
$key = hash('sha256', $password);

// Most secure
// You must store this secret random key in a safe place of your system.
//$key = openssl_random_pseudo_bytes(openssl_cipher_iv_length($method));

echo "Method: " . $method . "\n";
$encrypted = encrypt($data, $key, $method);
echo "Encrypted: ". $encrypted . "\n";
$decrypted = decrypt($encrypted, $key, $method);
echo "Decrypted: ".  $decrypted . "\n"; // plain text
```


To get a list of all openssl_encrypt methods use:

```php
print_r(openssl_get_cipher_methods());
```

## Links

* <https://stackoverflow.com/questions/1220751/how-to-choose-an-aes-encryption-mode-cbc-ecb-ctr-ocb-cfb>
