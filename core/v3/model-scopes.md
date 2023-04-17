---
title: Models
description: Creating and using model scopes in LdapRecord
---

# Models: Scopes

## Introduction

Model "scoping" allows you to define functions or classes that dynamically add
filters to a model query. There are two types of scopes you may add to
models, and there are benefits and drawbacks to each type:

**Local Scopes**:

- Applied conditionally on model queries, being called explicitly
- Can accept parameters

**Global Scopes**:

- Applied globally on model queries
- Cannot accept parameters

### Local Query Scopes

#### Creating a local scope

Local scopes allow you to add constraints to LDAP queries created from models dynamically.

To define a local scope, create a new `public` method, prefix it with `scope`,
followed by the name you would like to call the scope by.

Local scopes must also accept the `LdapRecord\Query\Model\Builder` as the first parameter.

For this example, let's create a local model scope that will return Active Directory locked out users:

```php
use LdapRecord\Models\Model;
use LdapRecord\Query\Model\Builder;

class User extends Model
{
    // ...

    /**
     * Apply the scope to the query.
     *
     * @param Builder $builder
     *
     * @return Builder
     */
    public function scopeLockedOut(Builder $query)
    {
        return $query->where('lockouttime', '>', 1);
    }
}
```

#### Using a local scope

Now that we have defined a local scope inside of our model, we can call it like so:

```php
$usersLockedOut = User::lockedOut()->get();
```

Local scopes may also be chained, and accept parameters. This allows you further narrowing down your search results.

Let's add another scope to our example model that will only return users of a particular company:

```php
// User.php

public function scopeLockedOut(Builder $query)
{
    return $query->where('lockouttime', '>', 1);
}

public function scopeCompany(Builder $query, $companyName)
{
    return $query->where('company', '=', $companyName);
}
```

Now we can use both of these scopes in succession:

```php
$users = User::company('acme')->lockedOut()->get();
```

Local scopes are very powerful, allowing you to generate readable, understandable queries!

### Global Query Scopes

Global scopes allow you to add constraints to all LDAP queries that are created on a particular model.
Writing a query scope allows you to be certain that a particular filter is always applied, rather
than adding constraints every time you query the model.

#### Creating a global scope

To create a global query scope, create a class in your application that implements
the `LdapRecord\Models\Scope` interface. This interface will require you to add
an `apply` method. The `apply` method accepts the query `Builder` in the first
parameter, and the `Model` in second parameter.

For an example, let's say our application must only retrieve user accounts whom
are employees of a particular company. We will create a file in our application
in the directory `app/Ldap/Scopes` with the file name `CompanyScope`:

```php
<?php

namespace App\Ldap\Scopes;

use LdapRecord\Models\Scope;
use LdapRecord\Models\Model;
use LdapRecord\Query\Model\Builder;

class CompanyScope implements Scope
{
    /**
     * Apply the scope to the query.
     *
     * @param Builder $builder
     * @param Model   $model
     */
    public function apply(Builder $builder, Model $model)
    {
        $builder->where('company', '=', 'Acme Company');
    }
}
```

> If you are looking to select additional attributes in your scope
> using the `select` method, use `addSelect` instead so your
> query selects are not overwritten.

#### Apply the global scope

Now that we've written our global scope, we can apply it to our users model.

We will do this by adding an override to the models `boot` method and using the `addGlobalScope` method:

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

        static::addGlobalScope(new CompanyScope);
    }
}
```

After adding the scope, queries ran on the `User` model will always add the filter:

```text
(company=\41\63\6d\65\20\43\6f\6d\70\61\6e\79)
```

Which your LDAP server will read as:

```text
(company=Acme Company)
```

This is due to all values being automatically escaped using the LdapRecord query builder.

#### Anonymous global scopes

Instead of creating a class scope, you may also define global scopes using Closures.
This is useful for simple scopes that do not warrant a separate class:

```php
<?php

namespace App\Ldap;

use LdapRecord\Models\Model;
use LdapRecord\Query\Model\Builder;

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

        static::addGlobalScope('manager', function (Builder $builder) {
            $builder->where('manager', '=', 'cn=John Doe,dc=local,dc=com');
        });
    }
}
```

#### Removing Global Scopes

If you would like to remove a global scope for a particular query, you may use
the `withoutGlobalScope` method. The method accepts the class name of the
global scope as its argument:

```php
User::withoutGlobalScope(CompanyScope::class)->get();
```

Or, if you've defined a global scope using a Closure:

```php
User::withoutGlobalScope('manager')->get();
```

If you would like to remove several or even all of the global
scopes, you may use the `withoutGlobalScopes` method:

```php
// Remove all of the global scopes:
User::withoutGlobalScopes()->get();

// Remove some of the global scopes:
User::withoutGlobalScopes([
    CompanyScope::class, 'manager'
])->get();
```
