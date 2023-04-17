---
title: Quickstart
description: Get up and running fast with LdapRecord.
---

# Quick Start

Install LdapRecord using [composer](https://getcomposer.org/):

```bash
composer require directorytree/ldaprecord
```

Use LdapRecord:

```php
use LdapRecord\Container;
use LdapRecord\Connection;
use LdapRecord\Models\Entry;

// Create a new connection:
$connection = new Connection([
    'hosts' => ['192.168.1.1'],
    'port' => 389,
    'base_dn' => 'dc=local,dc=com',
    'username' => 'cn=user,dc=local,dc=com',
    'password' => 'secret',
]);

// Add the connection into the container:
Container::addConnection($connection);

// Get all objects:
$objects = Entry::get();

// Get a single object:
$object = Entry::find('cn=John Doe,dc=local,dc=com');

// Getting attributes:
foreach ($object->memberof as $group) {
    echo $group;
}

// Modifying attributes:
$object->company = 'My Company';

// Saving changes:
$object->save();
```
