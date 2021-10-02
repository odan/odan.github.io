---
title: Download all Dilbert comics from 1990 until today
layout: post
comments: true
published: false
description: 
keywords: 
---

```php
set_time_limit(3600 * 6);

$begin = new DateTime('1990-01-01');
$end = new DateTime(date('Y-m-d'));
$end = $end->modify('+1 day');
$interval = new DateInterval('P1D');
$daterange = new DatePeriod($begin, $interval, $end);

foreach ($daterange as $date) {
    $dateYmd = $date->format("Y-m-d");
    $url = sprintf("http://dilbert.com/strip/%s", $dateYmd);
    $html = file_get_contents($url);
    preg_match_all('/http:\/\/assets\.amuniversal\.com\/[a-z0-9A-Z]{32}/', $html, $match);
    if (!isset($match[0][0])) {
        continue;
    }
    
    $data = file_get_contents($match[0][0]);
    $fileName = sprintf('%s/%s.jpg', __DIR__, $dateYmd);
    file_put_contents($fileName, $data);
    echo $fileName . "<br>";
}
```
