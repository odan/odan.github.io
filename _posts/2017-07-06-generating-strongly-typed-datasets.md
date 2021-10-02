---
title: Generating Strongly Typed DataSets
layout: post
comments: true
published: false
description: 
keywords: csharp, poco, database
---

## C# .NET

Use a class (POCO) for this:

```csharp
public class ClassName
{
    public string Col1 { get; set; }
    public int Col2 { get; set; }
}
```

Now you can use a loop to fill a list and ToArray if you really need an array:

```csharp
ClassName[] allRecords = null;
string sql = @"SELECT col1,col2
               FROM  some table";
using (var command = new SqlCommand(sql, con))
{
    con.Open();
    using (var reader = command.ExecuteReader())
    {
        var list = new List<ClassName>();
        while (reader.Read())
            list.Add(new ClassName { Col1 = reader.GetString(0), Col2 = reader.GetInt32(1) });
        allRecords = list.ToArray();
    }
}
```

Note that i've presumed that the first column is a string and the second an integer. 
Just to demonstrate that C# is typesafe and how you use the DataReader.GetXY methods.

Source: <https://stackoverflow.com/a/20101807/1461181>

## Creating a Model

* <https://docs.microsoft.com/en-us/ef/core/modeling/>