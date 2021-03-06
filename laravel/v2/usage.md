---
title: Usage
description: LdapRecord-Laravel Usage Guide
---

# Usage

## Models

> This usage documentation is intentionally kept short and sweet to prevent duplication.
> Be sure to review the core [LdapRecord documentation](/docs/core/v2) as it explains
> all of LdapRecord's features in-depth.

Once you've configured your connections in your `config/ldap.php` file,
you're ready to start running queries and operations on your LDAP server.

If you're connecting to an Active Directory or OpenLDAP server, you may utilize the
[built-in models](/docs/core/v2/models#predefined-models) to get up and running right away.

If you would like to create your own models, you can generate one via the `make:ldap-model` command:

```bash
php artisan make:ldap-model User
```

This will create a new LdapRecord model inside of your application in the `app/Ldap` folder.

> If the `Ldap` folder does not exist, it will be created automatically.

By default, the generated model will not have any `$objectClasses` set. LdapRecord requires
this attribute to be set for objects to be created properly in your directory.

If no `$objectClasses` are set, queries will not be scoped to the object you are querying
and you will have to set the models `$objectClasses` manually before creating new objects.

```php
namespace App\Ldap;

use LdapRecord\Models\Model;

class User extends Model
{
    public static $objectClasses = [
        'top',
        'person',
        'organizationalperson',
        'user',
    ];
}
```

> You may want to extend from the `LdapRecord\Models\ActiveDirectory\Entry` class to
> utilize some helper methods that are limited to the use of Active Directory.
> </br></br>
> This can save you time from having to implement functionality manually.

### Searching

To begin querying your model, you can statically call [query methods](/docs/core/v2/searching) off of the model:

```php
$users = User::where('company', '=', 'Acme')->get();
```

When using the above example model for searching your LDAP directory, the following filter will be used:

```text
(&(objectclass=top)(objectclass=person)(objectclass=organizationalperson)(objectclass=user)(company=Acme))
```

### Creating / Updating

To create a new object in your directory, call the `create` method:

```php
$user = User::create([
    'company'   => 'Acme',
    'givenname' => 'Steve',
    'sn'        => 'Bauman',
    'cn'        => 'Steve Bauman',
]);
```

When creating the above example model, the `objectclass` attribute will automatically be
sent with all other attributes you have set for the user creation. This effectively
creates the proper object in your directory.

You may also create a new model instance, set its attributes, and call the `save` method:

```php
$user = new User;

$user->cn = 'Steve Bauman';
$user->givenname = 'Steve';
$user->sn = 'Bauman';
$user->company = 'Acme';

$user->save();
```

Similarly, to update an object, modify a model that was returned from a query and call the `save` method:

```php
$user = User::find('cn=Steve Bauman,dc=local,dc=com');

$user->company = 'Acme';

$user->save();
```

> If you need help understanding user creation and management, take a look at the Active Directory
> [user management tutorial](/docs/laravel/v2/active-directory/users/).

### Scopes

Sometimes you may need to utilize several of the same query filters around your application.
Model scopes are a perfect for this, as you can extract these filters into its own class
and apply it to a model query.

> Be sure to take a peek at the [query scopes](/docs/core/v2/models#query-scopes)
> documentation for a more in-depth look.

To create a new model scope, call the command:

```bash
php artisan make:ldap-scope OnlyAccountants
```

This will create a new model scope in your applications `app\Ldap\Scopes` directory.

```php
namespace App\Ldap\Scopes;

use LdapRecord\Models\Model;
use LdapRecord\Models\Scope;
use LdapRecord\Query\Model\Builder;

class OnlyAccountants implements Scope
{
    public function apply(Builder $query, Model $model)
    {
        $query->where('title', '=', 'Accountant');
    }
}
```

Now, you can either apply this scope globally so the query filter is applied on every
query of your model, or apply it when you need it. Let's walk through both.

To apply your scope globally, override your models protected static `boot`
method, and then call the `addGlobalScope` method:

```php
namespace App\Ldap;

use LdapRecord\Models\Model;
use App\Ldap\Scopes\OnlyAccountants;

class User extends Model
{
    // ...

    protected static function boot()
    {
        parent::boot();

        static::addGlobalScope(new OnlyAccountants);
    }
}
```

> You must pass a **new** instance of your scope into the `addGlobalScope` method, **not** the class name.

Any search queries that are performed on your `User` model will now be properly scoped.

If you wish to apply a scope in certain situations, you may use the `withGlobalScope` method:

```php
$accountants = User::withGlobalScope('accountants', new OnlyAccountants)->get();
```

As you may have noticed above, you must provide a named string for the scope you are passing in.

## Basic Authentication

Sometimes you simply want to know if a users LDAP credentials are valid.
To do this, you must retrieve your LDAP connection from the LdapRecord
[connection container](/docs/core/v2/connections/#container).

To do so, you must call the `getConnection` method on the `Container` and pass in the name of
your connection that appears in your `config/ldap.php` file:

```php
use LdapRecord\Container;

$connection = Container::getConnection('default');
```

To retrieve your `default` connection that you have set in your `ldap.php` configuration
file, call the `getDefaultConnection` method:

```php
use LdapRecord\Container;

$connection = Container::getDefaultConnection();
```

Once you have your connection, call the `auth()->attempt` method with your users
Distinguished Name and their password:

```php
use LdapRecord\Container;

$connection = Container::getConnection('default');

if ($connection->auth()->attempt('cn=user,dc=local,dc=com', 'SuperSecret')) {
    // Credentials are valid!
}
```

If you don't want your user to have to enter in their Distinguished Name, locate the
user in your directory first who is attempting to authenticate, and then pass in
their Distinguished Name:

```php
use LdapRecord\Container;
use LdapRecord\Models\ActiveDirectory\User;

$connection = Container::getConnection('default');

$user = User::findByOrFail('samaccountname', 'sbauman');

if ($connection->auth()->attempt($user->getDn(), 'SuperSecret')) {
    // Credentials are valid!
}
```

If you need to determine why the users authentication is failing (for example, if their password has expired),
you can retrieve the last message that was generated from your LDAP server. This message will usually
contain a code that you can use to determine the cause of failure:

```php
if ($connection->auth()->attempt($user->getDn(), 'SuperSecret')) {
    // Credentials are valid!
} else {
    $message = $connection->getLdapConnection()->getDiagnosticMessage();

    if (strpos($message, '532') !== false) {
        return "Your password has expired.";
    }
}
```
