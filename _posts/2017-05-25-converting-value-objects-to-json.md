---
title: Converting value objects to JSON
layout: post
comments: true
published: false
description: 
keywords: 
---

A value object usually consists only of setter / getter methods and stores the values 
in protected member variables. Normally json_encode is used to convert arrays to JSON. 
However, Value Objects must implement [JsonSerializable](https://secure.php.net/manual/en/jsonserializable.jsonserialize.php) so that the conversion to JSON works. Here's an example:

```php
class QuoteEntity implements JsonSerializable
{
    protected $id;
    protected $author;
    protected $quote;

    public function __construct(array $data) {
        // No id if we're creating
        if(isset($data['id'])) {
            $this->id = $data['id'];
        }
        $this->quote = $data['quote'];
        $this->author = $data['author'];
    }

    public function getId() {
        return $this->id;
    }

    public function getQuote() {
        return $this->quote;
    }

    public function getAuthor() {
        return $this->author;
    }
    
    public function jsonSerialize() {
        return array(
            'id' => $this->getId(),
            'quote' => $this->getQuote(),
            'author' => $this->getAuthor()
        );
    }
 }
```

## Usage

```php
$entity = new QuoteEntity(array(
    'id' => 1, 
    'quote' => 'Text...', 
    'author' => 'Bill'
));

echo json_encode($entity, JSON_PRETTY_PRINT);
```

## Output

```json
{
    "id": 1,
    "quote": "Text...",
    "author": "Bill"
}
```

## Read more

* https://secure.php.net/manual/en/jsonserializable.jsonserialize.php
* https://www.reddit.com/r/PHPhelp/comments/6d8hdi/using_slim_and_php_to_learn_apis/
* https://3v4l.org/6T6W0
