---
title: REST, RESTful, REST-like API Resources
layout: post
comments: true
published: false
description: 
keywords: 
---

## Papers

* [Roy Fielding's paper on REST](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm) (where the word comes from)
* [REST APIs must be hypertext-driven](http://roy.gbiv.com/untangled/2008/rest-apis-must-be-hypertext-driven)
* [Architectural Styles and the Design of Network-based Software Architectures](http://www.ics.uci.edu/~fielding/pubs/dissertation/top.htm) (Dissertation)

## Blog posts

* [Richardson Maturity Model - steps toward the glory of REST](https://martinfowler.com/articles/richardsonMaturityModel.html)
* [HATEOAS 101: Introduction to a REST API Style](https://apigee.com/about/blog/technology/hateoas-101-introduction-rest-api-style-video-slides)
* [Learn REST: A RESTful Tutorial](https://www.restapitutorial.com/)
* [Good Practices in API Design](https://swaggerhub.com/blog/api-design/api-design-best-practices/)
* [7 Tips for Designing a Better REST API](http://www.kennethlange.com/posts/7_tips_for_designing_a_better_rest_api.html)
* [Joshua Bloch: Bumper-Sticker API Design](https://www.infoq.com/articles/API-Design-Joshua-Bloch)
* [Best Practices for Designing a Pragmatic RESTful API](http://www.vinaysahni.com/best-practices-for-a-pragmatic-restful-api)
* [Act Three: The Maturity Heuristic](https://www.crummy.com/writing/speaking/2008-QCon/act3.html)
* [There’s a model hiding in your REST API](http://transmission.vehikl.com/theres-a-model-hiding-in-your-rest-api/)
* [The Ultimate Guide to API Design](https://blog.qmo.io/ultimate-guide-to-api-design/#documentation)
* [RESTful Webservices](https://www.mittwald.de/blog/webentwicklung-webdesign/webentwicklung/restful-webservices-1-was-ist-das-uberhaupt) (german)
* [SOAP vs REST differences](https://stackoverflow.com/a/19884975/1461181)

## What is HATEOAS?

<https://en.wikipedia.org/wiki/HATEOAS>

### Drawbacks of HATEOAS

Humans can follow links, even as a website goes through significant changes. 
Code can follow links, but can't make clear independent decisions when significant changes happen to the API.

## Specifications

`There is no REST specification, REST is a architecture style.`

Read more: 

* OpenAPI Specification for RESTful APIs: <https://swagger.io/specification/>

## REST API Guidelines

* [Microsoft REST API Guidelines](https://github.com/Microsoft/api-guidelines/blob/master/Guidelines.md)
* [Google API Design Guide](https://cloud.google.com/apis/design/)
* [RESTful APIs](http://tech.sparkfabrik.com/2017/03/04/php-rest-tools-showdown-series---part-1-really-restful-apis/)
* [Response Handling](https://blogs.mulesoft.com/dev/api-dev/api-best-practices-response-handling/)
* [HTTP Status Codes](http://www.iana.org/assignments/http-status-codes/http-status-codes.xhtml)
* [Cient errors 400, 422](https://developer.github.com/v3/#client-errors)

## Books

* [REST und HTTP: Entwicklung und Integration nach dem Architekturstil des Web](https://www.amazon.de/REST-HTTP-Entwicklung-Integration-Architekturstil/dp/3864901200) (german)
* [RESTful Web APIs](https://www.amazon.de/RESTful-Web-APIs-Leonard-Richardson/dp/1449358063)
* [RESTful Web Services](https://www.amazon.de/RESTful-Web-Services-Leonard-Richardson/dp/0596529260)
* [REST API Design Rulebook](https://www.amazon.com/REST-Design-Rulebook-Mark-Masse/dp/1449310508)
* [RESTful Web Services Cookbook](http://shop.oreilly.com/product/9780596801694.do)

## Tools

* [Generate JSON schema from JSON](https://jsonschema.net/)
* [JSON Schema](https://json-schema.org/) (Validation)
* [XMLSpy - A Graphical JSON Schema Editor](https://www.altova.com/download-json-schema-editor.html)

## Documentation

* API documentation with [swagger](https://swagger.io/) (recommended)
* <https://bocoup.com/blog/documenting-your-api>
* [Documenting your REST API](https://gist.github.com/iros/3426278)

## Building a RESTful API

* [REST, RESTful API Quick Reference](https://odan.github.io/2017/04/17/rest-restful-api-quick-reference.html)
* [Slim Framework](https://www.slimframework.com/) (PHP)

## Calling a RESTful API using PHP

* [Guzzle](http://docs.guzzlephp.org/en/latest/) (recommended)
* [Using GuzzlePHP with RESTful API's](http://josephralph.co.uk/using-guzzle-with-restful-apis-digitalocean-api/)
* [CURL and PHP REST client](https://stackoverflow.com/a/21271348/1461181) 
* [POSTing JSON Data With PHP cURL](https://lornajane.net/posts/2011/posting-json-data-with-php-curl)

## REST alternatives

* [JSON-RPC 2.0](https://www.jsonrpc.org/specification) (A mix of GraphQL and RESTful)
* [GraphQL](https://graphql.org/) (Facebook)
* [SOAP Webservice](https://en.wikipedia.org/wiki/SOAP) (XML)
* [gRPC](https://grpc.io/) (Google)
* [JSON API](https://jsonapi.org/)

## Drawbacks

* [REST is the new SOAP](https://medium.freecodecamp.org/rest-is-the-new-soap-97ff6c09896d)
* [RESTful APIs, the big lie](https://mmikowski.github.io/the_lie/)
* [When REST isn't Good Enough](https://www.braintreepayments.com/blog/when-rest-isnt-good-enough/)
* [Why is GitHub using GraphQL and not REST?](https://developer.github.com/v4/#why-is-github-using-graphql)

## Discussions

* <https://news.ycombinator.com/item?id=13706618>
* <https://news.ycombinator.com/item?id=10042969>
