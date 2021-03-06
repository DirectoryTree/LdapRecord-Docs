---
title: Searching
description: Searching your directory using LdapRecord
---

# Searching

## Introduction

Using the LdapRecord query builder makes building LDAP queries feel effortless.

It allows you to generate LDAP filters using a fluent and
convenient interface, similar to Eloquent in Laravel.

> The LdapRecord query builder escapes all fields & values
> given to its `where()` methods. There is no need to clean or
> escape strings before passing them into the query builder.

## Creating a new Query

To create a new search query, call the `query()` method on your `Connection` instance:

```php
$query = $connection->query();
```

Or you can chain all your methods if you'd prefer:

```php
$results = $connection->query()->where('cn', '=', 'John Doe')->get();
```

> Querying your LDAP connection manually will return raw LDAP results
> in a `Collection`. You must query using [models](/docs/core/v1/models#retrieving-models)
> themselves if you would like them to be returned instead.

## Selects

> Fields are case in-sensitive. For example, you can
> insert `CN`, `cn` or `cN`, they will return the same result.

Selecting only the LDAP attributes you need will increase the speed of your queries.

```php
// Passing in an array of attributes
$query->select(['cn', 'samaccountname', 'telephone', 'mail']);

// Passing in each attribute as an argument
$query->select('cn', 'samaccountname', 'telephone', 'mail');
```

## Executing Searches

#### Finding a record

If you're trying to find a single record, you must use the `find()` method
and insert the distinguished name of the record you are looking for:

```php
$record = $query->find('cn=John Doe,dc=acme,dc=org');

if ($record) {
    // Record was found!
} else {
    // Hmm, looks like we couldn't find anything...
}
```

#### Finding a record (or failing)

If you'd like to try and find a single record and throw an exception when
it hasn't been found, use the `findOrFail()` method:

```php
try {
    $record = $query->findOrFail('cn=John Doe,dc=acme,dc=org');
} catch (LdapRecord\Models\ModelNotFoundException $e) {
    // Record wasn't found!
}
```

#### Finding a record by a specific attribute

If you're looking for a single record with a specific attribute, use the `findBy()` method:

```php
// We're looking for a record with the 'samaccountname' of 'jdoe'.
$record = $query->findBy('samaccountname', 'jdoe');
```

You can also use `findByOrFail()` to generate an exception when a record is not found.

#### Retrieving results

To get the results from a search, simply call the `get()` method:

```php
$results = $query->select(['cn', 'samaccountname'])->get();
```

Results will be returned inside of an `LdapRecord\Query\Collection` instance.

#### Retrieving the first record

To retrieve the first record of a search, call the `first()` method:

```php
$record = $query->first();
```

Results will return **the model instance only**.

You can also use `firstOrFail()` to generate an exception when no records are found.

## Limit

To limit the results records returned from your LDAP server and increase the
speed of your queries, you can use the `limit()` method:

```php
// This will only return 5 records that contain the name of 'John':
$records = $query->where('cn', 'contains', 'John')->limit(5)->get();
```

## Wheres

To perform a where clause on the search object, use the `where()` function:

```php
$query->where('cn', '=', 'John Doe');
```

This query would look for a record with the common name of 'John Doe' and return the results.

We can also perform a 'where equals' without including the operator:

```php
$query->whereEquals('cn', 'John Doe');
```

We can also supply an array of key - value pairs to quickly add multiple wheres:

```php
$wheres = [
    'givenname' => 'John',
    'company'   => 'Acme',
];

$query->where($wheres);
```

Or, if you require conditionals, you can quickly add multiple wheres with nested arrays:

```php
$query->where([
   ['cn', '=', 'John Doe'],
   ['manager', '!', 'Suzy Doe'],
]);
```

#### Where Starts With

We could also perform a search for all objects beginning with the common name of 'John' using the `starts_with` operator:

```php
$results = $query->where('cn', 'starts_with', 'John')->get();

// Or:

$results = $query->whereStartsWith('cn', 'John')->get();
```

#### Where Ends With

We can also search for all objects that end with the common name of `Doe` using the `ends_with` operator:

```php
$results = $query->where('cn', 'ends_with', 'Doe')->get();

// Or:

$results = $query->whereEndsWith('cn', 'Doe')->get();
```

#### Where Between

To search for records between two values, use the `whereBetween` method.

For the example below, we'll retrieve all records that were created between two dates:

```php
$from = (new DateTime('October 1st 2016'))->format('YmdHis.0\Z');
$to = (new DateTime('January 1st 2017'))->format('YmdHis.0\Z');

$users = $query->whereBetween('whencreated', [$from, $to])->get();
```

#### Where Contains

We can also search for all objects with a common name that contains `John Doe` using the `contains` operator:

```php
$results = $query->where('cn', 'contains', 'John Doe')->get();

// Or:

$results = $query->whereContains('cn', 'John Doe')->get();
```

#### Where Not Contains

You can use a 'where not contains' to perform the inverse of a 'where contains':

```php
$results = $query->where('cn', 'not_contains', 'John Doe')->get();

// Or:

$results = $query->whereNotContains('cn', 'John Doe');
```

#### Where Has

Or we can retrieve all objects that have a common name attribute using the wildcard operator (`*`):

```php
$results = $query->where('cn', '*')->get();

// Or:

$results = $query->whereHas('cn')->get();
```

This type of filter syntax allows you to clearly see what your searching for.

#### Where Not Has

You can use a 'where not has' to perform the inverse of a 'where has':

```php
$results = $query->where('cn', '!*')->get();

// Or:

$results = $query->whereNotHas($field)->get();
```

#### Where Deleted

To retrieve **only** deleted models from your LDAP server, use the `whereDeleted` method:

```php
$results = $query->whereDeleted()->get();
```

If you would like to **include** deleted models from your LDAP
server in your query results, use the `withDeleted` method:

```php
$results = $query->withDeleted()->get();
```

## Or Wheres

To perform an `or where` clause on the search object, use the `orWhere()` method.

For example:

```php
$results = $query->where('cn', '=', 'John Doe')
                 ->orWhere('cn', '=', 'Suzy Doe')
                 ->get();
```

This query will return objects that have the common name of `John Doe` or `Suzy Doe`.

> You can also use all `where` methods as an or where, for example:
> `orWhereHas()`, `orWhereContains()`, `orWhereStartsWith()`, `orWhereEndsWith()`

## Dynamic Wheres

To perform a dynamic where, simply suffix a `where` with the field you're looking for.

This feature was directly ported from Laravel's Eloquent.

Here's an example:

```php
// This query:
$result = $query->where('cn', '=', 'John Doe')->first();

// Can be converted to:
$result = $query->whereCn('John Doe')->first();
```

You can perform this on **any** attribute:

```php
$result = $query->whereTelephonenumber('555-555-5555')->first();
```

You can also chain them:

```php
$result = $query->whereTelephonenumber('555-555-5555')
                ->whereGivenname('John Doe')
                ->whereSn('Doe')
                ->first();
```

You can even perform multiple dynamic wheres by separating your fields by an `And`:

```php
// This would perform a search for a user with the
// first name of 'John' and last name of 'Doe'.
$result = $query->whereGivennameAndSn('John', 'Doe')->first();
```

## Nested Filters

By default, the LdapRecord query builder automatically wraps your queries in `and` / `or` filters for you.
However, if any further complexity is required, nested filters allow you
to construct any query fluently and easily.

#### andFilter

The `andFilter` method accepts a closure which allows you to construct a query inside of an `and` LDAP filter:

```php
// Creates the filter: (&(givenname=John)(sn=Doe))
$results = $query->andFilter(function (LdapRecord\Query\Builder $q) {
    $q->where('givenname', '=', 'John')
      ->where('sn', '=', 'Doe');
})->get();
```

The above query would return records that contain the first name `John` **and** the last name `Doe`.

#### orFilter

The `orFilter` method accepts a closure which allows you to construct a query inside of an `or` LDAP filter:

```php
// Creates the filter: (|(givenname=John)(sn=Doe))
$results = $query->orFilter(function (LdapRecord\Query\Builder $q) {
    $q->where('givenname', '=', 'John')
      ->where('sn', '=', 'Doe');
})->get();
```

The above query would return records that contain the first name `John` **or** the last name `Doe`.

#### notFilter

The `notFilter` method accepts a closure which allows you to construct a query inside a `not` LDAP filter:

```php
// Creates the filter: (!(givenname=John)(sn=Doe))
$results = $query->notFilter(function (LdapRecord\Query\Builder $q) {
    $q->where('givenname', '=', 'John')
      ->where('sn', '=', 'Doe');
})->get();
```

The above query would return records that **do not** contain the first name `John` **or** the last name `Doe`.

#### Complex Nesting

The above methods `andFilter` / `orFilter` can be chained together and nested
as many times as you'd like for larger complex queries:

```php
$query = $query->orFilter(function (LdapRecord\Query\Builder $q) {
    $q->where('givenname', '=', 'John')->where('sn', '=', 'Doe');
})->andFilter(function (LdapRecord\Query\Builder $q) {
    $q->where('department', '=', 'Accounting')->where('title', '=', 'Manager');
})->getUnescapedQuery();

echo $query; // Returns '(&(|(givenname=John)(sn=Doe))(&(department=Accounting)(title=Manager)))'
```

## Raw Filters

> Raw filters are not escaped. **Do not** accept user input into the raw filter method.

Sometimes you might just want to add a raw filter without using the query builder.
You can do so by using the `rawFilter()` method:

```php
$results = $query->rawFilter('(samaccountname=jdoe)')->get();

// Or use an array
$filters = [
    '(samaccountname=jdoe)',
    '(surname=Doe)',
];

$results = $query->rawFilter($filters)->get();

// Or use multiple arguments
$results = $query->rawFilter($filters[0], $filters[1])->get();

// Multiple raw filters will be automatically wrapped into an `and` filter:
$query = $query->getUnescapedQuery();

echo $query; // Returns (&(samaccountname=jdoe)(surname=Doe))
```

## Paginating

Paginating your search results will allow you to return more results than your LDAP cap (usually 1000).

For example, if your LDAP server contains 10,000 records and you paginate by 1000, 10 queries will be executed.

> Calling `paginate()` will retrieve **all** records from your LDAP server for the current query.
> Be careful with large result sets, as you may run out of memory.

```php
// Perform global "all" search, paginating by 1000 records:
$results = $query->paginate(1000);

foreach ($results as $result) {
    //
}
```

## Base DN

To set the base DN of your search you can use one of two methods:

```php
// Using the `in()` method:
$results = $query->in('ou=Accounting,dc=acme,dc=org')->get();

// Using the `setDn()` method:
$results = $query->setDn('ou=Accounting,dc=acme,dc=org')->get();
```

Either option will return the same results. Use which ever method you prefer to be more readable.

## Root DSE

To fetch the [Root DSE](https://ldapwiki.com/wiki/RootDSE) object in your directory, execute the below query:

```php
$rootDse = $query->query()
    ->in(null)
    ->read()
    ->whereHas('objectclass')
    ->first();
```

## Search Options

#### Recursive

By default, all searches performed are recursive.

If you'd like to disable recursive search and perform a single level search, use the `listing()` method:

```php
$result = $query->listing()->get();
```

This would perform an `ldap_listing()` instead of an `ldap_search()`.

#### Read

If you'd like to perform a read instead of a listing or a recursive search, use the `read()` method:

```php
$result = $query->read()->where('objectClass', '*')->get();
```

This would perform an `ldap_read()` instead of an `ldap_listing()` or an `ldap_search()`.

> Performing a `read()` will always return _one_ record in your result.

#### Custom Controls

If you'd like to [add server controls](https://www.php.net/manual/en/ldap.controls.php) to your query, use the `addControl` method:

```php
$result = $query->addControl('1.2.840.113556.1.4.417', $isCritical = true)->get();
```

## Retrieving the ran query

If you'd like to retrieve the current query to save or run it at another
time, use the `getQuery()` method on the query builder.

This will return the escaped filter.

```php
$query = $query->where('cn', '=', 'John Doe')->getQuery();

echo $query; // Returns '(cn=\4a\6f\68\6e\20\44\6f\65)'
```

To retrieve the unescaped filter, call the `getUnescapedQuery()` method:

```php
$query = $query->where('cn', '=', 'John Doe')->getUnescapedQuery();

echo $query; // Returns '(cn=John Doe)'
```

Now that you know how to search your directory, lets move onto [creating / modifying LDAP records](/docs/core/v1/models).
