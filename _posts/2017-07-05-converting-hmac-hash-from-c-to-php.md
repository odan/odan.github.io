---
title: Converting HMAC Hash from C# to PHP
layout: post
comments: true
published: false
description: 
keywords: 
---

With this code you can generate a Shared Access Signature to access an Azure SB Queue.

## C#

```csharp
string expiry = "1499177142";
string resourceUri = "http://nifi-eventhub.servicebus.windows.net/hub1";
string keyName = "hub-user";
string secretkey = "secret";

string stringToSign = HttpUtility.UrlEncode(resourceUri) + "\n" + expiry;

HMACSHA256 hmac = new HMACSHA256(Encoding.UTF8.GetBytes(secretkey));
byte[] hashBytes = hmac.ComputeHash(Encoding.UTF8.GetBytes(stringToSign));

var signature = Convert.ToBase64String(hashBytes);

// 7kS3apSDpJFTYI1vxuo4t7syGG3FTBYI8foamMOtrEE=
Console.WriteLine(signature);

var sasToken = String.Format(CultureInfo.InvariantCulture, "SharedAccessSignature sr={0}&sig={1}&se={2}&skn={3}",
HttpUtility.UrlEncode(resourceUri), HttpUtility.UrlEncode(signature), expiry, keyName);

Console.WriteLine(sasToken);
```

## PHP

```php
<?php

$resourceURI = "http://nifi-eventhub.servicebus.windows.net/hub1";
$keyName = "hub-user";
$key = "secret";
$expiry = '1499177142'; // timestamp

// The format for the string is <resourceURI> + \n + <expiry>    
$stringToSign = strtolower(rawurlencode($resourceURI)) . "\n" . $expiry;

// Hash the URL encoded string using the shared access key
$sig = hash_hmac("sha256", utf8_encode($stringToSign), utf8_encode($key), false);

// Convert hexadecimal string to binary and then to base64
$sig = hex2bin($sig);
$sig = base64_encode($sig);

// 7kS3apSDpJFTYI1vxuo4t7syGG3FTBYI8foamMOtrEE=
echo $sig . "<br>\n";

// Generate authorization token
$token = "SharedAccessSignature sr=" . urlencode($resourceURI) . "&sig=" . rawurlencode($sig) . "&se=" . $expiry . "&skn=" . $keyName;
echo $token . "<br>\n";

```

Source: [StackOverflow](https://stackoverflow.com/q/44908713/1461181)