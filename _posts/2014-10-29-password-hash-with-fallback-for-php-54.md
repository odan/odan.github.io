---
title: Password hash with fallback for PHP <= 5.4
layout: post
comments: true
published: false
description: 
keywords: 
---

When you need to hash a password, just feed it to the function
and it will return the hash which you can store in your database. 
The important thing here is that you don’t have to provide a salt 
value or a cost parameter. The new API will take care of all of 
that for you. And the salt is part of the hash, so you don’t 
have to store it separately.
 
Here is a implementation for PHP 5.5 and older:

```php
<?php

function create_password_hash($password, $algo = 1, $options = array())
{
    if (function_exists('password_hash')) {
        // php >= 5.5
        $hash = password_hash($password, $algo, $options);
    } else {
        $salt = mcrypt_create_iv(22, MCRYPT_DEV_URANDOM);
        $salt = base64_encode($salt);
        $salt = str_replace('+', '.', $salt);
        $hash = crypt($password, '$2y$10$' . $salt . '$');
    }
    return $hash;
}

function verify_password_hash($password, $hash)
{
    if (function_exists('password_verify')) {
        // php >= 5.5
        return password_verify($password, $hash);
    } else {
        return $hash === crypt($password, $hash);
    }
}
```

### Usage

```php
<?php

// Test
$hash = create_password_hash('secret');
echo $hash . "\n";

if (verify_password_hash('secret', $strHash)) {
    echo 'Password is valid!';
} else {
    echo 'Invalid password.';
}
```

Links:

* https://www.sitepoint.com/hashing-passwords-php-5-5-password-hashing-api/
* https://stackoverflow.com/questions/536584/non-random-salt-for-password-hashes/536756#536756
