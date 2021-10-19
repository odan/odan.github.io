---
title: Basic CRUD operations with PDO
layout: post
comments: true
published: true
description: 
keywords: 
---

CRUD = Create, Read, Update, Delete

## Open a database connection

```php
$host = '127.0.0.1';
$dbname = 'test';
$username = 'root';
$password = '';
$charset = 'utf8mb4';
$collate = 'utf8mb4_unicode_ci';
$dsn = "mysql:host=$host;dbname=$dbname;charset=$charset";
$options = [
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
    PDO::ATTR_PERSISTENT => false,
    PDO::ATTR_EMULATE_PREPARES => true,
    PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
    PDO::MYSQL_ATTR_INIT_COMMAND => "SET NAMES $charset COLLATE $collate"
];

$pdo = new PDO($dsn, $username, $password, $options);
```

**Security note:** When connecting to a database with PDO, 
it is important to ensure that the database credentials are not published if the connection fails.
To do this, wrap the PDO DSN in a try/catch block to catch any connection exception.
If your application does not catch the exception thrown from the PDO constructor, 
the default action taken by the zend engine is to terminate the script and display 
a back trace. This back trace will likely reveal the full database connection details, 
including the username and password. It is your responsibility to catch this exception.

**Example:**

```php
// ...

try {
    $pdo = new PDO($dsn, $username, $password, $options);
} catch (Exception $exception) {
    throw new RuntimeException('Error establishing a database connection.');
}
```

## Select multiple rows

Without parameters:

```php
$userRows = $pdo->query('SELECT * FROM users')->fetchAll();
```

With parameters (prepared statements):

```php
$statement = $pdo->prepare("SELECT * FROM employees WHERE name = :name");
$statement->execute(['name' => $name]);

foreach ($statement as $row) {
    // do something with $row
}

// or with the fetch method
while ($row = $statement->fetch()) {
   // do something with $row
}
```

## Select a single row

With parameters (prepared statements):

```php
$statement = $pdo->prepare("SELECT * FROM users WHERE email = :email AND status=:status LIMIT 1");
$statement->execute(['email' => $email, 'status' => $status]);

$userRow = $statement->fetch();
```

## Insert a single row

```php
$row = [
    'username' => 'bob',
    'email' => 'bob@example.com'
];
$sql = "INSERT INTO users SET username=:username, email=:email;";
$status = $pdo->prepare($sql)->execute($row);

if ($status) {
    $userId = (int)$pdo->lastInsertId();
}
```

## Insert multiple rows

```php
$rows = [];

$rows[] = [
    'username' => 'bob',
    'email' => 'bob@example.com'
];

$rows[] = [
    'username' => 'max',
    'email' => 'max@example.com'
];

$sql = "INSERT INTO users SET username=:username, email=:email;";
$statement = $pdo->prepare($sql);

foreach ($rows as $row) {
    $statement->execute($row);
}
```

## Update a single row

```php
$row = [
    'id' => 1,
    'username' => 'bob',
    'email' => 'bob2@example.com'
];

$sql = "UPDATE users SET username=:username, email=:email WHERE id=:id;";
$pdo->prepare($sql)->execute($row);
```

## Update multiple rows

```php
$row = [
    'updated_at' => '2017-01-01 00:00:00'
];

$sql = "UPDATE users SET updated_at=:updated_at";
$statement = $pdo->prepare($sql);
$statement->execute($row);

// optional
$affected = $statement->rowCount();
```

## Delete a single row

```php
$where = ['id' => 1];
$pdo->prepare("DELETE FROM users WHERE id=:id")->execute($where);
```

## Delete multiple rows

```php
$pdo->prepare("DELETE FROM users")->execute();
```

## PDO data types

You can set a explicit data type for the parameter using the `PDO::PARAM_*` constants.

```php
// Usage
$email = (string)$_POST['email'];

$statement = $pdo->prepare('INSERT INTO users SET email=:email;');
$statement->bindValue(':email', $email, PDO::PARAM_STR);
$statement->execute();
```

## Prepared statements using the IN clause

According to the [PHP manual](https://www.php.net/manual/en/pdo.prepare.php) it's not possible with PDO:

> You cannot bind multiple values to a single named parameter in, for example, the IN() clause of an SQL statement.

This helper function converts all array values into a (safe) quoted string. 

```php
function quote_values(PDO $pdo, array $values) {
    array_walk($values, function (&$value) use ($pdo) {
        if($value === null) {
            $value = 'NULL';
            return;
        }
        $value = $pdo->quote(is_bool($value) ? (int)$value : (string)$value);
    });
    
    return implode(',', $values);
}
```

### Usage

```php
$ids = [
    1,
    2,
    3,
    "'", 
    null,
    'string',
    123.456,
    true,
    false,
];

$pdo = new PDO('sqlite::memory:');
$sql = sprintf("SELECT id FROM users WHERE id IN(%s)", quote_values($pdo, $ids));
echo $sql . "\n";

$statement = $pdo->prepare($sql);
$statement->execute();
```

Generated SQL:

```sql
SELECT id FROM users WHERE id IN('1','2','3','''',NULL,'string','123.456','1','0')
```

Other solutions to create IN clauses:

* [PHP PDO Prepared Statements to Prevent SQL Injection](https://websitebeaver.com/php-pdo-prepared-statements-to-prevent-sql-injection#where-in-array)

* [Prepared statements and IN clause](https://phpdelusions.net/pdo#in)
