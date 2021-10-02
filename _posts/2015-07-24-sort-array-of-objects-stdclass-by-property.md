---
title: Sort array of objects (stdClass) by property
layout: post
comments: true
published: false
description: 
keywords: 
---

```php
/**
 * Sort array of objects (stdClass) by property
 *
 * @param array $objects array of objects to sort
 * @param string $fieldName field to sort
 * @param int $sortOrder sort order (default is SORT_ASC)
 * @param int $sortFlag sort flag (default is SORT_REGULAR)
 *
 * @return array array of objects
 */
public function sortObjectsByByField($objects, $fieldName, $sortOrder = SORT_ASC, $sortFlag = SORT_REGULAR)
{
    $sortFields = array();
    foreach ($objects as $key => $row) {
        $sortFields[$key] = $row->{$fieldName};
    }
    array_multisort($sortFields, $sortOrder, $sortFlag, $objects);
    
    return $objects;
}
```

