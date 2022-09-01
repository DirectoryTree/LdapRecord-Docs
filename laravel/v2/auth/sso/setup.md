---
title: Pass-through / SSO Authentication Setup
description: Setting up pass-through / SSO authentication
---

# Pass-through / SSO Setup

## Middleware

To enable single-sign-on in your Laravel application, insert the included `WindowsAuthenticate`
middleware on your middleware stack inside your `app/Http/Kernel.php` file:

```php
protected $middlewareGroups = [
    'web' => [
        // ...
        \LdapRecord\Laravel\Middleware\WindowsAuthenticate::class,
    ],
];
```

> The `WindowsAuthenticate` middleware uses the rules you have configured inside your `config/auth.php` file.
> A user may successfully authenticate against your LDAP server when visiting your site, but depending
> on your rules, may not be imported or logged in.

## Multi-Domain SSO

To be able to use multi-domain single-sign-on, your LDAP directory servers must first be joined in a trust.

Consider we have two domains: **alpha.local** and **bravo.local**.

If you have a web server that is joined to the **alpha.local** domain that is hosting your
Laravel application, it must allow users to authenticate to the **bravo.local** domain.

Once you have a working trust defined between your domains, you must follow the steps of
[setting up multi-domain authentication](/docs/laravel/v2/auth/multi-domain/).
You may skip step 2, if you do not need a login page for your users.

After completing the above linked guide, you must instruct the `WindowsAuthenticate` middleware to
utilize your LDAP authentication guards that you have configured in your `config/auth.php` file
by calling the `guards` method:

```php
// app/Providers/AuthServiceProvider.php

/**
 * Register any authentication / authorization services.
 *
 * @return void
 */
public function boot()
{
    $this->registerPolicies();

    WindowsAuthenticate::guards(['alpha', 'bravo']);
}
```

Or, if you prefer, you may define the `WindowsAuthenticate` middleware as a named middleware inside
your `app/Http/Kernel.php`, and insert the guard names in the definition of your routes:

```php
// app/Http/Kernel.php

/**
 * The application's route middleware.
 *
 * These middleware may be assigned to groups or used individually.
 *
 * @var array
 */
protected $routeMiddleware = [
    'auth' => \App\Http\Middleware\Authenticate::class,
    'auth.windows' => \LdapRecord\Laravel\Middleware\WindowsAuthenticate::class,
    // ...
],
```

Then, utilize it inside your routes file:

> **Important**: When guarding your routes that require authentication via the
> `auth` middleware, you must add both guard names into it as well.

```php
// routes/web.php

Route::middleware([
    'auth.windows:alpha,bravo',
    'auth:alpha,bravo',
])->group(function () {
    // ...
});
```

> **Important**:
> <br/><br/>
> The actual order of the middleware definition is critical here, so your users that are
> accessing your site through single-sign-on are logged into your application, _prior_
> to hitting the `auth` middleware, which validates that they are in-fact logged in.
> <br/><br/>
> Otherwise, they will be simply redirected to your login page.

## SSO Domain Verification

To prevent security issues using multiple-domain authentication using the `WindowsAuthenticate`
middleware, domain verification will be performed on the authenticating user.

This verification checks if the user's _domain name_ is contained inside of their
_full distinguished name_, which is retrieved from each of your configured LDAP guards.

> Only 'Domain Components' are checked in the user's distinguished name. More on this below.

To describe this issue in further detail -- the `WindowsAuthenticate` middleware retrieves all of your configured
authentication guards inside of your `config/auth.php` file. It then determines which one is using the `ldap`
driver, and attempts to locate the authenticating users from **each connection**.

Since there is the possibility of users having the same `sAMAccountName` on two separate domains,
LdapRecord must verify that the user retrieved from your domain is in-fact the user who
is connecting to your Laravel application via Single-Sign-On.

For example, if a user visits your Laravel application with the username of:

```text
ACME\sbauman
```

And LdapRecord locates a user with the distinguished name of:

```text
cn=sbauman,ou=users,dc=local,dc=com
```

They will be denied authentication. This is because the authenticating user has a domain of
`ACME`, but it is not contained inside of their distinguished name domain components (`dc`).

Using the same example, if the located user's distinguished name is:

```text
cn=sbauman,ou=users,dc=acme,dc=com
```

Then they will be allowed to authenticate, as their `ACME` domain exists inside
of their distinguished name domain components (`dc=acme`). Comparison against
each domain component will be performed in a **case-insensitive** manor.

If you would like to disable this check, you must call the static method `bypassDomainVerification`
on the `WindowsAuthenticate` middleware inside of your `AuthServiceProvider`:

> **Important**: This is a security issue if you use multi-domain authentication,
> since users who have the same `sAMAccountName` could sign in as each other.
> **You have been warned.** If however, you connect to only one domain
> inside your application, there is no security issue, and you may
> disable this check as shown below.

```php
// app/Providers/AuthServiceProvider.php

use LdapRecord\Laravel\Middleware\WindowsAuthenticate;

/**
 * Register any authentication / authorization services.
 *
 * @return void
 */
public function boot()
{
    $this->registerPolicies();

    WindowsAuthenticate::bypassDomainVerification();
}
```

### Swapping the Domain Extractor

> **Important**: This feature is available as of v2.3.0.

If you would like to override the default mechanism that extracts the domain
from the user's account that is retrieved from the PHP request key, call the
`WindowsAuthenticate::extractDomainUsing()` method and supply a callback.

The callback should return a string, or an array with two values.
The first being the user's username, the second being the user's domain.

The returned value(s) will be passed into the Domain Validator, for validation.

The first (and only) argument of the closure will be equal to the
retrieved value from the configured PHP `$_SERVER` key (default is `AUTH_USER`).

```php
WindowsAuthenticate::extractDomainUsing(function ($account) {
    [$username, $domain] = array_pad(
        array_reverse(explode('\\', $account)),
        2,
        null
    );

    return [$username, $domain];
});
```

To bypass extraction, supply a closure and return the account's value:

```php
WindowsAuthenticate::extractDomainUsing(function ($account) {
    return $account;
});
```

### Swapping the Domain Validator

> **Important**: This feature is available as of v2.3.0.

If you'd like to validate the user's domain in your own way,
call the `WindowsAuthenticate::validateDomainUsing()`
method, and supply either a closure, or a class.

The first argument will be the user's LdapRecord model, the second
will be the user's username and the third argument will be the
user's domain (extracted with the above Domain Extractor).

Return `true`/`false` whether the user has passed validation.

#### Using a Closure

Register the closure into the middleware:

```php
use LdapRecord\Models\Model;
use LdapRecord\Laravel\Middleware\WindowsAuthenticate;

WindowsAuthenticate::validateDomainUsing(function (Model $user, $username, $domain = null) {
    // Validate the user's domain.
});
```

#### Using a Class

Create the class with an `__invoke()` method:

```php
use LdapRecord\Models\Model;

class DomainValidator
{
    /**
     * Determine if the user passes domain validation.
     *
     * @param Model       $user
     * @param string      $username
     * @param string|null $domain
     *
     * @return bool
     */
    public function __invoke(Model $user, $username, $domain = null)
    {
        // Validate the user's domain.
    }
}
```

Register the class into the middleware:

```php
WindowsAuthenticate::validateDomainUsing(DomainValidator::class);
```

## Changing the Server Key

By default, the `WindowsAuthenticate` middleware uses the `AUTH_USER` key inside of PHP's `$_SERVER`
array (`$_SERVER['AUTH_USER']`). If you would like to change this, call the `serverKey` method on
the `WindowsAuthenticate` middleware inside of your `AuthServiceProvider`:

```php
// app/Providers/AuthServiceProvider.php

use LdapRecord\Laravel\Middleware\WindowsAuthenticate;

/**
 * Register any authentication / authorization services.
 *
 * @return void
 */
public function boot()
{
    $this->registerPolicies();

    WindowsAuthenticate::serverKey('PHP_AUTH_USER');
}
```

## Remember Single-Sign-On users

As of LdapRecord-Laravel version `v1.9.0`, users signed in to your application via the
`WindowsAuthenticate` middleware will no longer be automatically "remembered".

This shouldn't have any effect on your application, but if you need to re-enable
this feature, you must call the `rememberAuthenticatedUsers` method on the
`WindowsAuthenticate` middleware inside of your `AuthServiceProvider`:

```php
// app/Providers/AuthServiceProvider.php

use LdapRecord\Laravel\Middleware\WindowsAuthenticate;

/**
 * Register any authentication / authorization services.
 *
 * @return void
 */
public function boot()
{
    $this->registerPolicies();

    WindowsAuthenticate::rememberAuthenticatedUsers();
}
```

## Selective / Bypassing Single-Sign-On

Occasionally you may need to allow users who are not a part of the domain
to login to your application, as well as allowing domain users to
automatically sign in via Single-Sign-On.

Depending on your web servers operating system, this process can be different.

### Linux (HTTPD)

If you're using the Apache `httpd` server with plugins enabling the sharing of a domain joined user's
username via the `REMOTE_USER` server variable, you must update the `WindowsAuthenticate` middleware
to use this variable, instead of the default `AUTH_USER`.

To do this, call the `WindowsAuthenticate::serverKey()` method in your `AuthServiceProvider::boot()` method:

```php
// app/Providers/AuthServiceProvider.php

use LdapRecord\Laravel\Middleware\WindowsAuthenticate;

/**
 * Register any authentication / authorization services.
 *
 * @return void
 */
public function boot()
{
    $this->registerPolicies();

    WindowsAuthenticate::serverKey('REMOTE_USER');
}
```

If a user is not on a domain joined computer, then the `REMOTE_USER` variable will
be `null` and the `WindowsAuthenticate` middleware will be automatically bypassed,
allowing regular web application users to sign in.

### Windows (IIS)

A Windows hosted application with NTLM / Windows authentication enabled is unfortunately all-or-nothing
on your entire web application instance. This means, you cannot enable a single HTTP endpoint in your
application to use Single-Sign-On or exempt a portion of your application. However, there is a
workaround that is used frequently in the industry.

The goal is to have two URL's that point to the same Laravel application. One has Windows authentication
enabled, and another does not. This is typically identified by an `sso` sub-domain:

```html
<!-- Standard URL -->
my-app.com

<!-- Single-Sign-On URL -->
sso.my-app.com
```

To do this, you must create a new IIS application instance and point to the same Laravel application.
Then, you simply have Windows authentication enabled on one instance, and left disabled on another.

Nothing needs to be done in your Laravel application. The `WindowsAuthenticate` middleware
will only attempt to authenticate users when the `AUTH_USER` server key is present,
so it can remain in the global middleware stack.

## Forcing logouts on non Single-Sign-On users

If a user successfully authenticates to your Laravel application through single-sign-on, and
their LDAP account happens to be deleted or disabled, the user will remain authenticated
to your application for the duration of your Laravel application's session.

If you would like all users in your application to be signed out automatically
if SSO credentials are not available from your web server, call the
`logoutUnauthenticatedUsers` method on the `WindowsAuthenticate`
middleware in your `AuthServiceProvider::boot()` method:

> **Important**: Only enable this feature if Single-Sign-On is the only way
> you authenticate users. If a non-Single-Sign-On user has a session open,
> it will be ended automatically on their next request.

```php
// app/Providers/AuthServiceProvider.php

use LdapRecord\Laravel\Middleware\WindowsAuthenticate;

/**
 * Register any authentication / authorization services.
 *
 * @return void
 */
public function boot()
{
    $this->registerPolicies();

    WindowsAuthenticate::logoutUnauthenticatedUsers();
}
```
