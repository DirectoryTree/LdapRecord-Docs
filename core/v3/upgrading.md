---
title: Upgrade to v3
description: Upgrading from LdapRecord v2
---

# Upgrading to Version 3

## High Impact Changes

### Updating Dependencies

#### PHP 8.1.0 Required

LdapRecord v3 now requires PHP 8.1.0 or greater.

#### Composer Dependencies

You should update the following dependency in your application's `composer.json` file:

```json
"directorytree/ldaprecord": "^3.0"
```

### Strict PHP Types Implemented

LdapRecord now has strict types implemented in all classes for all methods and properties.

If you've created your own [models](/docs/core/v3/models), [scopes](/docs/core/v3/model-scopes), 
or have extended any other LdapRecord class, you will need to adjust any overridden 
properties or methods with their respective types.

## Medium Impact Changes

### LdapRecord\Container `manager` Method Renamed

An `LdapRecord\Conatiner::manager()` method has been renamed to `getConnectionManager()`.

### LdapRecord\ConnectionManager Proxy Methods Removed

The below `LdapRecord\ConnectionManager` proxy methods have been removed. 

You may replace these methods with their listed equivalent:

| Proxy Method Removed   | Replacement         |
|------------------------|---------------------|
| `reset()`              | `flush()`           |
| `allConnections()`     | `getConnnections()` |
| `getEventDispatcher()` | `getDisptacher()`   |
| `setEventDispatcher()` | `setDispatcher()`   |

### LdapRecord\Utilities Removed

The `LdapRecord\Utilities` class has been removed in favor of
individual utility classes that provide the same features and functions.

Please see the [helpers](/docs/core/v3/helpers) documentation for their equivalent utility classes.

### Tightenco/Collect Replaced

The `tightenco/collect` package has now been replaced with Laravel's core 
`illuminate/collections` package, that became available in Laravel 8.0.

Due to this change, if you're installing LdapRecord in a Laravel 
application, it must be at least version 8.0.0 or greater.

### LdapRecord\Query\Builder methods renamed

The below query builder methods have been renamed to more 
clearly convey the LDAP action that they will execute:

| From                | To                   |
|---------------------|----------------------|
| `listing()`         | `list()`             |
| `createAttribute()` | `addAttribute()`     |
| `updateAttribute()` | `replaceAttribute()` |
| `deleteAttribute()` | `removeAttributes()` |
| N\A                 | `removeAttribute()`  |

## Low Impact Changes

### LdapRecord\Ldap Method Return Types Changed

#### `bind`

The `Ldap::bind()` method now returns an `LdapRecord\LdapResultRepsonse`, 
which provides a class for interacting with an LDAP response in detail:

```php
$connection = new \Ldap\Connection(['...']);

$response = $connection->getLdapConnection()->bind($username, $password);

$response->errorCode; // int
$response->matchedDn; // string|null
$response->errorMessage; // string|null
$response->referrals; // array|null
$response->controls; // array|null

$response->successful(); // bool
$response->failed(); // bool
```

#### `parseResult`

The `Ldap::parseResult()` method now returns an `LdapRecord\LdapResultRepsonse`, 
as with the `bind()` method mentioned above.

### LdapRecord\Models\Relations\OneToMany Method Return Types Changed

The methods listed below now no longer return a value (`void`):

| Method              |
|---------------------|
| `attach()`          |
| `detach()`          |

```php
$groups = ['...'];

$user = User::find('...');

// Returns "void"
$user->groups()->attach($groups);
```

### LdapRecord\Models\Relations\HasOne Method Return Type Changed

The `HasOne::attach()` method now no longer returns a value (`void`). I.e.:

```php
$manager = User::find('...');

$user = User::find('...');

// Returns "void"
$user->manager()->attach($manager);
```
