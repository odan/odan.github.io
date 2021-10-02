---
title: The PHP loop helper function
layout: post
comments: true
published: false
description: 
keywords: 
---

## Code

```php
function loop($items, callable $callback)
{
    if(empty($items)) {
        return;
    }
    foreach($items as $item) {
        if($callback($item) === false) {
            break;
        }
    }
}
```

## Usage

```php
$items = array(1, 2, 3, 4);

// Show all
loop($items, function($item) {
    echo $item . "\n"; 
});

echo "\n";

// Stop loop if item greater then 3
loop($items, function($item) {
    if($item > 3) {
        return false;
    }
    echo $item . "\n"; 
});
```

## Demo
* https://3v4l.org/7QqF5

## Keywords
* collections
* closure