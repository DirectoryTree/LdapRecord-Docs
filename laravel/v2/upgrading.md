---
title: Upgrading from v1
description: LdapRecord-Laravel upgrading from v1 guide
---

# Upgrading to Version 2

## LdapRecord v2 Core

The core [LdapRecord](/docs/core/v2) repository has been updated to version 2.

Please visit the [upgrade guide](/docs/core/v2/upgrading/) to see any changes in the core that may have an impact on your application.

## Authentication Events

Authentication events have been moved into a new namespace with a new name, and modified properties.

This action was taken to unify the event structure accross all authentication events.

> **Important**: Each event class listed below has its parent namespace `LdapRecord\Laravel` omitted for brevity.

| v1                                 | v2                                      |
| ---------------------------------- | --------------------------------------- |
| `Events\Authenticating`            | `Events\Auth\Binding`                   |
| `Events\Authenticated`             | `Events\Auth\Bound`                     |
| `Events\AuthenticationFailed`      | `Events\Auth\BindFailed`                |
| `Events\AuthenticationRejected`    | `Events\Auth\Rejected`                  |
| `Events\AuthenticatedWithWindows`  | `Events\Auth\CompletedWithWindows`      |
| `Events\AuthenticatedModelTrashed` | `Events\Auth\EloquentUserTrashed`       |
| `Events\DiscoveredWithCredentials` | `Events\Auth\DiscoveredWithCredentials` |

## Import Events

Import events have been moved into a new namespace with a new name, and modified properties.

> **Important**: Each event class listed below has its parent namespace `LdapRecord\Laravel` omitted for brevity.

| v1                      | v2                             |
| ----------------------- | ------------------------------ |
| `Events\Imported`       | `Events\Import\Imported`       |
| `Events\Importing`      | `Events\Import\Importing`      |
| `Events\Synchronized`   | `Events\Import\Synchronized`   |
| `Events\Synchronizing`  | `Events\Import\Synchronizing`  |
| `Events\DeletedMissing` | `Events\Import\DeletedMissing` |

## Attribute Hydrator Changes

Attribute Hydrator classes have been moved into a new namespace.

They have maintained the same API as v1.

> **Important**: Each event class listed below has its parent namespace `LdapRecord\Laravel` omitted for brevity.

| v1                            | v2                                   |
| ----------------------------- | ------------------------------------ |
| `EloquentHydrator`            | `Import\EloquentHydrator`            |
| `EloquentUserHydrator`        | `Import\EloquentUserHydrator`        |
| `Hydrators\Hydrator`          | `Import\Hydrators\Hydrator`          |
| `Hydrators\GuidHydrator`      | `Import\Hydrators\GuidHydrator`      |
| `Hydrators\DomainHydrator`    | `Import\Hydrators\DomainHydrator`    |
| `Hydrators\PasswordHydrator`  | `Import\Hydrators\PasswordHydrator`  |
| `Hydrators\AttributeHydrator` | `Import\Hydrators\AttributeHydrator` |

## LdapImporter Changes

The `LdapRecord\Laravel\LdapImporter` has been moved and renamed to `LdapRecord\Laravel\Import\Synchronizer`.

It maintains the same API as v1, with some new public methods for convenience.

## LdapUserImporter Changes

The `LdapRecord\Laravel\LdapUserImporter` has been moved and renamed to `LdapRecord\Laravel\Import\UserSynchronizer`.

It maintains the same API as v1.

## Directory Emulator

When running application tests using the `DirectoryEmulator` your must now
call `DirectoryEmulator::tearDown()` in PHPUnit's `tearDown()` method:

```php
protected function tearDown(): void
{
    DirectoryEmulator::teardown();

    parent::tearDown();
}
```
