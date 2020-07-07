---
title: Migrating to LdapRecord | Adldap2 Migration
description: Migrating from Adldap2 to LdapRecord
extends: _layouts.documentation
section: content
---

# Migrating from Adldap2

- [Introduction](#introduction)
- [High Impact Changes](#high-impact-changes)
  - [Adldap to Container](#adldap-to-container)
  - [Providers to Connections](#providers-to-connections)
  - [Adldap Interface Removal](#adldap-interface-removal)
  - [Schema Interface Removal](#schema-interface-removal)
  - [Query Builder](#query-builder)
  - [Exceptions](#exceptions)
  - [Models](#)
- [Medium Impact Changes](#medium-impact-changes)
  - [Configuration](#configuration)
- [Low Impact Changes](#)
  - [Authentication](#)
  - [Query Builder](#)

## Introduction

This guide will cover the largest changes that have been made during the development of LdapRecord.

> Please be aware, this guide cannot cover all use-cases. You will have to adjust and change things.
> LdapRecord was not meant to be a drop-in replacement of Adldap2. It is a brand new project
> -- but keeps some similarities with the Adldap2 library.

## High Impact Changes {#high-impact-changes}

### Adldap to Container {#adldap-to-container}

The `Adldap\Adldap` class is responsible for storing connections.

This has been replaced with the `LdapRecord\Container` class:

**Adldap2**:
```php
use Adldap\Adldap;

$ad = new Adldap();

$config = ['...'];

$ad->addProvider($config, $name = 'default');
```

**LdapRecord**:
```php
use LdapRecord\Container;

$config = ['...'];

Container::addConnection($config, $name = 'default');
```

### Providers to Connections {#providers-to-connections}

The `Adldap\Connections\Provider` has been replaced with `LdapRecord\Connection`.

Here is an example:

> **Important:** Provider configuration has [undergone minimal changes](#configuration).
> Most options have been carried over.

**Adldap2**:
```php
use Adldap\Connections\Provider;

$provider = new Provider([
    'hosts' => ['192.168.0.1'],
    // ...
]);
```

**LdapRecord**:
```php
use LdapRecord\Connection;

$connection = new Connection([
    'hosts' => ['192.168.0.1'],
    // ...
]);
```

#### Connecting

**Adldap2**:
```php
// With configured username / password:
$provider->connect();

// With dynamic username / password:
$provider->connect('cn=user,dc=local,dc=com', 'secret');
```

**LdapRecord**:
```php
// With configured username / password:
$connection->connect();

// With dynamic username / password:
$connection->connect('cn=user,dc=local,dc=com', 'secret');
```

#### Methods

### Adldap Interface Removal {#adldap-interface-removal}

The `Adldap\AdldapInterface` has been removed. There is no equivalent in LdapRecord.

### Schema Interface Removal {#schema-interface-removal}

The Adldap2 schema interface and classes have been completely removed. There is no equivalent in LdapRecord.

All attributes you were using from the schema must accessed directly on models.

**Removed:**

- `Adldap\Schemas\Schema`
- `Adldap\Schemas\FreeIPA`
- `Adldap\Schemas\OpenLDAP`
- `Adldap\Schemas\ActiveDirectory`
- `Adldap\Schemas\SchemaInterface`

See why this change has occurred [here](https://stevebauman.ca/posts/why-ldap-record/).

### Query Builder {#query-builder}

The following methods have been removed:

- `$query->findBaseDn()`
- `$query->findBySid()`
- `$query->findBySidOrFail()`
- `$query`
- `$query->sortBy()`
- `$query->isSorted()`
- `$query->getSortByFlags()`
- `$query->getSortByField()`
- `$query->getSortByDirection()`
- `$query->whereEnabled()`
- `$query->whereDisabled()`
- `$query->whereMemberOf()`

### Exceptions

The following exceptions have been renamed:

From | To |
--- | --- |
`Adldap\AdldapException` | `LdapRecord\LdapRecordException` |
`Adldap\Auth\BindException` | `LdapRecord\Auth\BindException` |
`Adldap\Connections\ConnectionException` | `LdapRecord\ConnectionException` |
`Adldap\Auth\UsernameRequiredException` | `LdapRecord\Auth\UsernameRequiredException` |
`Adldap\Auth\PasswordRequiredException` | `LdapRecord\Auth\PasswordRequiredException` |
`Adldap\Models\ModelNotFoundException` | `LdapRecord\Models\ModelNotFoundException` |
`Adldap\Models\ModelDoesNotExistException` | `LdapRecord\Models\ModelDoesNotExistException` |

### Models

## Medium Impact Changes {#medium-impact-changes}

### Configuration {#configuration}

The following configuration options have been removed:

- `schema`
- `account_prefix`
- `account_suffix`

> All other configuration options have been kept identical and operate the same.

If you require the same functionality, simply append / prepend a string to the users username:

**Adldap2**:
```php
$config = [
    'account_prefix' => 'cn=',
    'account_suffix' => ',ou=users,dc=local,dc=com',
];

$username = 'jdoe';
$password = 'secret';

if ($provider->auth()->attempt($username, $password)) {
    //
}
```

**LdapRecord**:
```php
$username = 'jdoe';
$password = 'secret';

$prefix = 'cn=';
$suffix = ',ou=users,dc=local,dc=com';

if ($connection->auth()->attempt($prefix.$username.$suffix, $password)) {
    //
}
```

## Low Impact Changes {#low-impact-changes}


### Authentication

