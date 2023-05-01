---
title: Filter Parser
description: Parsing LDAP Query Filters
---

# Filter Parser

## Introduction

LdapRecord comes with a built-in LDAP filter parser, giving you the
ability to read the nodes within to extract all of their attributes.

Let's start with a small example by parsing the filter `(cn=Steve)`:

```php
use LdapRecord\Query\Filter\Parser;

// array: [
//  0 => LdapRecord\Query\Filter\ConditionNode
// ]
$nodes = Parser::parse('(cn=Steve)');

$condition = $nodes[0];

$condition->getAttribute(); // "cn"
$condition->getOperator(); // "="
$condition->getValue(); // "Steve"
```

When group filters have been detected, you will recieve a `GroupNode` instead.

With a `GroupNode`, you can retrieve all nested nodes via the `getNodes()` method:

```php
// array: [
//  0 => LdapRecord\Query\Filter\GroupNode
// ]
$nodes = Parser::parse('(&(cn=Steve)(sn=Bauman))');

$group = $nodes[0];

$group->getOperator(); // "&"

// array: [
//  0 => LdapRecord\Query\Filter\ConditionNode
//  1 => LdapRecord\Query\Filter\ConditionNode
// ]
$group->getNodes();
```

> **Important**: The parser will always return an array of `Node` instances.

## Parsing From User Input

If you're accepting user input input to parse, make sure you use a try/catch
block to catch any potential `ParserException` that may be thrown:

```php
$input = '(&(cn=Steve)(sn=Bauman))(mail=sbauman@local.com';

try {
    $nodes = Parser::parse($input);
} catch (\LdapRecord\Query\Filter\ParserException $e) {
    $e->getMessage(); // "Unclosed filter group. Missing ")" parenthesis"
}
```

## Parsing Bad Filters

The filter parser should not be considered as a filter validator. Filters
that would otherwise fail to execute on an LDAP server can still be parsed.

For example, this filter that would otherwise fail due to not being enclosed
by a surrounding and/or (`&` / `|`) statement, can still be parsed by the filter parser:

```php
// array: [
//  0 => LdapRecord\Query\Filter\ConditionNode
//  1 => LdapRecord\Query\Filter\ConditionNode
// ]
$result = Parser::parse('(cn=Steve)(sn=Bauman)');
```

As you can see, an array of nodes is returned, allowing you to parse each nested node individually.

## Assembling Nodes

The filter parser can also re-assemble nodes into their string based format. This
can help when you want to process a filter to remove any unneeded spacing:

```php
$nodes = Parser::parse('(&  (cn=Steve   ) ( sn= Bauman) )  ');

// Returns: "(&(cn=Steve)(sn= Bauman))"
$filter = Parser::assemble($nodes);
```

> **Important**: As you can see above, the parser will not trim spaces
> inside of condition values, in order to preserve the true value.

## Display Filter Tree

If you're looking to display a tree of parsed LDAP filters, here's a recursive function to get you started:

```php
use LdapRecord\Query\Filter\Parser;
use LdapRecord\Query\Filter\GroupNode;
use LdapRecord\Query\Filter\ConditionNode;

function tree($node)
{
	if ($node instanceof GroupNode) {
        return "<ul>
            <li>
                {$node->getOperator()}
                
                <ul>" . tree($node->getNodes()) . "</ul>
            </li>
        </ul>";
    }
  
  	if ($node instanceof ConditionNode) {
        return "<li>{$node->getAttribute()} {$node->getOperator()} {$node->getValue()}</li>";
    }

    if (is_array($node)) {
        return array_reduce($node, function ($carry, $node) {
            return $carry .= tree($node);
        });
    }
};

$input = '(|(&(cn=Steve)(sn=Bauman))(mail=sbauman@local.com))';

$group = Parser::parse($input);

echo tree($group);

// Result:
// <ul>
//   <li>
//       |
//       <ul>
//         <ul>
//             <li>
//               &
//               <ul>
//                   <li>cn = Steve</li>
//                   <li>sn = Bauman</li>
//               </ul>
//             </li>
//         </ul>
//       </ul>
//   </li>
// </ul>
```

## Available Methods

### `LdapRecord\Query\Filter\Parser`

```php
Parser::parse($filter); // (ConditionNode|GroupNode)[]

Parser::assemble($nodes); // string
```

### `LdapRecord\Query\Filter\ConditionNode`

```php
$condition->getAttribute(); // string
$condition->getOperator(); // string
$condition->getValue(); // string
$condition->getRaw(); // string
```

### `LdapRecord\Query\Filter\GroupNode`
```php
$group->getOperator(); // string ("&", "|", "!")
$group->getNodes(); // (ConditionNode|GroupNode)[]
$group->getRaw(); // string
```
