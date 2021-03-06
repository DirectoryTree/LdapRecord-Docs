---
title: Quickstart
description: LdapRecord-Laravel Quickstart Guide
---

# Quickstart

## Introduction

LdapRecord-Laravel requires the following:

| Requirements                                              |
| --------------------------------------------------------- |
| PHP >= 7.2                                                |
| Laravel >= 5.6                                            |
| PHP LDAP extension enabled                                |
| An LDAP server (Active Directory, OpenLDAP, FreeIPA etc.) |

## Install, Setup & Usage

### Step 1 - Install LdapRecord-Laravel

Require LdapRecord-Laravel via [composer](https://getcomposer.org/):

```bash
composer require directorytree/ldaprecord-laravel
```

### Step 2 - Publish the LDAP configuration file

```bash
php artisan vendor:publish --provider="LdapRecord\Laravel\LdapServiceProvider"
```

### Step 3 - Configure your LDAP connection

Paste these environment variables into your `.env` file, and configure each option as necessary:

```dotenv
LDAP_LOGGING=true
LDAP_CONNECTION=default
LDAP_HOST=127.0.0.1
LDAP_USERNAME="cn=user,dc=local,dc=com"
LDAP_PASSWORD=secret
LDAP_PORT=389
LDAP_BASE_DN="dc=local,dc=com"
LDAP_TIMEOUT=5
LDAP_SSL=false
LDAP_TLS=false
```

View the core [configuration](/docs/core/v1/configuration) documentation for more information on each option.

### Step 4 - Use LdapRecord

To begin, you may either use the built-in [models that LdapRecord comes with](/docs/core/v1/models#predefined-models),
or you may create your own models that reference the connection you have created in your `config/ldap.php` file.

Call the below command to create a new LdapRecord model:

```bash
php artisan make:ldap-model User
```

Then use it in your application:

```php
<?php

namespace App\Http\Controllers;

use App\Ldap\User;

class LdapUserController extends Controller
{
    public function index()
    {
        $users = User::get();

        return view('ldap.users.index', ['users' => $users]);
    }
}
```

### Step 5 - Setup Authentication

View the [authentication quickstart guide](/docs/laravel/v1/auth/quickstart) if you require LDAP authentication in your application.
