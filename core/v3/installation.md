---
title: Installation
description: Installing LdapRecord
---

# Installation

LdapRecord requires the following:

| Requirements                                              |
|-----------------------------------------------------------|
| PHP >= 8.1                                                |
| PHP LDAP extension enabled                                |
| An LDAP server (Active Directory, OpenLDAP, FreeIPA etc.) |

LdapRecord uses [Composer](https://getcomposer.org) for installation.

After installing Composer, run the following command in the root directory of your project:

```bash
composer require directorytree/ldaprecord
```

If your application doesn't already require Composer's autoload, you will need to do it manually.

Insert the following line at the top of your projects PHP script (usually `index.php`):

```php
require __DIR__ . '/vendor/autoload.php';
```
