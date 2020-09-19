---
title: Using LdapRecord-Laravel with Laravel Jetstream
description: Setting up LDAP authentication with Laravel JetStream
extends: _layouts.laravel-documentation
section: content
---

# Laravel Jetstream

- [Introduction](#introduction)
- [Fortify Setup](#fortify-setup)
- [Using Usernames](#using-usernames)
 - [Fortify Setup](#fortify-setup-usernames)
 - [Sync Attributes](#sync-attributes-usernames)
- [Fallback Authentication](#fallback-auth)
- [Eloquent Model Binding](#model-binding)
- [Displaying LDAP Error Messages (password expiry, account lockouts)](#displaying-ldap-error-messages)

## Introduction {#introduction}

Laravel Jetstream utilizes [Laravel Fortify](https://github.com/laravel/fortify) for authentication under the hood.
We will customize various aspects of it to allow our LDAP users to sign in successfully.

## Fortify Setup {#fortify-setup}

### Authentication Callback

For this example application, we will authenticate our LDAP users with their email address using the LDAP attribute `mail`.

For LdapRecord to properly locate the user in your directory during sign in, we will
override Fortify's authentication callback using the `Fortify::authenticateUsing`
method in our `AuthServiceProvider.php` file:

```php
// app/Providers/AuthServiceProvider.php

public function boot()
{
    $this->registerPolicies();

    Fortify::authenticateUsing(function ($request) {
        $validated = Auth::validate($credentials = [
            'mail' => $request->email,
            'password' => $request->password
        ]);

        return $validated ? Auth::getLastAttempted() : null;
    });
}
```

As you can see above, we set the `mail` key which is passed to the LdapRecord authentication provider.

A search query will be executed on your directory for a user that contains the `mail` attribute equal
to the entered `email` that the user has submitted on your login form. The `password`
key will not be used in the search.

If a user cannot be located in your directory, or they fail authentication, they will be redirected to the
login page normally with the "*Invalid credentials*" error message.

> You may also add extra key => value pairs in the `credentials` array to further scope the
> LDAP query. The `password` key is automatically ignored by LdapRecord.

### Features

Since we are synchronizing data from our LDAP server, we must disable the following
features by commenting them out inside of the `config/fortify.php`:

- `Features::updateProfileInformation()`
- `Features::updatePasswords()`

```php
// config/fortify.php

// Before:
'features' => [
    Features::registration(),
    Features::resetPasswords(),
    // Features::emailVerification(),
    Features::updateProfileInformation(),
    Features::updatePasswords(),
    // Features::twoFactorAuthentication(),
],

// After:
'features' => [
    // Features::registration(),
    // Features::resetPasswords(),
    // Features::emailVerification(),
    // Features::updateProfileInformation(),
    // Features::updatePasswords(),
    // Features::twoFactorAuthentication(),
],
```

> **Important**: You may keep `Features::registration()` enabled if you would like
> to continue accepting local application user registration and authentication.

## Using Usernames {#using-usernames}

To authenticate your users by their username (`sAMAccountName` for example), we must adjust some scaffolded Jetstream code.

### Fortify Setup {#fortify-setup-username}

#### Configuration {#fortify-setup-username-config}

Inside of our `config/fortify.php` file, we must change the `username` option to `username` from `email`:

```php
// config/fortify.php

// Before:
'username' => 'email',

// After:
'username' => 'username',
```

#### Authentication Callback {#fortify-setup-username-callback}

With our Fortiy configuration updated, we will jump into our `AuthServiceProvider.php` file
and setup our authentication callback using the `Fortify::authenticateUsing()` method:

```php
// app/Providers/AuthServiceProvider.php

public function boot()
{
    $this->registerPolicies();

    Fortify::authenticateUsing(function ($request) {
        $validated = Auth::validate($credentials = [
            'samaccountname' => $request->username,
            'password' => $request->password
        ]);

        return $validated ? Auth::getLastAttempted() : null;
    });
}
```

### Sync Attributes {#sync-attributes-usernames}

When using usernames, we must also adjust the `sync_attributes` option inside
of our `config/auth.php` file. We will adjust it to reflect our `username`
database column to be synchronized with the `samaccountname` attribute:

```php
// config/auth.php

'providers' => [
    // ...

    'ldap' => [
        // ...
        'database' => [
            // ...
            'sync_attributes' => [
                'name' => 'cn',
                'username' => 'samaccountname',
            ],
        ],
    ],
],
```

### Migration {#usernames-migration}

The built in `users` database table migration must also be modified to use a `username` column instead of `email`:

```php
// database/migrations/2014_10_12_000000_create_users_table.php

// Before:
$table->string('email')->unique();

// After:
$table->string('username')->unique();
```

### Login View

Now lets jump into our `login.blade.php` view to modify the HTML input field
to allow the use of a username instead of email address:

```html
<!-- resources/auth/login.blade.php -->

<!-- Before: -->
<div>
    <x-jet-label value="Email" />
    <x-jet-input class="block mt-1 w-full" type="email" name="email" :value="old('email')" required autofocus />
</div>

<!-- After: -->
<div>
    <x-jet-label value="Username" />
    <x-jet-input class="block mt-1 w-full" type="text" name="username" :value="old('username')" required autofocus />
</div>
```

### User Model

If you plan on allowing non-LDAP users to register and login to your application,
you must adjust the `$fillable` attributes property on your `app/Models/User.php`:

```php
// app/Models/User.php

class User extends Authenticatable implements LdapAuthenticatable
{
    // ...

    // Before:
    protected $fillable = [
        'name',
        'email',
        'password',
    ];

    // After:
    protected $fillable = [
        'name',
        'username',
        'password',
    ];
}
```

## Fallback Authentication {#fallback-auth}

Database fallback allows the authentication of local database users if **LDAP
connectivity is not present**, or **an LDAP user cannot be found**.

To enable this feature, you must define a `fallback` array inside of the credentials
you insert into the `Auth::validate` method in your `Fortify::authenticateUsing` callback:

```php
// app/Providers/AuthServiceProvider.php

public function boot()
{
    $this->registerPolicies();

    Fortify::authenticateUsing(function ($request) {
        $validated = Auth::validate($credentials = [
            'mail' => $request->email,
            'password' => $request->password,
            'fallback' => [
                'email' => $request->email,
                'password' => $request->password,
            ],
        ]);

        return $validated ? Auth::getLastAttempted() : null;
    });
}
```

For example, given the following `users` database table:

id | name | email | password | guid | domain |
--- | --- | --- | --- |
1 | Steve Bauman | sbauman@outlook.com | ... | `null` | `null` |

If a user attempts to login with the above email address and this user does
not exist inside of your LDAP directory, then standard Eloquent authentication
will be performed instead.

This feature is ideal for environments where:

- LDAP server connectivity may be intermittent
- Or; You have regular users registering normally in your application

> If you would like your LDAP users to be able to sign in to your application
> when LDAP connectivity fails or is not present, you must enable the
> [sync passwords](#database-password-sync) option, so your LDAP
> users can sign in using their last used password. 
> <br/><br/>
> If an LDAP users password has not been synchronized, they will not be able to sign in.

## Eloquent Model Binding {#model-binding}

If you are using [database synchronization](/docs/laravel/auth#database), model binding allows
you to access the **currently authenticated user's** LdapRecord model from their Eloquent
model. This grants you access to their LDAP data whenever you need it.

To begin, insert the `LdapRecord\Laravel\Auth\HasLdapUser` trait onto your User model:

```php
// app/Models/User.php

// ...
use LdapRecord\Laravel\Auth\HasLdapUser;
use LdapRecord\Laravel\Auth\LdapAuthenticatable;
use LdapRecord\Laravel\Auth\AuthenticatesWithLdap;

class User extends Authenticatable implements LdapAuthenticatable
{
    //...

    use HasLdapUser;

    // ...
}
```

Now, after an LDAP user logs into your application, their LdapRecord model will be
available on their Eloquent model via the `ldap` property:

> If their LDAP model cannot be located, the returned value will be `null`.

```php
// Instance of App\User
$user = Auth::user();

// Instance of App\Ldap\User
$user->ldap;

// Get LDAP user attributes
echo $user->ldap->getFirstAttribute('cn');

// Get LDAP user relationships:
$groups = $user->ldap->groups()->get();
```

> This property uses deferred loading -- which means that the users LDAP model only
> gets requested from your server when you attempt to access it. This prevents
> loading the model unnecessarily when it is not needed in your application.