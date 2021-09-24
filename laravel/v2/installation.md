---
title: Installation
description: LdapRecord-Laravel Install Guide
---

# Installation

LdapRecord-Laravel requires the following:

| Requirements                                              |
| --------------------------------------------------------- |
| PHP >= 7.3                                                |
| Laravel >= 5.6                                            |
| PHP LDAP extension enabled                                |
| An LDAP server (Active Directory, OpenLDAP, FreeIPA etc.) |

Require LdapRecord-Laravel via [composer](https://getcomposer.org/):

```bash
composer require directorytree/ldaprecord-laravel
```

## Configuration

> **Important**: It's recommended to only use one approach listed in this guide below. Using
> both approaches to configure your LDAP connections may lead to unexpected results.

### Using a configuration file

To publish the `ldap.php` configuration file, execute the below artisan command:

```bash
php artisan vendor:publish --provider="LdapRecord\Laravel\LdapServiceProvider"
```

The `ldap.php` file will then be created inside of your applications `config`, directory.

Inside the configuration file, setup your LDAP connections, or paste the following in your `.env` file to get started quicky:

> Review the [configuration documentation](/docs/core/v2/configuration) to see what each option is used for.

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

If you'd like to configure more options that are specified in the `ldap.php`
file, you may create your own env variables to control those options.

### Using an environment file (.env)

LDAP connections may be configured directly in your `.env` without having to publish any configuration files.

To get started, you may set the LDAP connections by specifying a comma-separated list of connection names using the `LDAP_CONNECTIONS` variable:

```dotenv
LDAP_CONNECTIONS=alpha,bravo
```

Next, set your default LDAP connection by using the `LDAP_CONNECTION` variable:

```dotenv
LDAP_CONNECTION=alpha
```

Then, to configure options for each connection you have specified, you must suffix them by `LDAP_{CONNECTION}_`:

```dotenv
LDAP_LOGGING=true

LDAP_CONNECTION=alpha

LDAP_CONNECTIONS=alpha,bravo

LDAP_ALPHA_HOSTS=10.0.0.1,10.0.0.2
LDAP_ALPHA_USERNAME="cn=admin,dc=alpha,dc=com"
LDAP_ALPHA_PASSWORD=secret
LDAP_ALPHA_PORT=389
LDAP_ALPHA_BASE_DN="dc=alpha,dc=com"
LDAP_ALPHA_TIMEOUT=5
LDAP_ALPHA_SSL=false
LDAP_ALPHA_TLS=false

LDAP_BRAVO_HOSTS=11.0.0.1,11.0.0.2
LDAP_BRAVO_USERNAME="cn=admin,dc=bravo,dc=com"
LDAP_BRAVO_PASSWORD=secret
LDAP_BRAVO_PORT=389
LDAP_BRAVO_BASE_DN="dc=bravo,dc=com"
LDAP_BRAVO_TIMEOUT=5
LDAP_BRAVO_SSL=false
LDAP_BRAVO_TLS=false
```

To configure [PHP LDAP options](https://www.php.net/manual/en/function.ldap-set-option.php) for
a connection using an env variable, use the configuration name pattern `LDAP_{CONNECTION}_OPT_{NAME}`.

For example, you may configure the option `LDAP_OPT_X_TLS_CERTFILE` for a connection named `alpha` like so:

```dotenv
LDAP_ALPHA_OPT_X_TLS_CERTFILE=/usr/bin/etc/path
```
