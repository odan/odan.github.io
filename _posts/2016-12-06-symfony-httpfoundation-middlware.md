---
title: Symfony HttpFoundation Middlware
layout: post
comments: true
published: false
description: 
keywords: 
---

Der Aufbau einer typischen Symfony Middleware ist fast identisch mit einer PSR-7 middleware. 
Der einzige Unterschied ist das Symfony Request- und Response Objekt (siehe Namespace).

Eine Symfony-Middleware muss folgende Signatur besitzen:

```php
function (
    Request $request, // the request
    Response $response, // the response
    callable $next // the next callback
) {
 // ...
}
```

## Middleware (Beispiel)

```php
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;

class MyMiddleware
{

   /**
    * Middleware.
    *
    * @param Request $request The request.
    * @param Response $response The response.
    * @param callable $next Callback to invoke the next middleware.
    * @return Response A response
    */
    public function __invoke(Request $request, Response $response, callable $next)
    {
        // do some fance stuff
        $response->setContent('Hello Middleware World!');

        // Call next calback and return response object
        return $next($request, $response);
    }
}
```

Als Nächstes benötigen wir einen Middleware-Dispacher der unsere callback-Funktionen ausführen soll.

```php
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;

/**
 * Middleware dispatcher.
 *
 * @param Request $request Request objct
 * @param Response $response Response object
 * @param array $queue Array with middleware
 * @return Response response
 */
function relay(Request $request, Response $response, array $queue)
{
    $runner = function (Request $request, Response $response) use(&$queue, &$runner) {
        $middleware = array_shift($queue);
        if ($middleware) {
            return $middleware($request, $response, $runner);
        }
        return $response;
    };
    return $runner($request, $response);
}
```

In einer Queue definieren wir eine Liste von Middleware callables.

```php
// Create a queue array of middleware callables
$queue = [];
$queue[] = new MyMiddleware();
// add more...
```

Zum Schluss lassen wir die Queue durch den Middleware-Runner laufen und geben den Response aus.

```php
$request = Request::createFromGlobals();
$response = new Response('', 200);
$response = relay($request, $response, $queue);
$response->send();
```
## Zusammenfassung

Dieses Konzept hat gezeigt, dass man auch mittels Symfony Komponenten einen
Middleware-Stack implementieren kann. 

Aus historischen Gründen wird Symfony wohl "niemals" auf PSR-7 und PSR-15 umstellen können.

Man sollte daher das hier vorgestellte Konzept nicht produktiv einsetzen, 
da diese Middleware konzeptionell nicht zum Symfony Kernel Event-Konzept passt. 

Bei Fragen und Anregungen bitte ein Kommentar hinterlassen.

## Links

* [https://inviqa.com/blog/introduction-psr-7-symfony](https://inviqa.com/blog/introduction-psr-7-symfony)
* [Using PSR-7 in Symfony](https://dunglas.fr/2015/06/using-psr-7-in-symfony/)
* <https://twitter.com/beberlei/status/1063873880677326853>
* <https://github.com/QafooLabs/QafooLabsNoFrameworkBundle/pull/28>
* <https://github.com/php-fig/fig-standards/pull/1120>
* <https://twitter.com/fabpot/status/1064946908605685760>
* <https://twitter.com/taylorotwell/status/1064958919842381827>
