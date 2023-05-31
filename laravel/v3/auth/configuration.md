---
title: Authentication Configuration
description: Configuring the LDAP authentication provider
---

# Authentication Configuration

## Introduction

To configure LDAP authentication, you must define or update a `provider` inside your `config/auth.php` file.

Let's walk through configuring both LDAP authentication mechanisms.

## Plain Authentication

To create a plain LDAP authentication provider, navigate to the `providers`
array, and paste the following `ldap` provider:

```php
// config/auth.php

'providers' => [
    // ...

    'ldap' => [
        'driver' => 'ldap',
        'model' => LdapRecord\Models\ActiveDirectory\User::class,
        'rules' => [],
    ],
],
```

If your application requires more than one LDAP connection, you must create a new provider for each connection.

This new provider must have its own unique `model` class set which must use your [alternate configured connection](/docs/core/v3/models#connections)
using the `$connection` property.

In the scenario of having multiple LDAP connections, it may be helpful to namespace the LDAP models
you create with the desired connection. For example:

```text
App\Ldap\DomainAlpha\User
```

This will allow you to segregate scopes, rules and other classes to their relating connection.

### Driver

The `driver` option must be `ldap` as this is what indicates to Laravel the proper authentication driver to use.

### Model

The `model` option must be the class name of your [LdapRecord model](/docs/core/v3/models). This model will be used
for fetching users from your directory.

### Rules

The `rules` option must be an array of class names of [authentication rules](#rules).

## Synchronized Database Authentication

To create a synchronized database LDAP authentication provider, navigate to the `providers` array,
and paste the following `ldap` provider:

> If your application requires two or more LDAP connections, you must create a new provider for each connection.

```php
// config/auth.php

'providers' => [
    // ...

    'ldap' => [
        'driver' => 'ldap',
        'model' => LdapRecord\Models\ActiveDirectory\User::class,
        'rules' => [],
        'database' => [
            'model' => App\Models\User::class,
            'sync_passwords' => false,
            'sync_attributes' => [
                'name' => 'cn',
                'email' => 'mail',
            ],
        ],
    ],
],
```

As you can see above, a `database` array is used to configure the association between your LDAP user and your Eloquent user.

### Database Model

The `database => model` key is the class name of the [Eloquent model](https://laravel.com/docs/eloquent) that will be
used for creating and retrieving LDAP users from your applications database.

> Be sure to add the required [trait and interface](/docs/laravel/v3/auth/database/installation) to this model as shown in the installation guide.

### Password Column

If your application uses a different password column than `password`, then you can configure
it using the `password_column` key inside your provider's configuration:

```php
'providers' => [
    // ...

    'ldap' => [
        // ...
        'database' => [
            // ...
            'password_column' => 'my_password_column',
        ],
    ],
],
```

You can also set the value to `false` if your database table does not have any password column at all:

```php
'providers' => [
    // ...

    'ldap' => [
        // ...
        'database' => [
            // ...
            'password_column' => false,
        ],
    ],
],
```

### Sync Passwords

The `database => sync_passwords` option enables password synchronization. Password synchronization captures and hashes
the users password **upon login** if they pass LDAP authentication. This helps in situations where you may want to
provide a "back up" option in case your LDAP server is unreachable, as well as a way of determining if a
users password is valid without having to call to your LDAP server and validate it for you.

> If you do not define the `sync_passwords` key or have it set `false`, a user is always applied a
> random 16 character hashed password. This hashed password is only set once upon initial
> import or login so no needless updates are performed on user records.

### Sync Attributes

The `database => sync_attributes` array defines a set of key-value pairs:

- The **key** of each array item is the column of your `users` database table
- The **value** is the _name_ of the users LDAP attribute to set the database value to

> You do not need to add your users `guid` or `domain` database columns. These are done automatically for you.

For further control on sync attributes, see the below [attribute handler](#attribute-handlers) feature.

### Sync Existing Records

The `database => sync_existing` array defines a set of key-value pairs:

- The **key** of each array item is the column of your `users` database table to query
- The **value** is the _name_ of the users LDAP attribute to query inside your database for

> If the LDAP attribute returns `null` for the given **value**, the value string will be used
> in the query instead. This is helpful to be able to use raw strings to scope your query by.

Let's walk through an example.

In our application, we have existing users inside our Laravel applications database:

| id  | name         | email             | password | guid   | domain |
| --- | ------------ | ----------------- | -------- | ------ | ------ |
| 1   | Steve Bauman | sbauman@local.com | ...      | `null` | `null` |
| 2   | John Doe     | jdoe@local.com    | ...      | `null` | `null` |

As you can see above, these users have `null` values for their `guid` and `domain` columns.

If you do not define a `sync_existing` array, and a user logs in with `sbauman@local.com`,
you will receive a SQL exception. This is because LdapRecord was unable to locate a local
database user using the users GUID. If this occurs, LdapRecord attempts to insert a new
user with the same email address.

To solve this issue, we will insert the following `sync_existing` array:

```php
'providers' => [
    // ...

    'ldap' => [
        // ...
        'database' => [
            // ...
            'sync_existing' => [
                'email' => 'mail',
            ],
        ],
    ],
],
```

Now when `sbauman@local.com` attempts to log in, if the user cannot be located
by their GUID, they will instead be located by their email address. Their
GUID, domain, and sync attributes you define will then synchronize.

### All Available Options Example

Here is a synchronized database provider fully configured with all available options set:

```php
// config/auth.php

'providers' => [
    // ...

    'ldap' => [
        'driver' => 'ldap',
        'model' => LdapRecord\Models\ActiveDirectory\User::class,
        'rules' => [],
        'database' => [
            'model' => App\Models\User::class,
            'sync_passwords' => true,
            'sync_attributes' => [
                'name' => 'cn',
                'email' => 'mail',
            ],
            'sync_existing' => [
                'email' => 'mail',
            ],
            'password_column' => 'password',
        ],
    ],
],
```

## Attribute Handlers

If you require logic for synchronizing attributes when users sign in to your application or are
being [imported](/docs/laravel/v3/auth/database/importing), you can create an attribute handler class
responsible for setting / synchronizing your database models attributes from their
LDAP model.

This class you define must have a `handle` method. This method must accept the LDAP model you
have configured as the first parameter and your Eloquent database model as the second.

For the example below, we will create a handler named `AttributeHandler.php` inside your `app/Ldap` folder:

> You do not need to call `save()` on your Eloquent database model.
> This is called for you after attribute synchronization.

```php
<?php

namespace App\Ldap;

use App\Models\User as DatabaseUser;
use App\Ldap\User as LdapUser;

class AttributeHandler
{
    public function handle(LdapUser $ldap, DatabaseUser $database)
    {
        $database->name = $ldap->getFirstAttribute('cn');
        $database->email = $ldap->getFirstAttribute('mail');
    }
}
```

> Attribute handlers are created using Laravel's `app()` helper, so you may type-hint
> any dependencies you require in your handlers constructor to be made available
> during synchronization.

Then inside your `config/auth.php` file for your provider, set the attribute handler class as the `sync_attributes` value:

```php
'providers' => [
    // ...

    'ldap' => [
        // ...
        'database' => [
            // ...
            'sync_attributes' => \App\Ldap\LdapAttributeHandler::class,
        ],
    ],
],
```

You may also add multiple if you'd prefer, or combine them with `key => value` pairs:

```php
// ...
'database' => [
    // ...
    'sync_attributes' => [
        'name' => 'cn',
        'email' => 'mail',
        \App\Ldap\MyFirstAttributeHandler::class,
        \App\Ldap\MySecondAttributeHandler::class,
    ],
],
```
