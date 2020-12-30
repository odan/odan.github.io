---
title: A new concept for a JSON-RPC Service
date: 2017-10-11
layout: post
comments: true
published: false
description: 
keywords: 
---

This article presents a new API concept based on established web standards. If you think that SOAP (XML) is a bloat and REST/RESTful is a mess then you should read further.


### Requirements

* Please read this [article](https://dev.to/mogui/no-rest-for-the-wicked-e8l) before you continue.
* Experience with JSON and web technologies

### Introduction

Currently, the API development is in a dead end. The strongly typed SOAP protocol was too 
"Java enterprise architect" like. 
REST tried to replace SOAP by saying "Do what you want, 
just use HTTP verbs, URIs and the HTTP payload". 
But sorry, this was a mess, because REST (and RESTful) is just a 
"Architectural Style" and was never intended to be standardized. 
REST with his "autodiscovery" feature failed, because nobody was able 
to write a real universal "AI" REST client. As a developer you still 
have to write and maintain the API documentation with Swagger (API Framework) or other tools. 
REST has no strongly typed validation like SOAP has, 
and that's what we need for a serious business / enterprise API.

How can we solve this problem?

Let us focus on the good parts and filter out the bad parts. 

## HTTP Considerations

### Transport

All interaction takes place over secure HTTP (SSL) between the client application 
(e.g., website or mobile app) and the service. 
All invocations must be made via HTTP **POST**. 
In particular, HTTP GET is not supported.

Use authorization methods like a API Key or Basic Authentication.

### URL

The URL for invoking the API is a single endpoint.

* Example endpoint: https://www.example.com/json/v1/invoke
* Another example: https://api.example.com/v1/invoke

The URL for getting the API schema a single endpoint.

* Example endpoint: https://www.example.com/json/v1/schema
* Another example: https://api.example.com/v1/schema

### Content Types

The HTTP content type of the request (sent as a HTTP request header) should be set to `application/json`. 

### Evaluation

[**JSON-RPC 2.0**](http://www.jsonrpc.org/specification) fits perfectly into the concept. Everything is a POST (like SOAP). A string contains the name of the method to be invoked (e.g. "Users.getById"). Last but not least, it's JSON.

[**JSON-Schema**](http://json-schema.org/) allows us to annotate and validate JSON documents. This will act as the "JSON Web Services Description Language" (JWSDL). 

[**JSON Schema Service Descriptor**](http://www.simple-is-better.org/json-rpc/jsonrpc20-schema-service-descriptor.html) This json-schema describes all JSON-RPC methods, parameters and return values.

### The service request / response sequence

1. The Client downloads the json-schema to get a list with all rpc methods, parameters and return types. This step is optional.
2. The client invokes the specific method via the JSON-RPC 2.0 protocol
3. The server validates the request (schema and data).
4. If the request is valid: Process request and send response.
5. If the request is not valid (wrong schema or validation error): Send the error object.

### The JSON service descriptor

Each top level property defines an available rpc methods. 
In addition, a service descriptor should contain the "id", "description" properties, 
and may contain the "version" property.

```json
{
  "id":"https://www.example.com/api/math/v1/schema",
  "description":"Some math methods. This property can be used for a description.",
  "version": "1.0.0",
  "Math.divide": {
    "description": "Divide one number by another",
    "type": "method",
    "returns": "number",
    "params": [
      { "type": "number", "name": "dividend", "required": true },
      { "type": "number", "name": "divisor", "required": true }
    ]
  }
}

```

### Example calls

Invoke the method `Math.divide` with named parameters:

```json
{
  "method": "Math.divide", 
  "params": {
    "dividend": 10,
    "divisor": 2
  }
}

```
The result looks good :-)

```json
{
  "result": 5
}

```

### Making an invalid call 

This is an method call with invalid request object. The second parameter (divisor) is not provided.

```json
{
  "method": "Math.divide", 
  "params": {
    "dividend": 10
  }
}

```

There was an error triggered during invocation.


```json
{
  "error": {
    "code": 422, 
    "message": "Validation failed",
    "errors": [
      {
        "field": "divisor",
        "message": "The parameter divisor is missing"
      }
    ]
  }
}

```

More details about the RPC error object: http://www.jsonrpc.org/specification#error_object


### Summary

This concept shows you that you could have the best of both worlds:
A lean JSON-RPC 2.0 API with a strong schema validation. 
Nothing is really new, but the combination is of these parts is the key.