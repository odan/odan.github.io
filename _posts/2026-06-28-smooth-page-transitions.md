---
title: Smooth page transitions without flickering
layout: post
comments: true
published: true
description: How to add smooth, SPA-like page transitions to a static site using Hotwire
keywords: cline, ai, agent, coding, deepseek, mistral, openai
---

**TL;DR:** Four lines of HTML give you instant page navigation with smooth crossfade animations in Chrome. No build tools, no JavaScript framework, no CSS.

---

Static sites are fast to serve but feel slow to navigate. Every link click triggers a full page reload - white flash, especially in Google Chrome.

I wanted something closer to a single-page application without adding a JavaScript framework, a build pipeline, or a client-side router. The solution was four HTML tags.

### The Setup

Add these to the `<head>` of your layout:

{% raw %}
```html
<script type="module"
    src="https://cdn.jsdelivr.net/npm/@hotwired/turbo@latest/dist/turbo.es2017-esm.min.js"></script>
<meta name="view-transition" content="same-origin">
<meta name="turbo-refresh-method" content="morph">
<meta name="turbo-refresh-scroll" content="preserve">
```
{% endraw %}

That's it. No npm, no build step, no JavaScript to write.

### How it works

**Turbo Drive** intercepts every same-origin link click, fetches the new page in the background, and swaps the `<body>` in place. No full page reload ever happens — no white flash.

**Morphing** takes this further. Instead of replacing the entire body, Turbo diffs the old and new DOM and only updates the elements that actually changed.

**View Transitions** tell Turbo to wrap the body swap in `document.startViewTransition()`. In Chrome and Edge, the browser responds with a smooth crossfade between the old and new page — the browser's own compositor handles it, no CSS animations needed.

**Scroll preservation** restores the scroll position after every navigation. Back and forward work instantly from cache.

### Browser support

| Browser | Navigation | Animation |
|---|---|---|
| Chrome / Edge 125+ | Instant body swap | Native crossfade |
| Firefox | Instant body swap | None |
| Safari | Instant body swap | None |

### Prefetching (Optional)

Turbo can preload pages on hover, making navigation feel instant. I disabled it globally and enabled it only for the main navigation:

{% raw %}
```html
<meta name="turbo-prefetch" content="false">

<nav data-turbo-prefetch="true">
    <a href="/">Home</a>
    <a href="/about.html">About</a>
</nav>
```
{% endraw %}

Links start loading as soon as the user hovers. By the time they click, the page is already cached and renders instantly.

### Summary

If you're running a static site and want it to feel faster, this could be one of the easiest solution you will get.
