---
title: Creating a custom rule in PHPStan to disallow accidental assignments in conditionals
layout: post
comments: true
published: true
description:
keywords: php phpstan
---

PHPStan is a great tool for catching common mistakes and bugs in your PHP code, 
but sometimes you may need to create your own custom rules to catch specific issues. 
One common issue that can be hard to catch is accidental assignments in conditionals. 
In this post, we'll show you how to create a custom rule in PHPStan to catch this issue.

First, let's take a look at an example of an accidental assignment in a conditional:

```php
if ($x = getValue()) {
    // do something
}
```

In this example, the developer probably meant to check if `getValue()` returns 
a truthy value, but instead they accidentally assigned the return value 
of `getValue()` to `$x`. This can cause unexpected behavior in the code 
and be hard to track down.

To catch this issue, we can create a custom rule in PHPStan that 
checks for assignments in conditional statements and raises an 
error if one is found. Here's an example implementation of the rule:

File: `src/Rules/DisallowAssignInConditionalRule.php`

```php
<?php

namespace App\Rules;

use PhpParser\Node;
use PhpParser\Node\Expr\Assign;
use PHPStan\Analyser\Scope;
use PHPStan\Rules\Rule;

/**
 * @implements Rule<Node\Expr\Assign>
 */
class DisallowAssignInConditionalRule implements Rule
{
    public function getNodeType(): string
    {
        return Assign::class;
    }

    public function processNode(Node $node, Scope $scope): array
    {
        $parent = $node->getAttribute('parent');

        if ($parent instanceof Node\Stmt\If_ ||
            $parent instanceof Node\Stmt\While_ ||
            $parent instanceof Node\Stmt\For_) {
            return [
                sprintf(
                    'Assignment in conditional statement on line %d is not allowed.',
                    $node->getLine()
                ),
            ];
        }

        return [];
    }
}
```

This implementation checks if the parent node of the assignment 
is a conditional statement (if, while, for), If the parent node 
is a conditional statement, it raises an error.

To use this rule, you need to include the class in your 
PHPStan configuration file `phpstan.neon` and enable the rule. 

```yaml
services:
    - 
        class: App\Rules\DisallowAssignInConditionalRule
        tags: [phpstan.rules.rule]
```

Once enabled, PHPStan will raise an error if it finds 
an assignment in a conditional statement, 
indicating that it may be an accidental assignment.

Please keep in mind that this is a basic example, 
you may need to adjust the implementation of your 
rule based on your specific needs.

In conclusion, creating custom rules in PHPStan can be a great way 
to catch specific issues in your code that might be hard to track down otherwise. 
With a bit of PHP knowledge and the right tools, 
you can create your own rules to catch common mistakes 
and bugs in your code, and make your development 
process more efficient and effective.
