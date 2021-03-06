---
title: Installation
description: LdapRecord-Laravel Install Guide
---

# Installation

LdapRecord-Laravel requires the following:

| Requirements                                              |
| --------------------------------------------------------- |
| PHP >= 7.2                                                |
| Laravel >= 5.6                                            |
| PHP LDAP extension enabled                                |
| An LDAP server (Active Directory, OpenLDAP, FreeIPA etc.) |

Require LdapRecord-Laravel via [composer](https://getcomposer.org/):

```bash
composer require directorytree/ldaprecord-laravel
```

Then, publish the `ldap.php` configuration file via the `artisan publish` command:

```bash
php artisan vendor:publish --provider="LdapRecord\Laravel\LdapServiceProvider"
```

Inside the published `config/ldap.php` file, setup your LDAP connections, or paste the following in your `.env` file:

> Review the [configuration documentation](/docs/core/v1/configuration) to see what each option is used for.

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

The `default` connection you specify is the LDAP connection that will be used for
models you define that do not have a [configured connection](/docs/core/v1/models#connections).

If you're using multiple LDAP connections, you will have to manually create and assign
unique `.env` variables for the above keys and then update the `config/ldap.php`
with the new variables.

For example:

```dotenv
#...

ALPHA_LDAP_HOST=192.160.0.1
ALPHA_LDAP_USERNAME="cn=user,dc=alpha,dc=com"

BRAVO_LDAP_HOST=192.170.0.1
BRAVO_LDAP_USERNAME="cn=user,dc=bravo,dc=com"
```
