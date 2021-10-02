---
title: Parsing a XML document with PHP DOM and Xpath
layout: post
comments: true
published: false
description: 
keywords: php
---

Example:

```php
<?php

$xmlContent = file_get_contents('z-invoice.xml');
$dom = new DOMDocument("1.0", "UTF-8");
$dom->preserveWhiteSpace = false;

$dom->loadXml($xmlContent);

$xpath = new DOMXPath($dom);

//
// Add namespaces automatically
//
// Fetch the namespaces, add a few lines to register these back with the document
// so that you can use them in XPath expressions...
foreach ($xpath->query('//namespace::*') as $namespaceNode) {
    $prefix = str_replace('xmlns:', '', $namespaceNode->nodeName);
    $namespaceUri = $namespaceNode->nodeValue;
    $xpath->registerNamespace($prefix, $namespaceUri);
}

//
// You can add the namespaces manually of course.
//
//$xpath = new DOMXPath($dom);
//$xpath->registerNamespace("xmlns", "http://tempuri.org/invoice_batch_generic.xsd");

// Root node
$rootNode = $xpath->query('/xmlns:invoice_batch_generic')->item(0);

// Invoice data
$invoiceNumber = $xpath->query('xmlns:account/xmlns:invoice/xmlns:invoice_number', $rootNode)->item(0)->nodeValue;
$taxRate = $xpath->query('xmlns:taxregioncategory/xmlns:total_tax_rate_per_category_and_region', $rootNode)->item(0)->nodeValue;
$totalTaxAmount = $xpath->query('xmlns:taxregioncategory/xmlns:total_tax_amount', $rootNode)->item(0)->nodeValue;

// Invoice items
$itemList = array();
$invoiceItemNodes = $xpath->query('xmlns:account/xmlns:invoice/xmlns:invoice_item', $rootNode);

// Then loop over the items...

/** @var DOMNode $invoiceItemNode */
foreach ($invoiceItemNodes as $invoiceItemNode) {
    $row = [];

    /** @var DOMNode $field */
    foreach ($invoiceItemNode->childNodes as $field) {
        $row[$field->tagName] = $field->nodeValue;
    }
    $itemList[] = $row;
}

print_r($itemList);
```

Example file:

* [z-invoice.xml](https://gist.github.com/odan/6aeb43a9f9c3fbd332d364c175a760fc#file-z-invoice-xml)
