---
title: Database Authentication Setup
description: Logging in with LdapRecord-Laravel database authentication
---

# Logging In

> **Important**: Before we begin, it's paramount to understand that LdapRecord does not support credentialess 
> authentication (unless your server supports anonymous binding). If you attempt to configure an LDAP 
> connection without credentials, your user will be logged out after the request ends. Please 
> see the [configuration guide](/docs/laravel/v3/configuration) for more information.

Once you have [configured a new authentication provider](/docs/laravel/v3/auth/database/configuration),
you're ready to start logging users into your application.

Logging in is simple -- you only need to use Laravel's `Auth` facade:

```php
use Illuminate\Support\Facades\Auth;

$credentials = [
    'mail' => 'jdoe@local.com',
    'password' => 'password',
];

if (Auth::attempt($credentials)) {
    $user = Auth::user();

    return redirect('/dashboard')->with([
        'message' => "Welcome back, {$user->name}"
    ]);
}
```

As you can see above, we set the `mail` key which is passed to the LdapRecord authentication provider.

A search query will be executed on your LDAP directory for a user that contains the `mail` attribute
equal to the entered email address. The `password` key will not be used in the search.

If you wish to log in a user by their username instead, simply change the `mail` key
to whichever attribute you would like to locate the user by in your LDAP directory.
For example, `samaccountname`:

> **Important**: Keep in mind you will have to alter your `sync_attributes` inside your `config/auth.php`
> file to synchronize this field into your `users` database record if you have not already done so.

```php
use Illuminate\Support\Facades\Auth;

$credentials = [
    'samaccountname' => 'jdoe',
    'password' => 'password',
];

if (Auth::attempt($credentials)) {
    //
}
```
