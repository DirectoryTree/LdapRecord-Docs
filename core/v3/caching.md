---
title: Query Caching
description: Using LdapRecord query caching
---

# Query Caching

## Introduction

LdapRecord supports caching LDAP search operations. This is useful when running
expensive queries. A `pagination` query on the root of your LDAP directory
may take several seconds to complete.

Caching allows you to store the entire result so it is not retrieved 
again from the server until the cache is set to expire.

## Requirements

Your application must have a caching implementation that uses the
[PSR Simple Cache](https://github.com/php-fig/simple-cache) interface.

For example, the Laravel cache system implements this interface.

## Getting Started

To set up caching, you must add your cache implementation to your LDAP connection instance.

This is done by the `setCache` method:

```php
use LdapRecord\Connection;

$connection = new Connection(['...']);

$connection->setCache($myAppCache);
```

## Usage

Once you've added your caching implementation to your LdapRecord 
connection, you may start caching queries on plain queries or 
queries on created on LdapRecord models.

Caching is enabled for a single query using the `cache()` method:

```php
$until = new \DateTime('tomorrow');

$results = $connection->query()->cache($until)->get();
```

In the above example, after retrieving the results from the server, they will be 
stored in your cache. Upon subsequent attempts, cached results will be returned
until they expire using the initial expiry date that was given.

> Queries are cached by generating a unique identifier associated with the query 
> instance. This identifier is built with the following components, to ensure 
> other queries executed do not call upon the same cache results:
>
> - Query limit
> - Query filter
> - Base DN of the query
> - Selected query attributes
> - LDAP Host Name / IP address
> - LDAP Search Type (Search/Listing/Read)
> - Whether the query should return paginated results.

### Manual Flushing

In circumstances where you would like to manually flush a query cache, 
you may pass in `true` into the second parameter (or via the `flush` typed parameter) which enforces a 
removal any cached results owned by the query:

```php
$results = $connection->query()->cache($until, flush: true)->get();
```

### Storing Indefinitely

If you would like to store a query's cache results indefinitely, omit a date from the cache:

```php
$results = $connection->query()->cache()->get();
```

### Custom Cache Keys

If you would like to store a query's cache results using a custom cache key, 
you may provide a key into the third parameter (or via the `key` typed 
parameter), without passing in a date:

```php
$results = $connection->query()->cache(new \DateTime('tomorrow'), key: 'users')->get();

// Storing indefinitely with a custom key:
$connection->query()->cache(key: 'users')->get();
```

Then, you can retrieve or flush the results outside the query instance if needed:

```php
use LdapRecord\Container;

$cache = $connection->getCache();

// Get the cached results:
$results = $cache->get('users');

// Delete the cached results:
$cache->delete('users');
```

> It's imperitive to know that results stored in the cache from the query builder
> are the raw results returned from your LDAP server. Model instances themselves
> (or references to them) for example, are not stored in the cache.
