---
title: Plain Authentication Overview
description: LdapRecord-Laravel plain authentication overview
---

# Plain Authentication Overview

If you have an application that doesn't require any user data to be synchronized to your database,
then you can utilize plain LDAP authentication.

> **It is paramount to understand that upon every request a logged-in user makes to your application,
> an LDAP search request will be made to retrieve them**. If you do not want this to occur, you must
> use [synchronized database authentication](/docs/laravel/v3/auth/database).

When a user is successfully authenticated, the Laravel `Auth::user()` method
will return an instance of your configured **LdapRecord** model:

```php
use Illuminate\Support\Facades\Auth;

$credentials = [
    'mail' => 'jdoe@local.com',
    'password' => 'secret',
];

if (Auth::attempt($credentials)) {
    $user = Auth::user();

    // Returns true:
    $user instanceof \LdapRecord\Models\Model;
}
```
