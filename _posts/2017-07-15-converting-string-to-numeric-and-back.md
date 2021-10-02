---
title: Converting string to numeric and back
layout: post
comments: true
published: false
description: 
keywords: 
---

### Code

```php
function numeric_encode($string)
{
    $result = '';
    $length = strlen($string);
    for ($i = 0; $i < $length; $i++) {
        $result .= sprintf('%03d', ord($string[$i]));
    }
    return $result;
}

function numeric_decode($data)
{
    $result = '';
    $length = strlen($data);
    for ($i = 0; $i < $length; $i = $i + 3) {
        $result .= chr((int)substr($data, $i, 3));
    }
    return $result;
}
```

## Usage

```php
$encoded = numeric_encode('abc123');
$decoded = numeric_decode($encoded);
echo $encoded . "\n"; // 097098099049050051
echo $decoded . "\n"; // abc123
```

[Demo](https://3v4l.org/N6op0)