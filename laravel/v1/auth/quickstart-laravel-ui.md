---
title: Authentication Quickstart Using Laravel UI
description: LdapRecord-Laravel Auth Quickstart Guide
---

# Authentication Quickstart

## Introduction

> Please complete the [LdapRecord-Laravel quickstart guide](/docs/laravel/v1/quickstart)
> to install LdapRecord and configure your LDAP connection prior to setting up
> authentication.

Before you begin, this guide assumes you have published Laravel's default authentication scaffolding using the `laravel/ui` package.

If you haven't done this yet, please follow Laravel's [auth scaffolding guide](https://laravel.com/docs/9.x/authentication)
to get started, then head back here once done.

## Debugging

Inside of your `config/ldap.php` file, ensure you have `logging` enabled during the setup of authentication.
Doing this will help you immensely in debugging connectivity and authentication issues.

If you encounter issues along the way, be sure to open your `storage/logs` directory after you
attempt signing in to your application and see what issues may be occurring.

In addition, you may also run the below artisan command to test connectivity to your LDAP server:

```bash
php artisan ldap:test
```

## Plain LDAP Authentication

### Step 1 - Configure the Authentication Driver

Inside of your `config/auth.php` file, we must add a new provider in the `providers` array.

In this example, we will create a provider named `ldap`:

```php
// config/auth.php

'providers' => [
    // ...

    'ldap' => [
        'driver' => 'ldap',
        'model' => LdapRecord\Models\ActiveDirectory\User::class,
    ],

    // ...
],
```

If you are using OpenLDAP, you must switch the providers `model` option to:

```php
LdapRecord\Models\OpenLDAP\User::class
```

Once you have setup your `ldap` provider, you must update the `provider` value in the `web` guard:

```php
// config/auth.php

'guards' => [
    'web' => [
        'driver' => 'session',
        'provider' => 'ldap', // Changed to 'ldap'
    ],

    // ...
],
```

### Step 2 - Setting up your LoginController

Now we must change our `LoginController` to allow LdapRecord to properly
locate users who are attempting to sign into our application. We do
this by changing the `credentials` method. In this method we will
return an array that contains the users username and password.

**The array keys you set here are important.** The `password` key must be present,
as this is sent directly to your LDAP server for verification.

The other key must be the **name of the LDAP attribute** you want LdapRecord to
locate the authenticating user with. It **must** be an attribute that has
a unique value per user in your directory. `uid`, `sAMAccountName`,
`mail`, or `userPrincipalName` are good examples of attributes
that have a unique value per user.

In this example we want to use the users `mail` LDAP attribute to sign them into our application.

```php
use Illuminate\Http\Request;

class LoginController extends Controller
{
    // ...

    protected function credentials(Request $request)
    {
        return [
            'mail' => $request->get('email'),
            'password' => $request->get('password'),
        ];
    }
}
```

### Step 3 - Modifying The Layout Blade View

When we use plain LDAP authentication, an instance of the LdapRecord `model` you have
configured for authentication will be returned when calling the `Auth::user()`
method. This means that our currently published blade views will throw an
exception due to using `Auth::user()->name` call inside of the view file
`views/layouts/app.blade.php`.

You must change the syntax to the following:

```html
<!-- resources/views/layouts/app.blade.php -->

<!-- From... -->
{{ Auth::user()->name }}

<!-- To... -->
{{ Auth::user()->getFirstAttribute('cn') }}
```

Once you've updated the syntax, your application is now ready to authenticate LDAP users.

## Synchronized Database Authentication

### Step 1 - Publish the Migration

LdapRecord requires you to have two additional user database columns.

| Column   | Reason                                                                                                                          |
| -------- | ------------------------------------------------------------------------------------------------------------------------------- |
| `guid`   | This is for storing your LDAP users `objectguid`. It is needed for locating and synchronizing your LDAP user to the database.   |
| `domain` | This is for storing your LDAP users connection name. It is needed for storing your configured LDAP connection name of the user. |

Go ahead and publish the migration using the below command:

```bash
php artisan vendor:publish --provider="LdapRecord\Laravel\LdapAuthServiceProvider"
```

Then, run the migrations with the `artisan migrate` command:

```bash
php artisan migrate
```

### Step 2 - Configure the Authentication Driver

Inside of your `config/auth.php` file, we must add a new provider in the `providers` array.

In this example, we will create a provider named `ldap`:

```php
// config/auth.php

'providers' => [
    // ...

    'ldap' => [
        'driver' => 'ldap',
        'model' => LdapRecord\Models\ActiveDirectory\User::class,
        'database' => [
            'model' => App\User::class,
            'sync_passwords' => false,
            'sync_attributes' => [
                'name' => 'cn',
                'email' => 'mail',
            ],
        ],
    ],
],
```

If you are using OpenLDAP, you must switch the providers `model` option to:

```php
LdapRecord\Models\OpenLDAP\User::class
```

If you are using a different LDAP type, you will need to [define your own LDAP model](/docs/core/v1/models/#defining-models)
and insert it there. This model is used for locating the authenticating user in your LDAP directory.

Once you have setup your `ldap` provider, you must update the `provider` value in the `web` guard:

```php
// config/auth.php

'guards' => [
    'web' => [
        'driver' => 'session',
        'provider' => 'ldap', // Changed to 'ldap'
    ],

    // ...
],
```

### Step 3 - Setting up your database user model

Now, we must add the following trait and interface to our `User` Eloquent model:

| Type      | Name                                            |
| --------- | ----------------------------------------------- |
| Interface | `LdapRecord\Laravel\Auth\LdapAuthenticatable`   |
| Trait     | `LdapRecord\Laravel\Auth\AuthenticatesWithLdap` |

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

These are required so LdapRecord can set and retrieve your users `domain` and `guid` database columns.

If you would like to override the database column names that are used, you can override the following methods:

| Methods                       |
| ----------------------------- |
| `User::getLdapDomainColumn()` |
| `User::getLdapGuidColumn()`   |

### Step 4 - Setting up your LoginController:

Now we must change our `LoginController` to allow LdapRecord to properly
locate users who are attempting to sign into our application. We do
this by changing the `credentials` method. In this method we will
return an array that contains the users username and password.

**The array keys you set here are important.** The `password` key must be present,
as this is sent directly to your LDAP server for verification.

The other key must be the **name of the LDAP attribute** you want LdapRecord to
locate the authenticating user with. It **must** be an attribute that has
a unique value per user in your directory. `uid`, `sAMAccountName`,
`mail`, or `userPrincipalName` are good examples of attributes
that have a unique value per user.

In this example we want to use the users `mail` LDAP attribute to sign them into our application.

```php
use Illuminate\Http\Request;

class LoginController extends Controller
{
    // ...

    protected function credentials(Request $request)
    {
        return [
            'mail' => $request->get('email'),
            'password' => $request->get('password'),
        ];
    }
}
```

Your application is now ready to authenticate LDAP users.
