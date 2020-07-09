---
title: Value Objects with "at least one item required" constraint
layout: post
comments: true
published: false
description: 
keywords: 
---

Here is an example how to enforce minimal collection constraints on PHP value objects:

```php
final class Price
{
    private $value;

    public function __construct(float $value)
    {
        $this->value = $value;
    }

    public function price(): float
    {
        return $this->value;
    }
}

final class Prices
{
    private $items = [];

    public function __construct(Price $firstPrice, Price ...$further)
    {
        $this->items = array_merge([$firstPrice], $further);
    }

    // this method is just optional
    public function add(Price $price)
    {
        $clone = clone $this;
        $clone->items[] = $price;
        return $clone;
    }

    /**
     * Return all items.
     *
     * @return Price[]
     */
    public function items(): array
    {
        return $this->items;
    }
}

//
// Usage
//
$prices = new Prices(new Price(99), new Price(120.1));

$prices = $prices->add(new Price(42));

// Cool
var_dump($prices->items());

// Without a price -> Too few arguments error
//$prices2 = new Prices();

```

## Links

* <https://twitter.com/Ocramius/status/983817609375371264>
* <https://twitter.com/nikolaposa/status/990687993009762307>

## Keywords

php, value objects, ddd