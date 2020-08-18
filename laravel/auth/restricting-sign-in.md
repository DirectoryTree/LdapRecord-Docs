---
title: Restricting Sign In
description: Restricting LDAP authentication
extends: _layouts.laravel-documentation
section: content
---

# Restricting Sign In

LdapRecord-Laravel provides multiple ways you can restrict which users can sign in:

- [Using Only Manually Imported Users](#using-only-imported-users)
- [Using an Organizational Unit](#using-an-organizational-unit)
- Using a Group Membership
- Using a Whitelist

## Using Only Manually Imported Users {#using-only-imported-users}

To allow only [manually imported LDAP users](/docs/laravel/importing) who exist inside of your
database to sign in to your application, you must create an [authentication rule](/docs/laravel/auth/configuration#rules).

Let's create this rule using the below artisan command:

```bash
php artisan make:ldap-rule OnlyImportedUsers
```

> A new rule will be created inside `app/Ldap/Rules/OnlyImportedUsers.php`

Inside this rule, we will simply return whether the users Eloquent model exists:

```php
<?php

namespace App\Ldap\Rules;

use LdapRecord\Laravel\Auth\Rule;

class OnlyImportedUsers extends Rule
{
    /**
     * Check if the rule passes validation.
     *
     * @return bool
     */
    public function isValid()
    {
        return $this->model->exists;
    }
}
```

Once you've completed modifying the rule, add it into your `config/auth.php`
file where your LDAP user provider has been configured:

```php
'providers' => [
    // ...

    'ldap' => [
        'driver' => 'ldap',
        'model' => LdapRecord\Models\ActiveDirectory\User::class,
        'rules' => [
            App\Ldap\Rules\OnlyImportedUsers::class, // <-- Added here.
        ],
    ],
],
```

> Make sure you run `php artisan config:clear` if you are caching your configuration files.

Now when you attempt to sign in to your application, you will only be able to sign in 
with a user who already has been imported into your local applications database.

## Using an Organizational Unit {#using-an-organizational-unit}

To use an Organizational Unit which contains your users that you want to
allow sign in to your application, we will leverage LdapRecord [model scopes](/docs/models/#query-scopes).

In our application, we have an Organizational Unit named `Accounting` with the following Distinguished Name:

```
ou=Accounting,ou=Users,dc=local,dc=com
```

Let's create a new model scope using the below command:

```bash
php artisan make:ldap-scope OnlyAccountingUsers
```

Now inside of the generated scope, we will scope the query 

```php
<?php

namespace App\Ldap\Scopes;

use LdapRecord\Models\Model;
use LdapRecord\Models\Scope;
use LdapRecord\Query\Model\Builder;

class OnlyAccountingUsers implements Scope
{
    /**
     * Apply the scope to the given query.
     *
     * @param Builder $query
     * @param Model   $model
     *
     * @return void
     */
    public function apply(Builder $query, Model $model)
    {
        $query->in('ou=Accounting,ou=Users,dc=local,dc=com');
    }
}
```



## Using a Group Membership

## Using a Whitelist