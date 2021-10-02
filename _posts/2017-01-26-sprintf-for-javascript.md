---
title: sprintf for JavaScript
layout: post
comments: true
published: false
description: 
keywords: 
---

The code:

```js
/**
 * Returns a formatted string using the first argument as a printf-like format.
 *
 * The first argument is a string that contains zero or more placeholders.
 * Each placeholder is replaced with the converted value from its corresponding argument.
 *
 * Supported placeholders are:
 *
 * %s - String.
 * %d - Number (both integer and float).
 * %% - single percent sign ('%'). This does not consume an argument.
 *
 * Argument swapping:
 *
 * %1$s ... %n$s
 *
 * When using argument swapping, the n$ position specifier must come immediately
 * after the percent sign (%), before any other specifiers, as shown in the example below.
 *
 * If the placeholder does not have a corresponding argument, the placeholder is not replaced.
 *
 * @author odan
 *
 * @param {...*} format [, args [, ...*]
 * @returns {String}
 */
function sprintf() {
    if (arguments.length < 2) {
        return arguments[0];
    }
    var args = arguments;
    var index = 1;
    var result = (args[0] + '').replace(/%((\d)\$)?([sd%])/g, function (match, group, pos, format) {
        if (match === '%%') {
            return '%';
        }
        if (typeof pos === 'undefined') {
            pos = index++;
        }
        if (pos in args && pos > 0) {
            return args[pos];
        } else {
            return match;
        }
    });
    return result;
}
```

### Other libraries

* <http://www.diveintojavascript.com/projects/javascript-sprintf>
