---
title: Plain Authentication Setup
description: Logging in with LdapRecord-Laravel plain authentication
---

# Logging In

Once you have [configured a new authentication provider](/docs/laravel/v2/auth/plain/configuration),
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
        'message' => "Welcome back, {$user->getName()}"
    ]);
}
```

As you can see above, we set the `mail` key which is passed to the LdapRecord authentication provider.

A search query will be executed on your LDAP directory for a user that contains the `mail` attribute
equal to the entered email address. The `password` key will not be used in the search.

If you wish to login a user by their username instead, simply change the `mail` key
to whichever attribute you would like to locate the user by in your LDAP directory.
For example, `samaccountname`:

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
