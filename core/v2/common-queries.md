---
title: Common Queries
description: Common LDAP queries using LdapRecord
---

# Common Queries

Most applications will require retrieving certain set / type of objects from a directory.

## Using Models

Utilizing LdapRecord's built in models allow you to easily query specific types of objects in your directory.

> The examples below assume you have created a `Connection` and have added them into the [Container](/docs/core/v2/connections#container).

### Active Directory

#### Generic Queries

```php
// All Active Directory objects:
// Note: We use 'paginate' here so over 1000 results can be returned.
$objects = \LdapRecord\Models\ActiveDirectory\Entry::paginate();

// All Active Directory users:
$users = \LdapRecord\Models\ActiveDirectory\User::get();

// All Active Directory contacts:
$contacts = \LdapRecord\Models\ActiveDirectory\Contact::get();

// All Active Directory groups:
$groups = \LdapRecord\Models\ActiveDirectory\Group::get();

// All Active Directory organizational units:
$ous = \LdapRecord\Models\ActiveDirectory\OrganizationalUnit::get();

// All Active Directory printers:
$printers = \LdapRecord\Models\ActiveDirectory\Printer::get();

// All Active Directory computers:
$computers = \LdapRecord\Models\ActiveDirectory\Computer::get();

// All foreign security principals:
$foreignPrincipals = \LdapRecord\Models\ActiveDirectory\ForeignSecurityPrincipal::get();
```

#### Users Created After a Date

```php
$date = (new DateTime('October 1st 2016'))->format('YmdHis.0\Z');

$users = User::where('whencreated', '>=', $date)->get();
```

### OpenLDAP

#### Generic Queries

```php
// All OpenLDAP objects:
// Note: We use 'paginate' here so over 1000 results can be returned.
$objects = \LdapRecord\Models\OpenLDAP\Entry::paginate();

// All OpenLDAP users:
$users = \LdapRecord\Models\OpenLDAP\User::get();

// All OpenLDAP groups:
$groups = \LdapRecord\Models\OpenLDAP\Group::get();

// All OpenLDAP organizational units:
$ous = \LdapRecord\Models\OpenLDAP\OrganizationalUnit::get();
```

## Without Models

If you do not want to use LdapRecord models, you can still use the query builder and retrieve raw LDAP results.

```php
use LdapRecord\Connection;

$connection = new Connection(['...']);

// All LDAP objects:
// Note: We use 'paginate' here so over 1000 results can be returned.
$objects = $connection->query()->paginate();
```

### Active Directory

```php
use LdapRecord\Connection;

$connection = new Connection(['...']);

// All Active Directory Users:
$users = $connection->query()->where([
    ['objectclass', '=', 'top'],
    ['objectclass', '=', 'person'],
    ['objectclass', '=', 'organizationalperson'],
    ['objectclass', '=', 'user'],
])->get();

// All Active Directory contacts:
$contacts = $connection->query()->where([
    ['objectclass', '=', 'top'],
    ['objectclass', '=', 'person'],
    ['objectclass', '=', 'organizationalperson'],
    ['objectclass', '=', 'contact'],
])->get();

// All Active Directory groups:
$groups = $connection->query()->where([
    ['objectclass', '=', 'top'],
    ['objectclass', '=', 'group'],
])->get();

// All Active Directory organizational units:
$ous = $connection->query()->where([
    ['objectclass', '=', 'top'],
    ['objectclass', '=', 'organizationalunit'],
])->get();

// All Active Directory printers:
$printers = $connection->query()
    ->where('objectclass', '=', 'printqueue')
    ->get();

// All Active Directory computers:
$computers = $connection->query()->where([
    ['objectclass', '=', 'top'],
    ['objectclass', '=', 'person'],
    ['objectclass', '=', 'organizationalperson'],
    ['objectclass', '=', 'user'],
    ['objectclass', '=', 'computer'],
])->get();

// All foreign security principals:
$foreignPrincipals = $connection->query()
    ->where('objectclass', '=', 'foreignsecurityprincipal')
    ->get();
```

### OpenLDAP

```php
// All OpenLDAP users:
$users = $connection->query()->where([
    ['objectclass', '=', 'top'],
    ['objectclass', '=', 'person'],
    ['objectclass', '=', 'organizationalperson'],
    ['objectclass', '=', 'inetorgperson'],
])->get();

// All OpenLDAP groups:
$groups = $connection->query()->where([
    ['objectclass', '=', 'top'],
    ['objectclass', '=', 'groupofuniquenames'],
])->get();

// All OpenLDAP organizational units:
$ous = $connection->query()->where([
    ['objectclass', '=', 'top'],
    ['objectclass', '=', 'organizationalunit'],
])->get();
```
