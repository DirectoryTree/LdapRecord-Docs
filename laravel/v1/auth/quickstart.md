---
title: Authentication Quickstart
description: LdapRecord-Laravel Auth Quickstart Guide
---

# Authentication Quickstart

## Introduction

> Please complete the [LdapRecord-Laravel quickstart guide](/docs/laravel/v1/quickstart)
> to install LdapRecord and configure your LDAP connection prior to setting up
> authentication.

Before you begin, this guide assumes you have published Laravel's authentication scaffolding using the `laravel/jetstream` package.

If you haven't done this yet, please follow Laravel Jetstream's [scaffolding guide](https://jetstream.laravel.com/1.x/installation)
to get started, then head back here once done.

## Debugging

Inside of your `config/ldap.php` file, ensure you have `logging` enabled during the setup of authentication.
Doing this will help you immensely in debugging connectivity and authentication issues.

If you encounter issues along the way, be sure to open your `storage/logs` directory after you
attempt signing in to your application and see what issues may be occurring.

In addition, you may also run the below artisan command to test
connectivity to each of your configured LDAP servers:

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

### Step 2 - Setting up Laravel Fortify

#### Authentication Callback

Laravel Jetstream uses [Laravel Fortify](https://github.com/laravel/fortify) for authentication.
We will configure its various features to support signing in with LdapRecord.

To support LDAP authentication, we must call the `Fortify::authenticateUsing()`
and supply our own callback, overriding Laravel Fortify's default:

We will call the above in our `AuthServiceProvider.php` file, inside the `boot()` method:

```php
// app/Providers/AuthServiceProvider.php

// ...
use Laravel\Fortify\Fortify;
use Illuminate\Support\Facades\Auth;

class AuthServiceProvider extends ServiceProvider
{
    // ...

    public function boot()
    {
        $this->registerPolicies();

        Fortify::authenticateUsing(function ($request) {
            $validated = Auth::validate([
                'mail' => $request->email,
                'password' => $request->password
            ]);

            return $validated ? Auth::getLastAttempted() : null;
        });
    }
}
```

As you can see above in the `Fortify::authenticateUsing()` callback, we are passing
an array of the users credentials to the `Auth::validate()` method. Most notibly,
we set the `mail` key in this credentials array which is passed to the
LdapRecord authentication provider.

Upon a user attempting to sign in, a search query will be executed on your directory for a user
that contains the `mail` attribute equal to the entered `email` that the user has submitted
on your login form. The `password` key will not be used in the search.

If a user cannot be located in your directory, or they fail authentication, they will be
redirected to the login page normally with the "_Invalid credentials_" error message.

> You may also add extra key => value pairs in the `credentials` array to further scope
> the LDAP query. The `password` key is automatically ignored by LdapRecord.

#### Feature Configuration

When using plain LDAP authentication, we must disable various Jetstream and Fortify features, such as
teams, two-factor authentication, profile photos, API, registration, updating profile information,
and resetting / updating passwords. We will make these changes in the `config/jetstream.php` and
`config/fority.php` configuration files respectively:

```php
// config/fortify.php

// Before:
'features' => [
    Features::registration(),
    Features::resetPasswords(),
    // Features::emailVerification(),
    Features::updateProfileInformation(),
    Features::updatePasswords(),
    Features::twoFactorAuthentication([
        'confirmPassword' => true,
    ]),
],

// After:
'features' => [
    // Features::registration(),
    // Features::resetPasswords(),
    // Features::emailVerification(),
    // Features::updateProfileInformation(),
    // Features::updatePasswords(),
    // Features::twoFactorAuthentication([
    //     'confirmPassword' => true,
    // ]),
],
```

```php
// config/jetstream.php

// Before:
'features' => [
    Features::profilePhotos(),
    Features::api(),
    Features::teams(),
],

// After:
'features' => [
    // Features::profilePhotos(),
    // Features::api(),
    // Features::teams(),
],
```

These features must be disabled since we cannot persist profile data,
two-factor authentication codes, and more, into the users LDAP object.

### Step 3 - Modifying Blade Views

When we use plain LDAP authentication, an instance of the LdapRecord `model` you have configured
for authentication will be returned when calling the `Auth::user()` method. This means that our
currently published blade views will immediately throw an exception due to calls such as:
`Auth::user()->name`. Most notably, the `views/navigation-dropdown.php` file, if you
are using the Livewire stack.

You must change the syntax to the following wherever it is found:

```html
<!-- From... -->
{{ Auth::user()->name }}

<!-- To... -->
{{ Auth::user()->getFirstAttribute('cn') }}
```

You will have to remove other calls completely, such as:

```html
{{ Auth::user()->profile_photo_url }}
```

These calls directly rely on Laravel's scaffolded database columns.

Once you've made the necessary modifications shown above, your
application is now ready to authenticate LDAP users.

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
            'model' => App\Models\User::class,
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
// app/Models/User.php

// ...
use LdapRecord\Laravel\Auth\LdapAuthenticatable;
use LdapRecord\Laravel\Auth\AuthenticatesWithLdap;

class User extends Authenticatable implements LdapAuthenticatable
{
    use AuthenticatesWithLdap;

    // ...
}
```

These are required so LdapRecord can set and retrieve your users `domain` and `guid` database columns.

If you would like to override the database column names that are used, you can override the following methods:

| Methods                       |
| ----------------------------- |
| `User::getLdapDomainColumn()` |
| `User::getLdapGuidColumn()`   |

### Step 4 - Setting up Laravel Fortify:

#### Authentication Callback & Password Confirmation

Laravel Jetstream uses [Laravel Fortify](https://github.com/laravel/fortify) for authentication.
We will configure its various features to support signing in with LdapRecord.

To support LDAP authentication, we must call the following two methods
and supply our own callbacks, overriding Laravel Fortify's default:

- `Fortify::authenticateUsing()`
- `Fortify::confirmPasswordsUsing()`

We will call the above in our `AuthServiceProvider.php` file, inside the `boot()` method:

```php
// app/Providers/AuthServiceProvider.php

// ...
use App\Models\User;
use Laravel\Fortify\Fortify;
use Illuminate\Support\Facades\Auth;

class AuthServiceProvider extends ServiceProvider
{
    // ...

    public function boot()
    {
        $this->registerPolicies();

        Fortify::authenticateUsing(function ($request) {
            $validated = Auth::validate([
                'mail' => $request->email,
                'password' => $request->password
            ]);

            return $validated ? Auth::getLastAttempted() : null;
        });

        Fortify::confirmPasswordsUsing(function (User $user, $password) {
            return Auth::validate([
                'mail' => $user->email,
                'password' => $password,
            ]);
        });
    }
}
```

As you can see above, we are passing an array of the users credentials to the `Auth::validate()`
method. Most notibly, we set the `mail` key in this credentials array which is passed to the
LdapRecord authentication provider.

Upon a user attempting to sign in, a search query will be executed on your directory for a user
that contains the `mail` attribute equal to the entered `email` that the user has submitted
on your login form. The `password` key will not be used in the search.

If a user cannot be located in your directory, or they fail authentication, they will be
redirected to the login page normally with the "_Invalid credentials_" error message.

> You may also add extra key => value pairs in the `credentials` array to further scope
> the LDAP query. The `password` key is automatically ignored by LdapRecord.

#### Feature Configuration

Since we are synchronizing data from our LDAP server, we must disable several
features by commenting them out inside of the `config/fortify.php` file:

```php
// config/fortify.php

// Before:
'features' => [
    Features::registration(),
    Features::resetPasswords(),
    // Features::emailVerification(),
    Features::updateProfileInformation(),
    Features::updatePasswords(),
    Features::twoFactorAuthentication([
        'confirmPassword' => true,
    ]),
],

// After:
'features' => [
    // Features::registration(),
    // Features::resetPasswords(),
    // Features::emailVerification(),
    // Features::updateProfileInformation(),
    // Features::updatePasswords(),
    Features::twoFactorAuthentication([
        'confirmPassword' => true,
    ]),
],
```

> **Important**: You may keep `Features::registration()` enabled if you would like
> to continue accepting local application user registration. Keep in mind, if you
> continue to allow registration, you will need to either use multiple Laravel
> authentication guards, or setup the [login fallback](/docs/laravel/v1/auth/laravel-jetstream/#fallback-auth) feature.

Your application is now ready to authenticate LDAP users.
