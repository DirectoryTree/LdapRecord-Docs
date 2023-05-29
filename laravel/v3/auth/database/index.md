---
title: Database Authentication Overview
description: LdapRecord-Laravel database synchronization authentication overview
---

# Database Authentication Overview

Synchronized Database LDAP Authentication means that an LDAP user which successfully passes LDAP authentication
will be created & synchronized to your local application's database. This is helpful as you can attach typical
relational database information to them, such as blog posts, attachments, etc.

When a user is successfully authenticated, the Laravel `Auth::user()` method
will return an instance of your configured **Eloquent** database model:

```php
use Illuminate\Support\Facades\Auth;

$credentials = [
    'mail' => 'jdoe@local.com',
    'password' => 'secret',
];

if (Auth::attempt($credentials)) {
    $user = Auth::user();

    // Returns true:
    $user instanceof \App\Models\User;
}
```

Subsequent requests to your application with logged-in users will retrieve the
logged-in user from your database, rather than your LDAP directory. This means
your application will stay operational if connectivity to your LDAP server
is dropped.
