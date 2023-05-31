---
title: Database Authentication Configuration
description: Configuring the database LDAP authentication provider
---

# Database Auth Configuration

## Introduction

To configure a synchronized database LDAP authentication provider, navigate
to the `providers` array inside of your `config/auth.php` file, and paste
the following `users` provider:

> You will have to remove/alter the default `users` provider, or create your own.

```php
// config/auth.php

'providers' => [
    // ...

    'users' => [
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

> **Important**:
> <br/><br/>
> If your application requires more than one LDAP connection, you
> must create a new provider for each connection.
> <br/><br/>
> This new provider must have its own unique `model` class which must use your
> [alternate configured connection](/docs/core/v2/models#connections) name
> using the `$connection` property.

In the scenario of having multiple LDAP connections, it may be helpful to namespace the LDAP models
you create with the desired connection. For example:

```text
App\Ldap\DomainAlpha\User
```

This will allow you to segregate scopes, rules and other classes to their relating connection.

## Driver

The `driver` option must be `ldap` as this is what indicates to Laravel the proper authentication driver to use.

## Model

The `model` option must be the class name of your [LdapRecord model](/docs/core/v2/models). This model will be used
for fetching users from your directory.

## Rules

The `rules` option must be an array of [authentication rule](#rules) class name's.

### Overview

LDAP authentication rules give you the ability to allow or deny users from signing into your
application using a condition you would like to apply. These rules are executed **after**
a user successfully passes LDAP authentication against your configured server.

Think of them as a final authorization gate before they are allowed in.

> Authentication rules are never executed if a user fails LDAP authentication.

### Creating Rules

Let's create an LDAP rule that only allows members of our domain `Administrators` group.

To create an authentication rule, call the `make:ldap-rule` command:

```bash
php artisan make:ldap-rule OnlyAdministrators
```

A rule will then be created in your applications `app/Ldap/Rules` directory:

```php
<?php

namespace App\Ldap\Rules;

use LdapRecord\Laravel\Auth\Rule;

class OnlyAdministrators extends Rule
{
    /**
     * Check if the rule passes validation.
     *
     * @return bool
     */
    public function isValid()
    {
        //
    }
}
```

In the authentication rule, there are two properties made available to us.

- A `$user` property that is the **LdapRecord** model of the authenticating user
- A `$model` property that is the **Eloquent** model of the authenticating user

Now, we will update the `isValid` method to check the LDAP users `groups` relationship to see if they are a member:

```php
<?php

namespace App\Ldap\Rules;

use LdapRecord\Laravel\Auth\Rule;
use LdapRecord\Models\ActiveDirectory\Group;

class OnlyAdministrators extends Rule
{
    public function isValid()
    {
        $administrators = Group::find('cn=Administrators,dc=local,dc=com');

        return $this->user->groups()->recursive()->exists($administrators);
    }
}
```

> We call the `recursive` method on the relationship to make sure
> that we load _groups of groups_ in case the user is not an
> immediate member of the `Administrators` group.

Once we have our rule defined, we will add it into our authentication provider in the `config/auth.php` file:

```php
'providers' => [
    // ...

    'users' => [
        'driver' => 'ldap',
        'model' => LdapRecord\Models\ActiveDirectory\User::class,
        'rules' => [
            App\Ldap\Rules\OnlyAdministrators::class,
        ],
        'database' => [
            // ...
        ],
    ],
],
```

Now when you attempt to login to your application with a LDAP user that successfully passes
LDAP authentication, they will need to be a member of the `Administrators` group.

> If you are caching your configuration, make sure you re-run `config:cache` to re-cache your modifications.

## Database Model

The `database => model` key is the class name of the [Eloquent model](https://laravel.com/docs/laravel/v2/eloquent) that will be
used for creating and retrieving LDAP users from your applications database.

> Be sure to add the required [trait and interface](/docs/laravel/v2/auth/database/installation#add-required-trait-and-interface) to this model as shown in the installation guide.

## Sync Password Column

If your application uses a different password column than `password`, then you can configure
it using the `password_column` key inside of your providers configuration:

```php
'providers' => [
    // ...

    'users' => [
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

    'users' => [
        // ...
        'database' => [
            // ...
            'password_column' => false,
        ],
    ],
],
```

## Sync Passwords

The `database => sync_passwords` option enables password synchronization.

Password synchronization captures and hashes the users password **upon login**
if they pass LDAP authentication. This helps in situations where you may want
to provide a "back up" option in case your LDAP server is unreachable, as
well as a way of determining if a users password is valid without having
to call to your LDAP server and validate it for you.

> If you do not define the `sync_passwords` key or have it set `false`, a user is always applied a
> random 16 character hashed password. This hashed password is only set once upon initial
> import or login so no needless updates are performed on user records.

## Sync Attributes

The `database => sync_attributes` array defines a set of key-value pairs that
describe which database column should be set and to which LDAP property:

```php
'sync_attributes' => [
    'email' => 'mail',
    'name' => 'cn',
]
```

- The **key** of each array item is the attribute of your `User` Eloquent model
- The **value** is the _name_ of the users LDAP attribute to set the Eloquent model attribute value to

> You do not need to add your users `guid` or `domain` database columns. These are done automatically for you.

For further control on sync attributes, see the below [attribute handler](#attribute-handlers) feature.

## Sync Existing Records

The `database => sync_existing` array defines a set of key-value pairs that
describe how existing database users should be sychronized:

```php
'sync_existing' => [
    'email' => 'mail',
],
```

- The **key** of each array item is the column of your `users` database table to query
- The **value** is the _name_ of the users LDAP attribute to set the database value to

> Alternatively inside of each `value` key, you may provide an array with an `attribute`
> key containing the LDAP attribute name and an `operator` key to use for the
> query when retrieving a record from the database. More on this below.

> **Important**: If the LDAP attribute returns `null` for the given **value**,
> the actual value will be used in the query instead. This is helpful to be
> able to use raw strings to scope your query by.

Let's walk through an example.

In our application, we have existing users inside of our database:

| id  | name         | email             | password | guid   | domain |
| --- | ------------ | ----------------- | -------- | ------ | ------ |
| 1   | Steve Bauman | sbauman@local.com | ...      | `null` | `null` |
| 2   | John Doe     | jdoe@local.com    | ...      | `null` | `null` |

As you can see above, these users have `null` values for their `guid` and `domain` columns.

If you do not define a `sync_existing` array and a user logs in with `sbauman@local.com`,
you will receive a SQL exception. This is because LdapRecord was unable to locate a local
database user using the users GUID. If this occurs, LdapRecord will attempt to insert a
new user with the same email address.

To resolve this issue, we will insert the following `sync_existing` array:

```php
'providers' => [
    // ...

    'users' => [
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

### Database Compatibility

In some database drivers, such as Postgres, there is case-sensitivity when executing `where`
clauses with the equals (`=`) operator. Consider the following data in your database:

| id  | name         | email             | password | guid   | domain |
| --- | ------------ | ----------------- | -------- | ------ | ------ |
| 1   | Steve Bauman | sbauman@local.com | ...      | `null` | `null` |
| 2   | John Doe     | jdoe@local.com    | ...      | `null` | `null` |

However, inside of the LDAP server, the `mail` attribute for Steve's record
is actually `SBauman@local.com`. While he could successfully authenticate,
the existing record would not be found in our database due to Postgres'
more strict SQL grammar. Changing the `sync_existing` configuration
to the following array syntax would allow us to change the
operator from an equals (`=`) to an `ilike`.

```php
'sync_existing' => [
    'email' => [
        'attribute' => 'mail',
        'operator' => 'ilike',
    ],
],
```

By replacing the value of the array to be an array with the `attribute` and `operator`
keys, we can fine-tune the query syntax to be more flexible to your needs.

## Attribute Handlers

If you require logic for synchronizing attributes when users sign into your application or are
being [imported](/docs/laravel/v2/auth/database/importing), you can create an attribute handler class
responsible for setting / synchronizing your database models attributes from their
LDAP model.

This class you define must have a `handle` method. This method must accept the LDAP model you
have configured as the first parameter and your Eloquent database model as the second.

For the example below, we will create a handler named `AttributeHandler.php` inside of your `app/Ldap` folder:

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

Then inside of your `config/auth.php` file for your provider, set the attribute handler class as the `sync_attributes` value:

```php
'providers' => [
    // ...

    'users' => [
        // ...
        'database' => [
            // ...
            'sync_attributes' => \App\Ldap\AttributeHandler::class,
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

> Attributes you specify are synchronized _in order_ (first to last), so you may
> access the already synchronized attributes in subsequent attribute handlers.

## All Available Options

Below is a synchronized database provider that is configured with all available options:

```php
// config/auth.php

'providers' => [
    // ...

    'users' => [
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
