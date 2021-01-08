---
title: Using htmlspecialchars correctly
layout: post
comments: false
published: true
description: 
keywords: php, xss
---

The first question is: When to use the [htmlspecialchars](https://php.net/manual/en/function.htmlspecialchars.php) function?

You use htmlspecialchars EVERY time you output content within HTML, 
so it is interpreted as content and not HTML.

If you allow content to be treated as HTML, you have just opened 
the door to bugs at a minimum, and total XSS hacks at worst.

Now to the next question: How to use htmlspecialchars()?

The problem is that htmlspecialchars is security relevant and at 
the same time difficult to use in the right (and safe) way.

Look at this strange list of parameters:

```php
string htmlspecialchars ( string $string [, int $flags = ENT_COMPAT | ENT_HTML401 [, string $encoding = ini_get("default_charset") [, bool $double_encode = TRUE ]]] )
```

The parameter `$flags` is a bitmask and can be combined with at least 10 different flags.

Let's play with this function to find the most secure flags:

### 1. Test: Using the default flags 

```php
echo htmlspecialchars('&"\'<> ', ENT_HTML401 | ENT_COMPAT, 'UTF-8')
```

This result is not secure because the `'` is not encoded to html.

```
&amp;&quot;'&lt;&gt; 
```

### 2. Test: Using the `ENT_HTML5` flag

```php
echo htmlspecialchars('&"\'<> ', ENT_HTML5, 'UTF-8')
```

This result is not secure because `"` and `'` is not encoded to html.

```
&amp;"'&lt;&gt; 
```


### 3. Test: Using the `ENT_QUOTES | ENT_SUBSTITUTE` flags

```php
echo htmlspecialchars('&"\'<> ', ENT_QUOTES | ENT_SUBSTITUTE, 'UTF-8')
```

This result looks perfect:

```
&amp;&quot;&#039;&lt;&gt; 
```

[Online Demo](https://3v4l.org/PvRtm)

## Conclusion

The most secure flags for htmlspecialchars are: `ENT_QUOTES | ENT_SUBSTITUTE` in combination with `UTF-8` encoding.

## Helper function

In real life, of course, you don't use htmlspecialchars that way because it's too verbose.

So here is a tiny template helper function for htmlspecialchars.

```php
/**
 * Convert all applicable characters to HTML entities.
 *
 * @param string $text The string being converted.
 *
 * @return string The converted string.
 */
function html($text)
{
    return htmlspecialchars($text, ENT_QUOTES | ENT_SUBSTITUTE, 'UTF-8');
}
```