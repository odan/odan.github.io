---
title: Slim Framework - Emitting chunked stream responses
layout: post
comments: true
published: false
description: 
keywords: 
---

What is Chunked transfer encoding?

> Chunked transfer encoding is a streaming data transfer mechanism available in version 1.1 of the 
> Hypertext Transfer Protocol (HTTP). In chunked transfer encoding, 
> the data stream is divided into a series of non-overlapping "chunks". 
> The chunks are sent out and received independently of one another. 
> No knowledge of the data stream outside the currently-being-processed 
> chunk is necessary for both the sender and the receiver at any given time.

[Read more](https://en.wikipedia.org/wiki/Chunked_transfer_encoding)

Most PSR-7 implementations use [streams](http://php.net/manual/en/book.stream.php). 
Since the stream is held in memory, problems can occur if the requests/responses are too large. 
Depending on memory limit and configuration, the limit may vary.

The `responseChunkSize` settings is used as number of bytes to read from body until it reaches the end of file.
If content length is known and it is less or equal than `responseChunkSize` (Default: 4096), 
then it only takes one iteration to read body's content.

Here you can see a Slim route with a callback that reads a ZIP file into the output buffer and flushes it directly into the response.

```php
use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface;

$app->get('/download', function (ServerRequestInterface $request, ResponseInterface $response) {
    $stream = fopen('file.zip', 'r+');
    
    return $response->withBody(new \Slim\Http\Stream($stream));
});
```

## Reading a chunked response

By reading and output response in smaller chunk, browser does not wait too long to get first byte. Reading big chunk is slower and may require larger memory consumption so browser will likely get first byte longer than smaller chunk.

You could use Guzzle to read bytes of the (response) stream until the end of the stream is reached

http://docs.guzzlephp.org/en/latest/request-options.html#stream

```php
$client = new \GuzzleHttp\Client();

$response = $client->request('GET', '/stream/20', ['stream' => true]);

// Read bytes off of the stream until the end of the stream is reached
$body = $response->getBody();
while (!$body->eof()) {
    echo $body->read(4096);
}
```