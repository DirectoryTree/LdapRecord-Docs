---
title: Authentication Setup & Features
description: LdapRecord-Laravel authentication setup guide
extends: _layouts.laravel-documentation
section: content
---

# Authentication Setup

Once you have [configured a new authentication provider](/docs/laravel/auth/configuration),
you will have to set up your authentication guard to use this new provider as the default.

For this example, we will change our default `web` guard to use our new `ldap` provider:

```php
// config/auth.php

'guards' => [
    'web' => [
        'driver' => 'session',
        'provider' => 'ldap', // Changed from 'users'
    ],
    
    // ...
],
```

## Authentication Scaffolding

LdapRecord-Laravel supports both [Laravel Jetstream](https://jetstream.laravel.com) and [Laravel UI](https://github.com/laravel/ui)
(deprecated) out-of-the-box. Select whichever authentication scaffolding you feel suits your application best.

- [Laravel Jetstream](/docs/laravel/auth/laravel-jetstream)
- [Laravel UI (deprecated)](/docs/laravel/auth/laravel-ui)
