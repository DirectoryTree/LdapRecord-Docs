---
title: Restricting Login
description: Restricting LDAP authentication login
---

# Restricting Login

## Introduction

LdapRecord-Laravel provides various ways you can prevent certain users
from logging into your application. Let's walk through some approaches.

## Using a Group Membership

To use a group membership for authorizing signing in to your application, we
will use an [authentication rule](/docs/laravel/v3/auth/configuration#rules).

In our example application, we want to only allow users who are members of
a single group to authenticate. This group will be called `Help Desk`.

Let's create our new authentication rule by running the below command:

```bash
php artisan ldap:make:rule OnlyHelpDeskUsers
```

> A new rule will be created inside `app/Ldap/Rules/OnlyHelpDeskUsers.php`

In the newly generated rule, we can check for group membership in
various ways, as well as check for nested group membership, and
even for multiple group memberships.

Let's walk through each example.

### Checking for a single group

When checking for a single group, we will use the relation `exists()` method:

```php
use LdapRecord\Models\Model as LdapRecord;
use Illuminate\Database\Eloquent\Model as Eloquent;

/**
 * Check if the rule passes validation.
 *
 * @return bool
 */
public function passes(LdapRecord $user, Eloquent $model = null): bool
{
    return $user->groups()->exists(
        'cn=Help Desk,ou=Groups,dc=local,dc=com'
    );
}
```

With the `exists()` method, we can also use an LdapRecord `Model` instance:

> This approach is useful, so an exception will be thrown when the group cannot be located.

```php
public function passes(LdapRecord $user, Eloquent $model = null): bool
{
    return $user->groups()->exists(
        Group::findOrFail('cn=Help Desk,ou=Groups,dc=local,dc=com')
    );
}
```

Or; A Common Name (`cn`):

```php
public function passes(LdapRecord $user, Eloquent $model = null): bool
{
    return $user->groups()->exists('Help Desk');
}
```

### Checking for multiple groups

To check that the user has **all** of a given set of groups, we will use the `exists()` method:

```php
public function passes(LdapRecord $user, Eloquent $model = null): bool
{
    return $user->groups()->exists(
        'cn=Help Desk,ou=Groups,dc=local,dc=com',
        'cn=Site Admins,ou=Groups,dc=local,dc=com'
    );
}
```

We can also use `Model` instances:

```php
public function passes(LdapRecord $user, Eloquent $model = null): bool
{
    return $user->groups()->exists([
        Group::findOrFail('cn=Help Desk,ou=Groups,dc=local,dc=com'),
        Group::findOrFail('cn=Site Admins,ou=Groups,dc=local,dc=com'),
    ]);
}
```

Or; Common Names (`cn`):

```php
public function passes(LdapRecord $user, Eloquent $model = null): bool
{
    return $user->groups()->exists([
        'Help Desk', 'Site Admins'
    ]);
}
```

### Checking for any given groups

To check that a user has **any** of a given set of groups, we will use the `contains()` method:

```php
public function passes(LdapRecord $user, Eloquent $model = null): bool
{
    return $user->groups()->contains([
        'Help Desk', 'Accounting'
    ]);
}
```

> You can also provide a `Model` instance or Distinguished Name into the `contains` method.

This will allow members of **either** the `Help Desk` or `Accounting` group to authenticate.

### Checking for nested group(s) recursively

Nested group checking allows LdapRecord to search recursively if a user is a member of a particular group.

For example, if a user is a member of an `Accounting` group, and this
`Accounting` group is a member of an `Office` group, you can tell
LdapRecord to search recursively for the `Office` group:

```php
public function passes(LdapRecord $user, Eloquent $model = null): bool
{
    return $user->groups()->recursive()->exists('Office');
}
```

Using the above example without the `recursive` call, it will fail to
determine the users group membership, since LdapRecord is only
searching for immediate memberships of the user:

```php
public function passes(LdapRecord $user, Eloquent $model = null): bool
{
    // Only searching immediate group memberships:
    return $user->groups()->exists('Office');
}
```

## Using an Organizational Unit

To use an Organizational Unit which contains your users that you want to
allow sign in to your application, we will leverage LdapRecord [model scopes](/docs/core/v3/models#query-scopes).

In our application, we have an Organizational Unit named `Accounting` with the following Distinguished Name:

```text
ou=Accounting,ou=Users,dc=local,dc=com
```

Let's create a new model scope using the below command:

```bash
php artisan ldap:make:scope OnlyAccountingUsers
```

Now inside the generated scope, we will limit the query to only
return users who are contained inside our `Accounting` OU:

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

        // You can also make this "environment aware" if needed:
        // $query->in(env('LDAP_USER_SCOPE'));
    }
}
```

After modifying the scope, we can now add the scope to our LDAP user model.

If you are using one of the [built-in predefined models](/docs/core/v3/models#predefined-models), you
can add the global scope to the model inside your `AuthServiceProvider::boot()` method:

```php
// app/Providers/AuthServiceProvider.php

use App\Ldap\Scopes\OnlyAccountingUsers;

/**
 * Register any authentication / authorization services.
 *
 * @return void
 */
public function boot()
{
    $this->registerPolicies();

    \LdapRecord\Models\ActiveDirectory\User::addGlobalScope(
        new OnlyAccountingUsers
    );
}
```

If you have created your own LDAP model, add the scope in the inside your models static `boot` method:

```php
<?php

namespace App\Ldap;

use LdapRecord\Models\Model;
use App\Ldap\Scopes\CompanyScope;

class User extends Model
{
    /**
     * The "booting" method of the model.
     *
     * @return void
     */
    protected static function boot()
    {
        parent::boot();

        static::addGlobalScope(new OnlyAccountingUsers);
    }
}
```

Now when you attempt to sign in to your application, only users who are
contained inside the `Accounting` OU will be allowed to authenticate.

## Using Only Manually Imported Users

To enforce only [manually imported LDAP users](/docs/laravel/v3/importing) who exist inside your
database to sign in to your application, you must use an [authentication rule](/docs/laravel/v3/auth/configuration#rules).

LdapRecord-Laravel includes this authentication rule out-of-the-box:

```
LdapRecord\Laravel\Auth\Rules\OnlyImported
```

To use this rule, insert it into the `rules` array into your authentication
provider configuration inside the `config/auth.php` file:

```php
// config/auth.php

'providers' => [
    // ...

    'ldap' => [
        'driver' => 'ldap',
        'model' => LdapRecord\Models\ActiveDirectory\User::class,
        'rules' => [
            LdapRecord\Laravel\Auth\Rules\OnlyImported::class, // <-- Added here.
        ],
    ],
],
```

> Make sure you run `php artisan config:clear` if you are caching your configuration files.

Now when you attempt to sign in to your application, you will only be able to sign in
with a user who has already been imported into your local application's database.
