---
title: Database Authentication Installation
description: LdapRecord-Laravel authentication install guide
---

# Database Authentication Installation

## Introduction

Database authentication requires the addition of two (2) database columns inside of your `users` database table:

- **`guid`** <br/> This is for storing your LDAP users `objectguid`. It is used for locating and synchronizing your LDAP user.<br/><br/>
- **`domain`** <br/> This is for storing your LDAP users connection name. It is used to identify users from different domains.

## Publishing The Required Migration

Publish the migration using the below command:

```bash
php artisan vendor:publish --provider="LdapRecord\Laravel\LdapAuthServiceProvider"
```

Then run the migration using the below command:

```bash
php artisan migrate
```

## Add The Required Trait and Interface

Add the following interface and trait to your `User` Eloquent model:

- **Trait**: `LdapRecord\Laravel\Auth\AuthenticatesWithLdap`
- **Interface**: `LdapRecord\Laravel\Auth\LdapAuthenticatable`

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

You may change the database column names in the published migration to anything you would like.
However, once you have done so, you must override the following methods in your Eloquent
`User` model that are provided by the above mentioned LdapRecord trait and interface:

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
        return 'my_domain_column';
    }

    public function getLdapGuidColumn()
    {
        return 'my_guid_column';
    }
}
```
