---
title: v2 Upgrade Guide
description: Upgrading from LdapRecord v1
---

# Upgrading to Version 2

### Models

#### Exception Handling

Models will now **always** throw the following exception after failure of creation/modification:

```php
LdapRecord\LdapRecordException
```

This is instead of potentially failing silently and returning `false`, as previously implemented.

Due to the above change, the following methods **will throw an exception** upon failure:

> **Important**: These methods now return no value (`void`).

- `$model->save()`
- `$model->update()`
- `$model->delete()`
- `$model->createAttribute()`
- `$model->updateAttribute()`
- `$model->deleteAttribute()`
- `$model->rename()`
- `$model->move()`

```php
// Before...
try {
  if ($model->save()) {
    //
  }
} catch (\LdapRecord\LdapRecordException $ex) {
  //
}

// After...
try {
  $model->save();
} catch (\LdapRecord\LdapRecordException $ex) {
  //
}
```

The follwowing static methods **will also throw an exception** upon failure:

> **Important**: These methods maintain their return value from v1.

- `Model::create()`
- `Model::destroy()`

#### Synchronize renamed to Refresh

The `$model->synchronize()` method has been renamed to follow Laravel's
Eloquent method name for the same purpose: `$model->refresh()`.

This will allow developers to utilize the same syntax across both Eloquent and LdapRecord:

```php
$user = User::find('cn=John Doe,dc=local,dc=com');

// Before...
$user->synchronize();

// After...
$user->refresh();
```

#### Automated Synchronization Removed

In v1 of LdapRecord, attributes on models would be automatically re-synchronized when any modification was performed on a model.

This ended up being very resource intensive when peforming bulk creations or updates on models.

If you require refreshing a models attribute's after creation or modification, use the `refresh()` method:

> **Important**: This means that Object GUID's are no longer available immediately after creation.

```php
use LdapRecord\Models\ActiveDirectory\User;

// Object GUID will not be available.
$user = User::create(['...']);

// Returns `null`:
$user->getConvertedGuid();

// Re-pulling attributes:
$user->refresh();

// Returns the newly assigned Object GUID:
$user->getConvertedGuid();
```

#### asDateTime Parameter Order

If you were casting an LDAP timestamp manually using the `$model->asDateTime()` method, the parameter order has been swapped:

```php
$type = 'windows-int';
$timestamp = '132460789290000000';

// Before...
$carbon = $model->asDateTime($type, $timestamp);

// After...
$carbon = $model->asDateTime($timestamp, $type);
```

### Query Builder

#### Not Found Exceptions

Previously in v1 of LdapRecord, when using `orFail()` methods directly on
raw `Connection` queries, a `ModelNotFoundException` would be thrown.

This didn't make much sense, since models are not returned from raw queries.

A new exception has been introduced to alleviate any confusion:

```
LdapRecord\Query\ObjectNotFoundException
```

> The `ModelNotFoundException` also now extends `ObjectNotFoundException`.

```php
$connection = new Connection(['...']);

// Before...
try {
    $connection->query()->firstOrFail();
} catch (\LdapRecord\Models\ModelNotFoundException $ex) {
    //
}

// After...
try {
    $connection->query()->firstOrFail();
} catch (\LdapRecord\Query\ObjectNotFoundException $ex) {
    //
}
```
