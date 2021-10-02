---
title: JavaScript HTML Encoding
layout: post
comments: true
published: false
description: 
keywords: 
---

```js
/**
 * Convert all applicable characters to HTML entities
 * 
 * @param {string} str
 * @returns {string}
 */
function encodeHtml(str) {
    if(!str) return "";
    return str.replace(/[^a-z0-9A-Z ]/g, function (c) {
        return "&#" + c.charCodeAt() + ";";
    });
}
```
