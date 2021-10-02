---
title: Fetching RSS Feeds with PHP
layout: post
comments: true
published: false
description: 
keywords: php rss
---

```php
/**
 * Read RSS feed items.
 *
 * @param string $uri URI
 * @param string $type rss or atom
 * @return array
 */
function get_rss_items($uri, $type = 'rss')
{
    $xml = file_get_contents($uri);
    $xml = simplexml_load_string($xml);
    $result = array();
    if ($type == 'rss') {
        $items = $xml->xpath("/rss/channel/item");
        foreach ($items as $item) {
            $published = date("Y-m-d h:i:s", strtotime((string) $item->pubDate));
            $result[] = array(
                'title' => (string) $item->title,
                'published' => $published,
                'href' => (string) $item->link,
            );
        }
    }
    if ($type == 'atom') {
        $xml->registerXPathNamespace('xmlns', 'http://www.w3.org/2005/Atom');
        $items = $xml->xpath("/xmlns:feed/xmlns:entry");
        foreach ($items as $item) {
            $published = date("Y-m-d h:i:s", strtotime((string) $item->published));
            $result[] = array(
                'title' => (string) $item->title,
                'published' => $published,
                'href' => (string) $item->link->attributes()->href,
            );
        }
    }
    // Sort by date
    usort($result, function($a, $b) {
        return $a['published'] < $b['published'];
    });
    return $result;
}
```

### Usage

```php
$items = get_rss_items('https://news.ycombinator.com/rss');
print_r($items);
```
