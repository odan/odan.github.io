---
title: Maximizing Database Efficiency - Why UUIDs Should Stay at the API Level
layout: post
comments: true
published: true
description: 
keywords: uuid
---

In software development, choosing the right identifiers for your database and API can significantly impact the performance, scalability, and security of your application. One common debate is whether to use UUIDs (Universally Unique Identifiers) or traditional integer-based primary keys in the database. 

While UUIDs offer several advantages, they should be primarily used at the API level rather than as primary keys in the database. In this blog post I will try to explore the reasons behind this recommendation and provides a approach for handling UUIDs.

## Advantages of Using UUIDs at the API Level

**Global Uniqueness:** UUIDs ensure that each identifier is unique across different systems and databases. This is particularly useful for distributed systems where multiple databases might need to sync or communicate with each other.

**Security:** Exposing sequential primary keys through APIs can be a security risk, as it makes it easier for malicious users to guess valid IDs. UUIDs are more secure as they are harder to predict.

## Why UUIDs Should Not Be Used as Primary Keys in Databases

While UUIDs are excellent for "API-level" operations, they come with several drawbacks when used as primary keys in databases:

**Performance Issues:** UUIDs are larger than typical integer-based keys (16 bytes vs. 4 or 8 bytes). This increased size can slow down database operations, especially when dealing with large datasets.

**Index Fragmentation:** Although newer versions like UUIDv7 offer sequential (sortable) generation to reduce fragmentation, they still aren't as efficient as simple integer-based keys. Index fragmentation can still be an issue due to the nature of UUIDs, especially if the database is not optimized to handle them.

## Mapping UUIDs to Internal IDs

To benefit of UUIDs while avoiding their drawbacks, a hybrid approach could be used:

**Use UUIDs at the API Level:** Expose UUIDs in your API endpoints to provide globally unique and secure identifiers to external clients.

**Map UUIDs to Internal IDs:** In your API layer, map incoming UUIDs to internal integer-based (primary) keys. This should be one of the first operation performed upon receiving an API request.

**Perform (Database) Operations Using Internal IDs:** Once the UUID is mapped to an internal ID, use this internal ID for all subsequent database queries, joins, and operations. This ensures that your database remains efficient and performant.

## Example Workflow

**Step 1: API Request**

A client makes a request to your API with a UUID.

```
GET /api/v1/users/550e8400-e29b-41d4-a716-446655440000
```

**Step 2: Controller**

The controller receives the request and extracts the UUID. It then calls the UUID mapper to map the UUID to an internal ID.

```cs
// The UUID mapper
int userId = _userUuidMapper.MapUuidToId(userUuid);
```

**Step 3: Service**

The service handles the business logic and 
retrieves the (user) data from the database using the internal ID.

The application performs the necessary database operations using the internal ID.

The service method returns a DTO which, if necessary, also contains the corresponding external UUID(s).

```cs
// The service
User user = _userReader.GetUserById(userId);
```

**Step 4: Response:**

The transformer takes the (user) data and handles the conversion
from the DTO into the JSON representation.

```cs
var result = _userTransformer.Transform(user);

return Ok(result);
```

Example:

```cs
// UserTransformer.cs
public class UserTransformer
{
    public UserResponse Transform(User user)
    {
        return new UserResponse
        {
            UUID = user.UUID,
            Name = user.Name,
            Email = user.Email
        };
    }
}

// UserResponse.cs
public class UserResponse
{
    public Guid UUID { get; set; }
    public string Name { get; set; }
    public string Email { get; set; }
}

// User.cs
public class User
{
    public int Id { get; set; }
    public Guid UUID { get; set; }
    public string Name { get; set; }
    public string Email { get; set; }
}

```

The controller returns the JSON response to the client.

```json
{
    "uuid": "550e8400-e29b-41d4-a716-446655440000",
    "name": "John Doe",
    "email": "john.doe@example.com"
}
```

## Conclusion

Using UUIDs at the API level provides the benefits of global uniqueness and enhanced security without compromising the performance and efficiency of your database. By mapping UUIDs to internal integer-based primary keys and using these keys for all database operations, you can achieve a balance between secure, scalable API design and high-performance database management. This approach ensures that your application can handle the demands of modern distributed systems while maintaining optimal performance.

## Read more

* [The Problem with Using a UUID Primary Key in MySQL](https://planetscale.com/blog/the-problem-with-using-a-uuid-primary-key-in-mysql)
