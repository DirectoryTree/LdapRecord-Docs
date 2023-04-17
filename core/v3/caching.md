---
title: Query Caching
description: Using LdapRecord query caching
---

# Query Caching

## Introduction

LdapRecord supports caching LDAP search operations. This is useful when running
expensive queries. A `pagination` query on the root of your LDAP directory
may take several seconds to complete.

Caching allows you to store the entire result so it is not retrieved again
from the server until the cache is set to expire.

## Requirements

Your application must have a caching implementation that uses the
[PSR Simple Cache](https://github.com/php-fig/simple-cache) interface.

For example, the Laravel cache system implements this interface.

## Getting Started

To setup caching, you must add your cache implementation to your LDAP connection instance.

This is done by the `setCache` method:

```php
use LdapRecord\Connection;

$connection = new Connection(['...']);

$connection->setCache($myAppCache);
```

## Usage

Once you've added your caching implementation to your LdapRecord connection, you
can start caching queries on plain queries or queries on LdapRecord models.

This is done via the `cache` method. This method accepts two parameters.

The first is a `DateTimeInterface` - which is an instance of a PHP Date
object set in the future as to when the cache for the query must expire.

The second is a boolean indicating whether the cache must be flushed
prior to running the query. This allows you to have control over
manually cache flushing for the query.

```php
$until = new \DateTime('tomorrow');

$results = $connection->query()->cache($until)->paginate();

// Manually flushing the cache...
$results = $connection->query()->cache($until, $flush = true)->paginate();
```

The above example will cache results at the time of running until the next the day.
Once the existing cache has expired, it will be re-cached again.

To cache model query results, call the same method upon your model query:

```php
use LdapRecord\Models\ActiveDirectory\User;

$until = new \DateTime('tomorrow');

$users = User::cache($until)->get();
```
