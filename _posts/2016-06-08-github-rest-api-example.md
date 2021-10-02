---
title: Github REST API example
layout: post
comments: true
published: false
description: 
keywords: 
---

```php
// Github REST API example
function github_request($url)
{
    $ch = curl_init();
    
    // Basic Authentication with token
    // https://developer.github.com/v3/auth/
    // https://github.com/blog/1509-personal-api-tokens
    // https://github.com/settings/tokens
    $access = 'username:token';
    
    curl_setopt($ch, CURLOPT_URL, $url);
    //curl_setopt($ch, CURLOPT_HTTPHEADER, array('Content-Type: application/xml'));
    curl_setopt($ch, CURLOPT_USERAGENT, 'Agent smith');
    curl_setopt($ch, CURLOPT_HEADER, 0);
    curl_setopt($ch, CURLOPT_USERPWD, $access);
    curl_setopt($ch, CURLOPT_TIMEOUT, 30);
    curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
    curl_setopt($ch, CURLOPT_SSL_VERIFYHOST, 0);
    curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, 0);
    $output = curl_exec($ch);
    curl_close($ch);
    $result = json_decode(trim($output), true);
    
    return $result;
}
```

### Usage

```php
$repos = github_request('https://api.github.com/user/repos?page=1&per_page=100');
print_r($repos);

//$events = github_request('https://api.github.com/users/:username/events?page=1&per_page=5');
$events = github_request('https://api.github.com/users/:username/events/public?page=1&per_page=5');
print_r($events);

$feeds = github_request('https://api.github.com/feeds/:username?page=1&per_page=5');
print_r($feeds);
```
