---
title: Creating immutable Value Objects in C# .NET
layout: post
comments: true
published: false
description: 
keywords: 
---

### Example

```cs
public class User : ICloneable
{
    private int id;

    private string firstName;

    private string lastName;

    public object Clone()
    {
        return this.MemberwiseClone();
    }

    public bool Equals(User user)
    {
        return this.id == user.id &&
            this.firstName == user.firstName &&
            this.lastName == user.lastName;         
    }

    public User withId(int id)
    {
        User clone = (User)this.MemberwiseClone();
        clone.id = id;

        return clone;
    }

    public int getId()
    {
        return this.id;
    }

    public User withFirstName(string firstName)
    {
        User clone = (User)this.MemberwiseClone();
        clone.firstName = firstName;

        return clone;
    }

    public string getFirstName()
    {
        return this.firstName;
    }

    public User withLastName(string lastName)
    {
        User clone = (User)this.MemberwiseClone();
        clone.lastName = lastName;

        return clone;
    }

    public string getLastName()
    {
        return this.lastName;
    }

}
```

## Usage

```cs
User user = new User();

// Set a single value
user = user.withId(1);

// Set multiple values (fluent interface)
user = user.withFirstName("John").withLastName("Doe");

// Read a value
Console.WriteLine(user.getId()); // 1

//
// Comparing value objects
//
User user2 = new User();
user2 = user2.withId(1);

User user3 = new User();
user3 = user2.withId(1);

bool equals = user2.Equals(user3); // True
Console.WriteLine(equals.ToString()); 
```