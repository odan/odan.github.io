---
title: Implementing a SOAP API with PHP 7
layout: post
comments: true
published: false
description: 
keywords: 
---

## Table of contents

* [Intro](#intro)
* [Creating a SOAP Endpoint](#creating-a-soap-endpoint)
* [Creating a SOAP Client](#creating-a-soap-client)
* [Creating a SOAP Client with C#](#creating-a-soap-client-with-c)

## Intro

It's 2017/2018 and you're probably wondering why I'm blogging about this topic, aren't you? I'll tell you a secret: XML is not dead. No, it's more important and necessary than ever. 

> "XML is crap. Really. There are no excuses. XML is nasty to parse for humans, and it's a disaster to parse even for computers. There's just no reason for that horrible crap to exist." Linus Torvalds

Just because it comes from [Mr Torvalds does not necessarily make it right](http://www.kidstrythisathome.com/2014/03/why-linus-torvalds-is-wrong-about-xml/).

XML is great, because it is the first time that a good standard has been set up to exchange data in a structured, machine-readable and validatable manner with minimal issues.

JSON has its excellent use cases too, e.g. for browsers communicating via Ajax to the server.

But how to validate JSON payload against a schema? 

Conceptually, RESTful APIs where never designed in such a way that you need [JSON-Schema](https://json-schema.org/) for it. 

Instead of a schema, only examples are usually used.

In the last years it became more and more popular to use the (Swagger) [OpenAPI Specification](https://swagger.io/specification/), which defines a standard, language-agnostic interface to RESTful APIs which allows both humans and computers to discover and understand the capabilities of the API. Of cource there are hundreds of libs, and quite a few interpret things completely differently than others.

A SOAP API is better suited in a larger enterprise context to ensure standardized and strong data structures between server-to-server communications.

## Creating a SOAP Endpoint

The endpoint is the URL where your service can be accessed by a client application. To inspect the [WSDL](https://en.wikipedia.org/wiki/Web_Services_Description_Language) you just add `?wsdl` to the web service endpoint URL.

### Requiremnts

While in .NET it's very simple, in PHP you need some extra work to get a SOAP API working.

* PHP 7+
* Enable `extension=php_soap.dll` in your `php.ini` file
* [laminas-soap](https://docs.laminas.dev/laminas-soap/)

I chose the Lamina's SOAP library because it's compatible with .NET clients.

### Installation

Install the laminas-soap library:

```
composer require laminas/laminas-soap
```

Let's create a simple hello world webservice.

Create a php file `api.php` that acts as a SOAP endpoint.

```php
<?php

// api.php

require_once __DIR__ . '/vendor/autoload.php';

class Hello
{
    /**
     * Say hello.
     *
     * @param string $firstName
     * @return string $greetings
     */
    public function sayHello($firstName)
    {
        return 'Hello ' . $firstName;
    }

}

$serverUrl = "http://localhost/api.php";
$options = [
    'uri' => $serverUrl,
];
$server = new \Laminas\Soap\Server(null, $options);

if (isset($_GET['wsdl'])) {
    $soapAutoDiscover = new \Laminas\Soap\AutoDiscover(new \Laminas\Soap\Wsdl\ComplexTypeStrategy\ArrayOfTypeSequence());
    $soapAutoDiscover->setBindingStyle(array('style' => 'document'));
    $soapAutoDiscover->setOperationBodyStyle(array('use' => 'literal'));
    $soapAutoDiscover->setClass('Hello');
    $soapAutoDiscover->setUri($serverUrl);
    
    header("Content-Type: text/xml");
    echo $soapAutoDiscover->generate()->toXml();
} else {
    $soap = new \Laminas\Soap\Server($serverUrl . '?wsdl');
    $soap->setObject(new \Laminas\Soap\Server\DocumentLiteralWrapper(new Hello()));
    $soap->handle();
}
```

Open the browser to check the WSDL file: http://localhost/api.php?wsdl

## Creating a SOAP Client

Create a new file `client.php`. 

File content:

```php
<?php

require_once __DIR__ . '/vendor/autoload.php';

$client = new Laminas\Soap\Client('http://localhost/api.php?wsdl');
$result = $client->sayHello(['firstName' => 'World']);

echo $result->sayHelloResult;
```

The result (response) should look like this:

```
Hello World
```

As result you will receive a `stdClass` instance:

```php
var_dump($result);
```

```php
class stdClass#4 (1) {
  public $sayHelloResult =>
  string(11) "Hello World"
}
```

It is possible that your PHP static analysis tool (e.g. phpstan) has a problem with calling `$client->sayHello(...)`. 
The reason is that this method does not exist and is called internally via a magic method. 
To avoid this warning there is a simple trick. Instead, call the web service methods 
using the `$client->call(method, params)`. In this case you simply pass the method name as a string. 
Note that the parameters must be passed in a double nested array.

This is the **wrong** way to use the `call` method:

```php
$result = $client->call('sayHello', ['firstName' => 'World']);
```

This example leads to the following error:

> Fatal error: Uncaught SoapFault exception: [env:Receiver] Too few arguments 
> to function Hello::sayHello(), 0 passed and exactly 1 expected in 
> laminas\laminas-soap\src\Client.php on line 1166

In this way, the `call` method is used correctly:

```php
$result = $client->call('sayHello', [['firstName' => 'World']]);
```

## Creating a SOAP Client with C#

Now, the simplest way is to generate proxy classes in C# application (this process is called adding service reference). There are 2 main ways of doing this. Dotnet provides ASP.NET services, which is old way of doing SOA, and WCF, which is the latest framework from MS and provides many protocols, including open and MS proprietery ones.

Now, enough theory and lets do it step by step.

1. Open your project (or create a new one) in visual studio
2. Right click on the project (on the project and not the solution) in Solution Explorer and click `Add Service Reference`
3. A dialog should appear (Add service reference). 
  Enter the url (`http://localhost/api.php?wsdl`) of your wsdl file and click the `Go` button.
4. Now you should see a `HelloService` (name may vary). You should see generated proxy class name and namespace. In my case, the namespace is `WindowsFormsApp1.ServiceReference1`, the name of proxy class is `HelloPortClient`. As I said above, class names may vary in your case. 
5. Go to your C# source code. Add `using WindowsFormsApp1.ServiceReference1`.
6. Now you can call the service this way:

```csharp
ServiceReference1.HelloPortClient helloClient = new ServiceReference1.HelloPortClient();

string result = helloClient.sayHello("World");

Console.WriteLine(result);
```

[Source](https://stackoverflow.com/questions/3100458/soap-client-in-net-references-or-examples)

Hope this helps. 

And now have fun creating your own SOAP webservice :-)

## Known issues

If you'll encounter any issues, please create a detailed error description in the comments section.

> It doesn't work on my machine

or

> I want to use GET/PUT/PATCH methods

* Don't confuse SOAP with a RESTful API. 
* SOAP uses the POST method and has a single endpoint (URL). 

> Is it possible to convert SOAP into a RESTful?

* SOAP and REST is something completely different. Usually the API must be completely rewritten.
* A SOAP endpoint itself could "translate" and forward the request to an RESTful API.

> Can I use that in Laravel / Symfony / Slim / Framework X too?

* This is a framework independent concept. It's possible.
* Depending on the framework, add the server code to the controller action and transform the XML response into a (PSR-7) response.
