---
title: RESTful API - Quick Reference
layout: post
comments: true
published: true
description: 
keywords: 
---

> A good API is not just easy to use but also hard to misuse.

### Resources

* Version your API
  * Path: /v1/users
  * Subdomain: api.v1.example.com/users
  
* URI must be nouns, not verbs
  * /cars
  * /users
  * /books/{id}
* All nouns are plurals
  * GET /users
  * DELETE /users/{id}
  * GET /users/{id}/reviews
  * POST /users/{id}/reviews
  * PUT /users/{id}/reviews/{reviewid}
* Map relations by sub-resources
  * GET /users/{id}/reviews
  * PUT /users/{id}/reviews/{reviewid}
* Allow for collections filtering, sorting and paging
  * GET /users/{id}/reviews?published=1
  * GET /users?sort[]=-age&sort[]=+name
  * GET /books?format[]=epub&format[]=mobi
* GET requests must never alter system/resource state (GET is readonly)
* Identify all the resources
  * Few resources are atomic; most are collections or views of other resources
  * Do not confuse identity (naming) with containment (storage)
  * Resources have more in common with procedures than they do with records or files
* Iteratively develop resources and uses cases (state transitions)
  * Do not try to do everything at once
  * Do not make any assumption about received content, order, versioning, etc.
  
### Security

* Allow only HTTPS requests
* Use access control, not obscurity, to control publication

### Authentication

* Use a standard and stateless authentication method:
  * API Key / Token (as part of the `Authorization` header)
  * Basic Authentication (Basic Auth)
* If the authentication is unsuccessful, the status code `403` (Forbidden) must be returned.

### HTTP Header

* Content-type: application/json

\pagebreak 

### HTTP Methods

HTTP Method | HTTP Path | CRUDL | SQL | [OperationId](https://github.com/watson-developer-cloud/api-guidelines/blob/master/swagger-coding-style.md#operationid) | Description | Response codes
|--- | ---  | --- | --- | --- | --- | --- |
GET | /users | List | SELECT | listUsers / getUsers | Used for retrieving a list of resources | 200 (OK), 404 (Not Found)
GET | /users/{id} | Read | SELECT | getUserById | Used for retrieving a single resources | 200 (OK), 404 (Not Found)
POST | /users | Create | INSERT | createUser or addUser | Used for creating resources | 201 (Created), 404 (Not Found), 409 (Conflict)
PUT | /users/{id} | Update	| UPDATE | updateUser | Used for updating resources | 200 (OK), 204 (No Content), 404 (Not Found)
DELETE | /users/{id} | Delete | DELETE | deleteUser | Used for deleting resources | 200 (OK), 404 (Not Found)

\pagebreak 

### HTTP Status Codes

Status codes indicate the result of the HTTP request.

* 1XX - Informational
* 2XX - Success
* 3XX - Redirection
* 4XX - Client error
* 5XX - Server error

Return meaningful status codes:

Code | Name| What does it mean?
--- | ---  | --- |
200 | OK | All right!
201 | Created | If resource was created successfully.
400 | Bad Request | 4xx Client Error
401 |	Unauthorized | You are not logged in, e.g. using a valid access token
403 |	Forbidden	| You are authenticated but do not have access to what you are trying to do
404	| Not found	| The resource you are requesting does not exist
405	| Method not allowed | The request type is not allowed, e.g. /users is a resource and POST /users is a valid action but PUT /users is not.
409 | Conflict | If resource already exists.
422	| Unprocessable entity | Validation failed. The request and the format is valid, however the request was unable to process. For instance when sent data does not pass validation tests.
500	| Server error | 5xx Server Error. An error occurred on the server which was not the consumer's fault.

\pagebreak 

### Error handling

#### General server error

Send a `500 Internal Server Error` response.

**HTTP status code:** 500

**Content:**

```json
{
  "error": {
    "message": "Application Error"
  }
}
```

**Content with error code:**

```json
{
  "error": {
    "code": 5001,
    "message": "Database connection failed"
  }
}
```

#### General client error

Sending invalid JSON will result in a `400 Bad Request` response.

**HTTP status code:** 400

**Content:**

```json
{
  "error": {
    "code": 400,
    "message": "Problems parsing JSON"
  }
}
```

#### Validation errors

Sending invalid fields will result in a `422 Unprocessable Entity` response.

**HTTP status code:** 422

**Content:**
```json
{
  "error": {
    "code": 422,
    "message": "Validation failed",
    "details": [
      {
        "field": "email",
        "message": "Email is required"
      }
    ]
  }
}
```

\pagebreak 

### Examples

#### jQuery

```javascript
// Get base url
var url = $('head base').attr('href')
```

A GET request:

```javascript
$.ajax({
    url: url + "users/1",
    type: "GET",
    cache: false,
    contentType: 'application/json',
    dataType: 'json'
}).done(function (data) {
    alert('Successfully');
    console.log(data);
}).fail(function (xhr) {
    var message = 'Server error';
    if (xhr.responseJSON && xhr.responseJSON.error.message) {
       message = xhr.responseJSON.error.message;
    }
    alert(message);
});
```

A POST request with payload:

```javascript
var data = {
    username: "max",
    email: "max@example.com"
};

$.ajax({
    url: url + "users",
    type: "POST",
    contentType: 'application/json',
    dataType: 'json',
    data: JSON.stringify(data)
}).done(function (data) {
    alert("Success");
    console.log(data);
}).fail(function (xhr) {
    if (xhr.status === 422) {
        // Show validation errors
        var response = xhr.responseJSON;
        alert(response.error.message);
        $(response.error.details).each(function (i, error) {
           console.log("Error in field [" + error.field + "]: " + error.message);
        });
    } else {
        var message = 'Server error';
        if (xhr.responseJSON && xhr.responseJSON.error.message) {
           message = xhr.responseJSON.error.message;
        }
        alert(message);
    }
});
```

### Read more

* <https://blog.restcase.com/7-rules-for-rest-api-uri-design/>
* <https://opensource.zalando.com/restful-api-guidelines/index.html#json-guidelines>
