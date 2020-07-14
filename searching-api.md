---
title: Searching API
description: A list of all available LdapRecord query builder methods.
extends: _layouts.documentation
section: content
---

Testing
# Available Search Methods (API)

<div class="api-method-list">
<p>
<a href="#addControl">addControl</a>
<a href="#addFilter">addFilter</a>
<a href="#addSelect">addSelect</a>
<a href="#andFilter">andFilter</a>
<a href="#cache">cache</a>
<a href="#clearFilters">clearFilters</a>
<a href="#delete">delete</a>
<a href="#deleteAttributes">deleteAttributes</a>
<a href="#escape">escape</a>
<a href="#find">find</a>
<a href="#findBy">findBy</a>
<a href="#findByOrFail">findByOrFail</a>
<a href="#findManyBy">findManyBy</a>
<a href="#findOrFail">findOrFail</a>
<a href="#first">first</a>
<a href="#firstOrFail">firstOrFail</a>
<a href="#get">get</a>
<a href="#getCache">getCache</a>
<a href="#getConnection">getConnection</a>
<a href="#getDn">getDn</a>
<a href="#getFilters">getFilters</a>
<a href="#getGrammar">getGrammar</a>
<a href="#getQuery">getQuery</a>
<a href="#getSelects">getSelects</a>
<a href="#getType">getType</a>
<a href="#getUnescapedQuery">getUnescapedQuery</a>
<a href="#hasControl">hasControl</a>
<a href="#hasSelects">hasSelects</a>
<a href="#in">in</a>
<a href="#insert">insert</a>
<a href="#insertAttributes">insertAttributes</a>
<a href="#isNested">isNested</a>
<a href="#isPaginated">isPaginated</a>
<a href="#limit">limit</a>
<a href="#listing">listing</a>
<a href="#model">model</a>
<a href="#nested">nested</a>
<a href="#newInstance">newInstance</a>
<a href="#newNestedInstance">newNestedInstance</a>
<a href="#notFilter">notFilter</a>
<a href="#orFilter">orFilter</a>
<a href="#orWhere">orWhere</a>
<a href="#orWhereApproximatelyEquals">orWhereApproximatelyEquals</a>
<a href="#orWhereContains">orWhereContains</a>
<a href="#orWhereEndsWith">orWhereEndsWith</a>
<a href="#orWhereEquals">orWhereEquals</a>
<a href="#orWhereHas">orWhereHas</a>
<a href="#orWhereNotContains">orWhereNotContains</a>
<a href="#orWhereNotEndsWith">orWhereNotEndsWith</a>
<a href="#orWhereNotEquals">orWhereNotEquals</a>
<a href="#orWhereNotHas">orWhereNotHas</a>
<a href="#orWhereNotStartsWith">orWhereNotStartsWith</a>
<a href="#orWhereRaw">orWhereRaw</a>
<a href="#orWhereStartsWith">orWhereStartsWith</a>
<a href="#paginate">paginate</a>
<a href="#query">query</a>
<a href="#rawFilter">rawFilter</a>
<a href="#read">read</a>
<a href="#recursive">recursive</a>
<a href="#rename">rename</a>
<a href="#select">select</a>
<a href="#setCache">setCache</a>
<a href="#setConnection">setConnection</a>
<a href="#setDn">setDn</a>
<a href="#setGrammar">setGrammar</a>
<a href="#update">update</a>
<a href="#updateAttributes">updateAttributes</a>
<a href="#where">where</a>
<a href="#whereApproximatelyEquals">whereApproximatelyEquals</a>
<a href="#whereBetween">whereBetween</a>
<a href="#whereContains">whereContains</a>
<a href="#whereDeleted">whereDeleted</a>
<a href="#whereEndsWith">whereEndsWith</a>
<a href="#whereEquals">whereEquals</a>
<a href="#whereHas">whereHas</a>
<a href="#whereIn">whereIn</a>
<a href="#whereNotContains">whereNotContains</a>
<a href="#whereNotEndsWith">whereNotEndsWith</a>
<a href="#whereNotEquals">whereNotEquals</a>
<a href="#whereNotHas">whereNotHas</a>
<a href="#whereNotStartsWith">whereNotStartsWith</a>
<a href="#whereRaw">whereRaw</a>
<a href="#whereStartsWith">whereStartsWith</a>
<a href="#withDeleted">withDeleted</a>
</p>
</div>

## Method Listing

#### `addControl` {#addControl}

Add a server control to be executed with the LDAP search query:

```php
$query = $connection->query();

$query->addControl(
    $oid = '1.2.840.113556.1.4.417', $isCritical = true, $value = null
);

// array:1 [▼
//  "1.2.840.113556.1.4.417" => array:3 [▼
//    "oid" => "1.2.840.113556.1.4.417"
//    "isCritical" => true
//    "value" => null
//  ]
// ]
var_dump($query->controls);
```

#### `addFilter` {#addFilter}

Add a filter with its bindings to the query:

> Available types are `and`, `or` and `raw`.

```php
$query = $connection->query();

$bindings = [
    'field' => 'cn',
    'operator' => '=',
    'value' => 'Steve Bauman',
];

$query->addFilter($type = 'and', $bindings);
```

#### `addSelect` {#addSelect}

Add an attribute to be selected for the query:

```php
$query = $connection->query();

// Using arguments:
$query->addSelect('cn');
$query->addSelect('foo', 'bar', 'baz');

// Using an array:
$query->addSelect(['sn', 'givenname']);

// array:7 [▼
//   0 => "cn"
//   1 => "foo"
//   2 => "bar"
//   3 => "baz"
//   4 => "sn"
//   5 => "givenname"
//   6 => "objectclass"
// ]
var_dump($query->getSelects());
```

#### `andFilter` {#addFilter}

Add a nested "and" filter to the query:

```php
$query = $connection->query();

$query->andFilter(function (\LdapRecord\Query\Builder $q) {
    $q->where('foo', '=', 'bar');
    $q->where('baz', '=', 'zal');
});

// "(&(foo=bar)(baz=zal))"
echo $query->getUnescapedQuery();
```

#### `cache` {#cache}

Cache the executed query until the given date has passed:

> Pass `true` as the second argument to force flush the cache if the query has been executed before.

```php
$query = $connection->query();

$until = new \DateTime('+1 day');

$query->cache($until, $flush = false);
```

#### `clearFilters` {#clearFilters}

Reset / clear all filters that have been added to the query:

```php
$query = $connection->query();

$query->where('foo', '=', 'bar');

$query->clearFilters();
```

#### `delete` {#delete}

Delete an entry from the directory:

```php
$query = $connection->query();

$query->delete('cn=John Doe,ou=Users,dc=local,dc=com');
```

#### `deleteAttributes` {#deleteAttributes}

Delete an attributes values from the directory:

```php
$query = $connection->query();

$entry = 'cn=Accounting Users,ou=Groups,dc=local,dc=com';

// Delete all values from an attribute, for example,
// removing all members from a particular group:
$query->deleteAttributes($entry, ['member' => []]);

// Delete a specific value from an attribute, for example,
// removing a specific member from a particular group:
$member = 'cn=John Doe,ou=Users,dc=local,dc=com';

$query->deleteAttributes($entry, ['member' => [$member]]);
```

#### `escape` {#escape}

Prepare a value to be escaped:

> This method accepts the same parameters as the built in PHP
> [`ldap_escape` method](https://www.php.net/manual/function.ldap-escape.php).

```php
$query = $connection->query();

// Returns instance of:
// LdapRecord\Models\Attributes\EscapedValue
$value = $query->escape('value', $ignore = '', $flags = 0);

// Prepare the value to be escaped for use in a distinguished name:
$value->dn();

// Prepare the value to be escaped for use in a filter:
$value->filter();

// Prepare the value to be escaped for use in a distinguished name and filter:
$value->both();

// Set the characters to ignore:
$value->ignore('/*');

// Get the escaped value:
$value->get();

// Can also be casted to string:
(string) $value;
```

#### `find` {#find}

Find an entry in the directory by its distinguished name:

```php
$query = $connection->query();

if ($entry = $query->find('cn=John Doe,dc=local,dc=com')) {
    // Found entry!
} else {
    // Not found.
}
```

#### `findBy` {#findBy}

Find the first matching entry in the directory by the given attribute and value:

```php
$query = $connection->query();

if ($entry = $query->findBy('samaccountname', 'johndoe')) {
    // Found entry!
} else {
    // Not found.
}
```

#### `findByOrFail` {#findByOrFail}

Find the first matching entry in the directory by the given attribute and value **or fail**:

```php
$query = $connection->query();

try {
    $entry = $query->findByOrFail('samaccountname', 'johndoe');
} catch (\LdapRecord\Models\ModelNotFoundException $ex) {
    // Not found.
}
```

#### `findManyBy` {#findManyBy}

Find may entries in the directory by the given attribute and values:

```php
$query = $connection->query();

$entries = $query->findManyBy('samaccountname', ['johndoe', 'janedoe', 'sbauman']);
```

#### `findOrFail` {#findOrFail}

Find an entry in the directory by its distinguished name **or fail**:

```php
$query = $connection->query();

try {
    $entry = $query->findOrFail('cn=John Doe,dc=local,dc=com');
} catch (\LdapRecord\Models\ModelNotFoundException $ex) {
    // Not found.
}
```

#### `first` {#first}

Get the first resulting entry of a query from the directory:

```php
$query = $connection->query();

$entry = $query->whereStartsWith('cn', 'Steve')->first();
```

#### `firstOrFail` {#firstOrFail}

Get the first resulting entry of a query from the directory **or fail**:

```php
$query = $connection->query();

try {
    $entry = $query->whereStartsWith('cn', 'Steve')->first();
} catch (\LdapRecord\Models\ModelNotFoundException $ex) {
    // Not entries returned.
}
```

#### `get` {#get}

Get the resulting entries of a query from the directory:

> **Important**: If you expect to have more than 1000 entries
> returned from your query, use the [paginate](#paginate)
> method instead, which will return all entries.

```php
$query = $connection->query();

$entries = $query->where('company', '=', 'Acme')->get();
```

#### `getCache` {#getCache}

Get the query cache (if set):

```php
$query = $connection->query();

// Returns null or instance of LdapRecord\Query\Cache:
$cache = $query->getCache();
```

#### `getConnection` {#getConnection}

Get the underlying connection the query is executing on:

```php
$query = $connection->query();

// Returns instance of LdapRecord\Connection:
$conn = $query->getConnection();
```

#### `getDn` {#getDn}

Get the base Distinguished Name that the query is executing on:

```php
$query = $connection->query();

$query->setDn('ou=Users,dc=local,dc=com');

// Returns 'ou=Users,dc=local,dc=com':
$base = $query->getDn();
```

#### `getFilters` {#getFilters}

Get the filters that have been added to the query:

```php
$query = $connection->query();

$query->where('company', '=', 'Acme');

// array:3 [▼
//   "and" => array:1 [▼
//     0 => array:3 [▼
//       "field" => "company"
//       "operator" => "="
//       "value" => LdapRecord\Models\Attributes\EscapedValue {#17 ▶}
//     ]
//   ]
//   "or" => []
//   "raw" => []
// ]
var_dump($query->getFilters());
```

#### `getGrammar` {#getGrammar}

Get the underlying query grammar instance:

```php
$query = $connection->query();

// Returns instance of LdapRecord\Query\Grammar:
$grammar = $query->getGrammar();
```

#### `getQuery` {#getQuery}

Get the raw, **escaped** LDAP query filter:

```php
$query = $connection->query();

$query->where('company', '=', 'Acme');

// Returns '(company=\41\63\6d\65)'
$filter = $query->getQuery();
```

#### `getSelects` {#getSelects}

Get the selected attributes of the query:

> **Important**: `objectclass` will always be included in the returned array.

```php
$query = $connection->query();

// array:1 [▼
//   0 => "objectclass"
// ]
var_dump($query->getSelects());

$query->select(['cn', 'mail', 'givenname']);

// array:4 [▼
//   0 => "cn"
//   1 => "mail"
//   2 => "givenname"
//   3 => "objectclass"
// ]
var_dump($query->getSelects());
```

#### `getType` {#getType}

Get the type of LDAP query being executed, either `search`, `listing` or `read`:

```php
$query = $connection->query();

// Returns 'search':
$query->getType();

// Returns 'listing':
$query->listing()->getType();

// Returns 'read':
$query->read()->getType();
```

#### `getUnescapedQuery` {#getUnescapedQuery}

Get the raw, **unescaped** LDAP query filter:

```php
$query = $connection->query();

$query->where('company', '=', 'Acme');

// Returns '(company=Acme)'
$filter = $query->getUnescapedQuery();
```

#### `hasControl` {#hasControl}

Determine if the query has a specific LDAP control OID added:

```php
$query = $connection->query();

if ($query->hasControl($oid = '1.2.840.113556.1.4.417')) {
    // The query has a control added for the OID.
}
```

#### `hasSelects` {#hasSelects}

Determine if the query has any selects added:

```php
$query = $connection->query();

// Returns false:
$query->hasSelects();

$query->select(['cn', 'sn']);

// Returns true:
$query->hasSelects();
```

#### `in` {#in}

Sets the base Distinguished Name to perform a search upon.

Alias for the [setDn](#setDn) method.

```php
$query = $connection->query();

// Get all entries below the 'Users' OU:
$query->in('ou=Users,dc=local,dc=com')->get();
```

#### `insert` {#insert}
#### `insertAttributes` {#insertAttributes}
#### `isNested` {#isNested}
#### `isPaginated` {#isPaginated}
#### `limit` {#limit}
#### `listing` {#listing}
#### `model` {#model}
#### `nested` {#nested}
#### `newInstance` {#newInstance}
#### `newNestedInstance` {#newNestedInstance}
#### `notFilter` {#notFilter}
#### `orFilter` {#orFilter}
#### `orWhere` {#orWhere}
#### `orWhereApproximatelyEquals` {#orWhereApproximatelyEqual}
#### `orWhereContains` {#orWhereContains}
#### `orWhereEndsWith` {#orWhereEndsWith}
#### `orWhereEquals` {#orWhereEquals}
#### `orWhereHas` {#orWhereHas}
#### `orWhereNotContains` {#orWhereNotContains}
#### `orWhereNotEndsWith` {#orWhereNotEndsWith}
#### `orWhereNotEquals` {#orWhereNotEquals}
#### `orWhereNotHas` {#orWhereNotHas}
#### `orWhereNotStartsWith` {#orWhereNotStartsWith}
#### `orWhereRaw` {#orWhereRaw}
#### `orWhereStartsWith` {#orWhereStartsWith}
#### `paginate` {#paginate}
#### `query` {#query}
#### `rawFilter` {#rawFilter}
#### `read` {#read}
#### `recursive` {#recursive}
#### `rename` {#rename}
#### `select` {#select}
#### `setCache` {#setCache}
#### `setConnection` {#setConnection}
#### `setDn` {#setDn}

Sets the base Distinguished Name to perform a search upon.

```php
$query = $connection->query();

// Get all entries below the 'Users' OU:
$query->setDn('ou=Users,dc=local,dc=com')->get();
```

#### `setGrammar` {#setGrammar}
#### `update` {#update}
#### `updateAttributes` {#updateAttributes}
#### `where` {#where}
#### `whereApproximatelyEquals` {#whereApproximatelyEquals}
#### `whereBetween` {#whereBetween}
#### `whereContains` {#whereContains}
#### `whereDeleted` {#whereDeleted}
#### `whereEndsWith` {#whereEndsWith}
#### `whereEquals` {#whereEquals}
#### `whereHas` {#whereHas}
#### `whereIn` {#whereIn}
#### `whereNotContains` {#whereNotContains}
#### `whereNotEndsWith` {#whereNotEndsWith}
#### `whereNotEquals` {#whereNotEquals}
#### `whereNotHas` {#whereNotHas}
#### `whereNotStartsWith` {#whereNotStartsWith}
#### `whereRaw` {#whereRaw}
#### `whereStartsWith` {#whereStartsWith}
#### `withDeleted` {#withDeleted}
