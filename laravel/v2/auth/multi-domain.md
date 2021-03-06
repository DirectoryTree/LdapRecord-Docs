---
title: Multi-Domain Authentication Guide
description: Setting up multi-domain authentication using LdapRecord-Laravel
---

# Multi-Domain Authentication

## Introduction

LdapRecord-Laravel allows you to authenticate users from as many LDAP directories as you'd like.

This useful when you have separate domains that are not joined in a trust.

## Configuration

To begin, you must create two separate LdapRecord models for each of your domains.

Having two separate models allows you to configure their connections independently.

### Configuring the LDAP Connections

For this guide, we will have two example domains named `Alpha` and `Bravo`. We first
need to set up these domain connections in our `ldap.php` configuration file:

```php
// config/ldap.php

// ...

'connections' => [
    'alpha' => [
        // ...
    ],

    'bravo' => [
        // ...
    ],
],
```

> Remember to test your connections using `php artisan ldap:test` to ensure
> you are able to connect to each of your LDAP servers.

Now we have our connections configured, you must create a `User` model for each one.

Let's go ahead and create them by running the below commands:

```bash
php artisan make:ldap-model Alpha\User
```

```bash
php artisan make:ldap-model Bravo\User
```

> The `Alpha` and `Bravo` sub-directories will be created for you automatically.

Then, we must edit their connections to reflect the connection name in the `config/ldap.php` file:

```php
// app/Ldap/Alpha/User.php

class User extends Model
{
    protected $connection = 'alpha';

    // ...
}
```

```php
// app/Ldap/Bravo/User.php

class User extends Model
{
    protected $connection = 'bravo';

    // ...
}
```

> You may want these models to extend the [built-in models](/docs/core/v2/models/#predefined-models), as they
> include functionality that you do not need to build yourself. It's completely up to you.

### Configuring the Authentication Guards

For each of our LDAP connections, we will setup new [authentication providers](/docs/laravel/v2/auth/configuration),
as well as their own guard inside of our `config/auth.php` file:

```php
// config/auth.php

'guards' => [
    'alpha' => [
        'driver' => 'session',
        'provider' => 'alpha',
    ],

    'bravo' => [
        'driver' => 'session',
        'provider' => 'bravo',
    ],
],

'providers' => [
    // ...

    'alpha' => [
        // ...
        'model' => App\Ldap\Alpha\User::class,
    ],

    'bravo' => [
        // ...
        'model' => App\Ldap\Bravo\User::class,
    ],
],
```

## Authentication Approaches

Due to each domain requiring it's own `guard` that we've configured in our `config/auth.php` file,
we need to be able to determine which domain the user who is attempting to login in is from,
so we can tell Laravel which guard to use for authenticating the user.

Let's walk through two examples of how we could determine their domain:

| Example                                              | Description                                                        |
| ---------------------------------------------------- | ------------------------------------------------------------------ |
| [Domain Selection](#authenticating-domain-selection) | Using a `<select>` dropdown                                        |
| [Email Address Suffix](#authenticating-email-suffix) | Using the users email address suffx / hostname (eg. `@domain.com`) |

### Domain Selection

In this example, we will add an HTML `<select>` input containing an `<option>` for each
domain we want to allow users to login to. This allows the user to select the domain
from the dropdown, enter their credentials, and then attempt signing in.

First, we will open up our `login.blade.php` file, and add the select option:

```html
<!-- resources/views/auth/login.blade.php -->

<form method="POST" action="{{ route('login') }}">
    @csrf

    <div class="form-group row">
        <select name="domain" class="form-control">
            @foreach(['alpha' => 'Alpha', 'bravo' => 'Bravo'] as $guard => $name)
                <option value="{{ $guard }}" {{ old('domain') == $guard ? 'selected' : '' }}>{{ $name }}</option>
            @endforeach
        </select>
    </div>

    <!-- ... -->
</form>
```

Then we must update our authentication logic to use the guard the user has selected during login:

```php
$credentials = [
    'mail' => $request->email,
    'password' => $request->password,
];

Auth::shouldUse($request->domain);

if (Auth::attempt($credentials)) {
    return redirect('/dashboard');
}
```

### Email Address Suffix

In this example, we will be determining the users domain from their email addresses host name (eg. `@alpha.com` and `@bravo.com`).

Using this method, we will simply split out their email address domain and use a lookup table to retrieve the proper guard:

```php
$credentials = [
    'mail' => $request->email,
    'password' => $request->password,
];

$guards = [
    'alpha.com' => 'alpha',
    'bravo.com' => 'bravo',
];

$domain = explode('@', $request->email)[1];

$guard = $guards[$domain] ?? 'alpha';

Auth::shouldUse($guard);

if (Auth::attempt($credentials)) {
    return redirect('/dashboard');
}
```

## Updating Your Web Routes

Having multiple authentication guards means that we need to update the `auth` middleware
that is covering our protected application routes inside of our `routes/web.php` file.

Luckily, this middleware accepts a comma separated list of guards you would like to
protect your routes by. You will need to add both of the guards you created above:

> By default, if no guards are given to the Laravel `auth` middleware, it will attempt
> to use the `default` guard configured inside of your `config/auth.php` file.

**Before**:

```php
// routes/web.php

Route::middleware('auth')->group(function () {
    // ...
})
```

**After**:

```php
// routes/web.php

Route::middleware('auth:alpha,bravo')->group(function () {
    // Both alpha and bravo domains can access these routes...
});
```

If you would like to restrict routes to certain domains, only include
one of them when adding the `auth` middleware to a route:

```php
// routes/web.php

Route::group(function () {
    // Only alpha domain users can access these routes...
})->middleware('auth:alpha');
```

This is extremely handy for permission management - as authenticated
users from certain domains can only access the routes that have
been defined for their domain.

You are now ready to authenticate users with multiple domains.
