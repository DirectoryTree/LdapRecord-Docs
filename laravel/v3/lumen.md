---
title: LdapRecord-Lumen
description: Getting started with LdapRecord in Lumen.
---

# LdapRecord-Lumen

## Introduction

LdapRecord-Lumen gives you the features of LdapRecord-Laravel using Lumen.

## Installation

LdapRecord-Lumen requires the following:

| Requirements                                              |
| --------------------------------------------------------- |
| PHP >= 7.3                                                |
| Lumen >= 5.6                                              |
| PHP LDAP extension enabled                                |
| An LDAP server (Active Directory, OpenLDAP, FreeIPA etc.) |

Require LdapRecord-Laravel via [composer](https://getcomposer.org/):

```bash
composer require directorytree/ldaprecord-lumen
```

Once composer completes, register the `LdapServiceProvider` inside of your `bootstrap/app.php` file:

```php
// bootstrap/app.php

$app->register(\LdapRecord\Lumen\LdapServiceProvider::class);
```

## Configuration

Publish the `ldap.php` configuration file via the `make:ldap-config` command:

```bash
php artisan make:ldap-config
```

A new LDAP configuration file will be created in your `config` directory.

Then, inside of your `.env` file, paste the following to configure your LDAP connection:

```text
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

## Usage

You can now begin using LdapRecord inside of your Lumen application:

```php
// routes/web.php

use LdapRecord\Models\ActiveDirectory\User;

$router->get('/users', function () {
    return User::get();
});
```
