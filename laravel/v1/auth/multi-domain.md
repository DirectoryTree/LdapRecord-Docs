---
title: Multi-Domain Authentication Guide
description: Setting up multi-domain authentication using LdapRecord-Laravel
---

# Multi-Domain Authentication

## Introduction

LdapRecord-Laravel allows you to authenticate users from as many LDAP directories as you'd like.

This useful when you have separate domains that are not joined in a trust.

## Step 1 - Configuration

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

> You may want these models to extend the [built-in models](/docs/core/v1/models/#predefined-models), as they
> include functionality that you do not need to build yourself. It's completely up to you.

### Configuring the Authentication Guards

For each of our LDAP connections, we will setup new [authentication providers](/docs/laravel/v1/auth/configuration),
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

## Step 2 - Login Controller Setup

To start authenticating users from both of your LDAP domains, we need to modify our `LoginController`.

> If you do not have a `LoginController`, follow Laravel's [Authentication Quick-Start](https://laravel.com/docs/laravel/v1/authentication#authentication-quickstart)
> guide to scaffold the controllers and views you need to continue below.

LdapRecord-Laravel comes with a built-in trait that makes authenticating users from multiple
directories easier. Go ahead and add it to the `LoginController`:

```php
// app/Http/Controllers/Auth/LoginController.php

use LdapRecord\Laravel\Auth\MultiDomainAuthentication;

class LoginController extends Controller
{
    use AuthenticatesUsers, MultiDomainAuthentication;

    // ...
```

Due to each domain requiring it's own `guard` that we've configured in our `config/auth.php` file,
we need to be able to determine which domain the user who is attempting to login in is from,
so we can tell Laravel which guard to use for authenticating the user.

Let's walk through two examples of how we can determine their domain:

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

Now, inside of our `LoginController.php`, we will override the `guard()` method:

```php
// app/Http/Controllers/Auth/LoginController.php

// ...

public function guard()
{
    return $this->getLdapGuard();
}
```

The `guard()` method is responsible for returning the name of the authentication
guard the user is signing into, or signing out of. We must override this to have
control over determining the suitable guard for the user signing in.

By default, the `MultiDomainAuthentication` trait will attempt to retrieve the users `guard`, from the request `domain` input value.

If you name your `<select>` input differently, simply override the `getLdapGuardFromRequest()` method and return its value instead.

```php
// app/Http/Controllers/Auth/LoginController.php

// ...

public function guard()
{
    return $this->getLdapGuard();
}

public function getLdapGuardFromRequest(Request $request)
{
    return $request->get('my-select-input');
}
```

### Email Address Suffix

In this example, we will be determining the users domain from their email addresses host name (eg. `@alpha.com` and `@bravo.com`).

Using this method, we do not need to modify our `login.blade.php` form. Instead, we will jump into
our `LoginController.php`, and override the `guard()` and `getLdapGuardFromRequest()` methods:

```php
// app/Http/Controllers/Auth/LoginController.php

// ...

public function guard()
{
    return $this->getLdapGuard();
}

public function getLdapGuardFromRequest(Request $request)
{
    $guards = [
        'alpha.com' => 'alpha',
        'bravo.com' => 'bravo',
    ];

    $domain = explode('@', $request->get('email'))[1];

    return $guards[$domain] ?? 'alpha';
}
```

The `guard()` method is responsible for returning the name of the authentication
guard the user is signing into, or signing out of. We must override this to have
control over determining the suitable guard for the user signing in.

The `getLdapGuardFromRequest()` method is responsible for determining the suitable
`guard` for the user using their given email address upon sign in.

If the user enters an email that is not available in our `$guards` array lookup, we will
return the `alpha` guard by default, and the authentication attempt will be made
to our `alpha` domain.

> You may wish to add a request validation rule instead to prevent users from signing
> in with invalid email domain. The way you implement this is totally up to you.

## Step 3 - Updating Your Web Routes

Having multiple authentication guards means that we need to update the `auth` middleware
that is covering our protected application routes inside of our `routes/web.php` file.

Luckily, this middleware accepts a list of guards you would like to use. You will need to add
both of the guards you created above for both LDAP domains to be able to access the same
protected routes:

> By default, if no guards are given to the Laravel `auth` middleware, it will attempt
> to use the `default` guard configured - **we do not want this behaviour**.

```php
// routes/web.php

Route::group(function () {
    // Both alpha and bravo domains can access these routes...
})->middleware('auth:alpha,bravo');
```

If you would like to restrict routes to certain domains, only include
one of them when adding the `auth` middleware to a route:

```php
// routes/web.php

Route::group(function () {
    // Only alpha domain users can access these routes...
})->middleware('auth:alpha');
```

This is extremely handy for permission management - as authenticated users from certain domains
can only access the routes that have been defined for their domain.
