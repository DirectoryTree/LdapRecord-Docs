---
title: Plain Authentication Configuration
description: Configuring the plain LDAP authentication provider
---

# Plain Auth Configuration

## Introduction

To configure a plain LDAP authentication provider, navigate to the `providers` array
inside your `config/auth.php` file, and paste the following `users` provider:

> You will have to remove/alter the default `users` provider, or create your own.

```php
// config/auth.php

'providers' => [
    // ...

    'users' => [
        'driver' => 'ldap',
        'model' => LdapRecord\Models\ActiveDirectory\User::class,
        'rules' => [],
        'scopes' => [],
    ],
],
```

> **Important**: If your application requires more than one LDAP connection, you
> must create a new provider for each connection.
> <br/><br/>
> This new provider must have its own unique `model` class which must use your
> [alternate configured connection](/docs/core/v3/models#connections) name
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

The `model` option must be the class name of your [LdapRecord model](/docs/core/v3/models). This model will be used
for fetching users from your directory.

### Using your own model

If you're using an LDAP directory that is not directly supported by
LdapRecord, you may publish your own model using the below command:

```bash
php artisan ldap:make:model User
```

> The model will be created inside the `app/Ldap` directory.

Once created, insert the following interface and trait onto the model:

**Interface**:

```php
Illuminate\Contracts\Auth\Authenticatable
```

**Trait**:

```php
LdapRecord\Models\Concerns\CanAuthenticate
```

Finally, you must define a `$guidKey` property which will contain the name of
the attribute your LDAP directory uses to store its unique identifier.

> **Important**: Don't forget to also define the models `$objectClasses`.

```php
namespace App\Ldap;

use LdapRecord\Models\Model;
use LdapRecord\Models\Concerns\CanAuthenticate;
use Illuminate\Contracts\Auth\Authenticatable;

class User extends Model implements Authenticatable
{
    use CanAuthenticate;

    public static array $objectClasses = ['...'];

    protected string $guidKey = 'uuid';
}
```

## Rules

The `rules` option must be an array of [authentication rule](#rules) class names.

### Overview

LDAP authentication rules give you the ability to allow or deny users from signing in to your
application using a condition you would like to apply. These rules are executed **after**
a user successfully passes LDAP authentication against your configured server.

Think of them as a final authorization gate before they are allowed in.

> Authentication rules are never executed if a user fails LDAP authentication.

### Creating Rules

Let's create an LDAP rule that only allows members of our domain `Administrators` group.

To create an authentication rule, call the `ldap:make:rule` command:

```bash
php artisan ldap:make:rule OnlyAdministrators
```

A rule will then be created in your applications `app/Ldap/Rules` directory:

```php
<?php

namespace App\Ldap\Rules;

use LdapRecord\Laravel\Auth\Rule;
use LdapRecord\Models\Model as LdapRecord;
use Illuminate\Database\Eloquent\Model as Eloquent;

class OnlyAdministrators implements Rule
{
    /**
     * Check if the rule passes validation.
     */
    public function passes(LdapRecord $user, Eloquent $model = null): bool
    {
        //
    }
}
```

In the authentication rule, a `$user` property will be made available to us.

Now, we will update the `passes` method to check the LDAP users `groups` relationship to see if they are a member:

```php
<?php

namespace App\Ldap\Rules;

use LdapRecord\Laravel\Auth\Rule;
use LdapRecord\Models\Model as LdapRecord;
use LdapRecord\Models\ActiveDirectory\Group;
use Illuminate\Database\Eloquent\Model as Eloquent;

class OnlyAdministrators implements Rule
{
    public function passes(LdapRecord $user, Eloquent $model = null): bool
    {
        $administrators = Group::find('cn=Administrators,dc=local,dc=com');

        return $user->groups()->recursive()->exists($administrators);
    }
}
```

> We call the `recursive` method on the relationship to make sure we load groups of
> groups in case the user is not an immediate member of the `Administrators` group.

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
    ],
],
```

Now when you attempt to log in to your application with an LDAP user that successfully passes
LDAP authentication, they will need to be a member of the `Administrators` group.

> If you are caching your configuration, make sure you re-run `config:cache` to re-cache your modifications.

## Scopes

The `scopes` option must be an array of [LdapRecord scope](/docs/core/v3/model-scopes.md) class names.

### Overview

The `scopes` inserted in this option allow you to apply query scopes to your configured
model, only during authentication. This is option is useful for when you don't want
to add global scopes to your configured model, but would like to scope the query
used to retrieve users during import and authentication into your application.

### Creating Scopes

Let's create an LDAP scope that scopes the authentication query to only return
users that are located inside the `Accounting` Organizational Unit.

To create a new LDAP scope, call the `ldap:make:scope` command:

```bash
php artisan ldap:make:scope Accounting
```

A scope will then be created in your applications `app/Ldap/Scopes` directory:

```php
<?php

namespace App\Ldap\Scopes;

use LdapRecord\Models\Model;
use LdapRecord\Models\Scope;
use LdapRecord\Query\Model\Builder;

class Accounting implements Scope
{
    /**
     * Apply the scope to the given query.
     */
    public function apply(Builder $query, Model $model): void
    {
        // ...
    }
}
```

Now let's update the `apply` method to only return users located inside the `Accounting` Organizational Unit:

> We're using [automatic base DN subsitution](/docs/core/v3/searching#automatic-base-dn-substitution)
> in query below by supplying `{base}` in the `$query->in()` method.

```php
class Accounting implements Scope
{
    /**
     * Apply the scope to the given query.
     */
    public function apply(Builder $query, Model $model): void
    {
        $query->in('ou=Accounting,{base}');
    }
}
```

Once we have our scope defined, we will add it into our authentication provider in the `config/auth.php` file:

```php
'providers' => [
    // ...

    'users' => [
        'driver' => 'ldap',
        'model' => LdapRecord\Models\ActiveDirectory\User::class,
        'rules' => [],
        'scopes' => [
            App\Ldap\Scopes\Accounting::class,
        ],
    ],
],
```

Now when you attempt to log in to your application with an LDAP user, that LDAP 
user must be located inside the `Accounting` Organizational Unit to be able 
to authenticate into your application. 

> If you are caching your configuration, make sure you re-run `config:cache` to re-cache your modifications.
