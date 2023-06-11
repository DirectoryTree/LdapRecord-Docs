---
title: Upgrade to LdapRecord-Laravel v3
description: Upgrading from LdapRecord-Laravel v2
---

# Upgrading to Version 3

We strive to record all potential breaking changes. However, as some of
these changes occur in lesser-known areas of the library, only a
fraction of them might have an impact on your application.

If you encounter any changes not documented here that have affected you,
please create a bug report on the [LdapRecord-Docs repository](https://github.com/DirectoryTree/LdapRecord-Docs/issues/new)
so that we can address the issue promptly.

## High Impact Changes

### Updating Dependencies

#### PHP >= 8.1.0 Required

LdapRecord-Laravel v3 now requires PHP 8.1.0 or greater.

#### Laravel >= 8.0 Required

LdapRecord-Laravel now requires Laravel version 8.0 or greater.

#### LdapRecord v3

The core [LdapRecord](/docs/core/v3) repository has been updated to version 3.

Please visit the [upgrade guide](/docs/core/v3/upgrading) to see any
changes in the core that may have an impact on your application.

#### Composer Dependencies

You should update the following dependency in your application's `composer.json` file:

```json
"directorytree/ldaprecord-laravel": "^3.0"
```

### Strict PHP Types Implemented

LdapRecord-Laravel (and the core LdapRecord repository) now has strict 
types implemented in all classes for all methods and properties.

If you've created your own [models](/docs/core/v3/models), [scopes](/docs/core/v3/model-scopes),
or have extended any other class from either repository, you will need to adjust any 
overridden properties or methods with their respective types.

### Configuration Updates

The `logging` configuration options have been moved to an array. You 
may either republish your configuration file by deleting the
existing one (`config/ldap.php`) and running:

```bash
php artisan vendor:publish --provider="LdapRecord\Laravel\LdapServiceProvider"
```

Or, by updating the option manually:

```diff
// config/ldap.php

return [
    // ...

-   'logging' => true,
-   'logging_channel' => 'stack',

+   'logging' => [
+       'enabled' => true,
+       'channel' => 'stack',
+   ],
];
```

## Medium Impact Changes

### LdapRecord\Laravel\Auth\Rule changes

The `LdapRecord\Laravel\Auth\Rule` abstract class has been moved to an interface and the
`isValid` method has also been renamed to `passes`, which now accepts an LdapRecord
`Model` as the first parameter and an Eloquent `Model` as the second:

```diff
namespace App\Ldap;

use LdapRecord\Models\Model as LdapRecord;
use Illuminate\Database\Eloquent\Model as Eloquent;

use LdapRecord\Laravel\Auth\Rule;

- class MyRule extends Rule
+ class MyRule implements Rule
{
    /**
     * Determine if the rule passes validation.
     */
-    public function isValid();
+    public function passes(LdapRecord $user, Eloquent $model = null): bool
    {
        // ...
    }
}
```

## Low Impact Changes

### "Make" Command Changes

The `make:ldap-*` methods have been renamed:

```diff
- php artisan make:ldap-rule
+ php artisan ldap:make:rule
```

```diff
- php artisan make:ldap-model
+ php artisan ldap:make:model
```

```diff
- php artisan make:ldap-scope
+ php artisan ldap:make:scope
```

### LdapRecord\Laravel\Auth\Validator

The `Validator` class has had the `fails` method removed, and the `passes()` method now accepts an LdapRecord
`Model` as the first parameter, and an Eloquent `Model` as the second:

```diff
+ use LdapRecord\Models\Model as LdapRecord;
+ use Illuminate\Database\Eloquent\Model as Eloquent;

class Validator
{
    // ...
-    public function passes();
+    public function passes(LdapRecord $user, Eloquent $model = null): bool;
    
-    public function fails();
}
```
