---
title: Authentication Installation
description: LdapRecord-Laravel authentication install guide
---

# Authentication Installation

If you are using [database synchronization](/docs/laravel/v1/auth#database), you must publish the
included migrations to add the following database columns to your `users` table:

|  Column  | Reason                                                                                                                          |
| :------: | ------------------------------------------------------------------------------------------------------------------------------- |
|  `guid`  | This is for storing your LDAP users `objectguid`. It is needed for locating and synchronizing your LDAP user to the database.   |
| `domain` | This is for storing your LDAP users connection name. It is needed for storing your configured LDAP connection name of the user. |

Publish the migration using the below command:

```bash
php artisan vendor:publish --provider="LdapRecord\Laravel\LdapAuthServiceProvider"
```

Then, add the following interface and trait to your `app/User.php` model:

|   Type    |                                                 |
| :-------: | ----------------------------------------------- |
| Interface | `LdapRecord\Laravel\Auth\LdapAuthenticatable`   |
|   Trait   | `LdapRecord\Laravel\Auth\AuthenticatesWithLdap` |

```php
// app/User.php

// ...

use LdapRecord\Laravel\Auth\LdapAuthenticatable;
use LdapRecord\Laravel\Auth\AuthenticatesWithLdap;

class User extends Authenticatable implements LdapAuthenticatable
{
    use Notifiable, AuthenticatesWithLdap;

    // ...
}
```

This trait and interface provide LdapRecord the ability of setting and getting your users
`domain` and `guid` database columns upon authentication.

## Migration Customization

You may change the database column names on the published migration to anything you'd like.
However, once you have done so, you must override the following methods in your Eloquent
`User` model that are provided by the LdapRecord trait and interface:

```php
// app/User.php

// ...

use LdapRecord\Laravel\Auth\LdapAuthenticatable;
use LdapRecord\Laravel\Auth\AuthenticatesWithLdap;

class User extends Authenticatable implements LdapAuthenticatable
{
    // ...

    public function getLdapDomainColumn()
    {
        return 'domain_column';
    }

    public function getLdapGuidColumn()
    {
        return 'guid_column';
    }
}
```
