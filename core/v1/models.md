---
title: Models
description: Creating and using models in LdapRecord
---

# Models: Getting Started

## Introduction

The LdapRecord ORM provides a beautiful and simple ActiveRecord implementation for working with your LDAP server.
Each "Model" represents a type of LDAP object that resides in your directory.

Models allow you query your directory as well as create, update, and delete records.

Before getting started, ensure you've added at least one connection to the [container](/docs/core/v1/connections#container).

By default, there are models included with LdapRecord for popular LDAP directories (namely Active Directory &
OpenLDAP) so you can get up and running as fast as possible. More on this [below](#predefined-models).

### Defining Models

To get started, you must create a new class that represents the LDAP object you would like to query.

For example, let's create a model that represents Active Directory users:

```php
<?php

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

As you can see above, we must add a public static property that contains the object classes of the LDAP record.

These object classes are used to locate the proper objects in your LDAP directory.

> If you do not provide any object classes, global directory searches will be performed when retrieving models.

### Predefined Models

LdapRecord comes with many predefined models that allow you to get started right away.

You may extend these built-in models and add your own methods & functionality, as well as override built-in functionality.

#### Entry Model

Use the `LdapRecord\Models\Entry` model for retrieving all objects from your directory - regardless of type.

#### Active Directory Models

Each below model references a type of object in Active Directory.

| Model                                                        |
| ------------------------------------------------------------ |
| `LdapRecord\Models\ActiveDirectory\Entry`                    |
| `LdapRecord\Models\ActiveDirectory\User`                     |
| `LdapRecord\Models\ActiveDirectory\Group`                    |
| `LdapRecord\Models\ActiveDirectory\Computer`                 |
| `LdapRecord\Models\ActiveDirectory\Contact`                  |
| `LdapRecord\Models\ActiveDirectory\Container`                |
| `LdapRecord\Models\ActiveDirectory\OrganizationalUnit`       |
| `LdapRecord\Models\ActiveDirectory\Printer`                  |
| `LdapRecord\Models\ActiveDirectory\ForeignSecurityPrincipal` |

#### OpenLDAP Models

| Model                                           |
| ----------------------------------------------- |
| `LdapRecord\Models\OpenLdap\Entry`              |
| `LdapRecord\Models\OpenLdap\User`               |
| `LdapRecord\Models\OpenLdap\Group`              |
| `LdapRecord\Models\OpenLdap\OrganizationalUnit` |

#### FreeIPA Models

| Model                             |
| --------------------------------- |
| `LdapRecord\Models\FreeIPA\Entry` |
| `LdapRecord\Models\FreeIPA\User`  |
| `LdapRecord\Models\FreeIPA\Group` |

> Don't see a model for the LDAP server you're using? [Create a pull request!](https://github.com/DirectoryTree/LdapRecord/pulls)

### Connections

By default, all models you create will try to use your `default` LDAP connection that resides in the connection
[container](/docs/core/v1/connections#container). To set your model to use an alternate connection,
define a `$connection` property equal to the name of your other connection.

```php
<?php

use LdapRecord\Models\Model;

class User extends Model
{
    protected $connection = 'domain-b';
}
```

### Distinguished Names

To get an objects full distinguished name call the `getDn` method:

```php
$user = User::find('cn=user,dc=local,dc=com');

// Returns 'cn=user,dc=local,dc=com'
$user->getDn();
```

To get an objects relative distinguished name, call the `getRdn` method:

```php
$user = User::find('cn=user,dc=local,dc=com');

// Returns 'cn=user'
$user->getRdn();
```

To get an objects parent distinguished name, call the `getParentDn` method:

```php
$user = User::find('cn=user,dc=local,dc=com');

// Returns 'dc=local,dc=com'
$user->getParentDn();
```

To get an objects name, call the `getName` method:

```php
$user = User::find('cn=user,dc=local,dc=com');

// Returns 'user'
$user->getName();
```

### Object GUIDs

To retrieve a models Object GUID (globally unique identifier) call the `getConvertedGuid` method.

This method will return the string variant of your models GUID. Some LDAP directories
(namely Active Directory) use hexadecimal byte arrays to store these, so conversion
is necessary.

```php
$user = User::find('cn=user,dc=local,dc=com');

$user->getConvertedGuid();
```

To retrieve the raw GUID value, use the `getObjectGuid` method.

By default, LdapRecord models will use the `objectguid` attribute in the
above methods. If your directory stores GUIDs in a different attribute,
define a `$guidKey` attribute inside of your model:

```php
class User extends Model
{
    protected $guidKey = 'entryuuid';
}
```

### Default Attribute Values

If you would like to define the default values for some of your model's attributes,
you may define an `$attributes` property on your model. This helps you to assign
static default values when creating objects in your directory:

> Due to LDAP's multi-valued nature, each attribute value you define **must** be
> an array, regardless if it is single-valued or or multi-valued.

```php
class User extends Model
{
    protected $attributes = [
        'company' => ['Acme'],
        'description' => ['User Account'],
        'manager' => ['cn=John Doe,dc=acme,dc=org']
    ];
}
```

## Retrieving Models

Once you've created an LdapRecord model you're ready to start retrieving data from your directory.
If you've used Laravel's [Eloquent ORM](https://laravel.com/docs/eloquent), you'll feel right at home.

You can think of a model as a powerful query builder allowing you to query your directory for objects
fluently and easily.

```php
<?php

$users = User::get();

foreach ($users as $user) {
    $user->getFirstAttribute('cn');
}
```

### Adding Constraints

Each model serves as a query builder for the object classes you've defined inside.
You can add constraints to your queries and then call `get()` to retrieve the
results.

```php
<?php

$users = User::whereStartsWith('cn', 'John')
            ->whereEndsWith('sn', 'Doe')
            ->limit(10)
            ->get();
```

> Since models are query builders, it's a good idea to review the
> [query builder](/docs/core/v1/searching) methods so you can utilize
> them to their full potential.

### Model Constraints

Models come with some built in constraint methods that you may find useful.

> The below constraints will only retrieve the models that are equal
> to the type you have retrieved. For example, retrieving the descendants
> of an organizational unit will only return organizational units that
> are direct descendants. <br/><br/>
>
> If you would like to avoid this, use the default `LdapRecord\Models\Entry`
> model, which provides no `objectclass` constraints on queries.

#### Ancestors

To retrieve the direct ancestors of a model, call the `ancestors()` constraint on a retrieved model:

```php
$ou = OrganizationalUnit::find('ou=Accountants,ou=Users,dc=local,dc=com');

$ancestors = $ou->ancestors()->get();
```

The above example will execute a `listing` on your LDAP directory in
the distinguished name `dc=local,dc=com`. This effectively pulls
the ancestors of the model.

#### Siblings

To retrieve the siblings of a model, call the `siblings()` constraint on a retrieved model:

```php
$ou = OrganizationalUnit::find('ou=Accountants,ou=Users,dc=local,dc=com');

$siblings = $ou->siblings()->get();
```

The above example will execute a `listing` on your LDAP directory in
the distinguished name `ou=Users,dc=local,dc=com`. This effectively
pulls the siblings of the model. The current model will also be
included in the resulting collection.

#### Descendants

To retrieve the descendants of a model, call the `descendants()` constraint on a retrieved model:

```php
$ou = OrganizationalUnit::find('ou=Accountants,ou=Users,dc=local,dc=com');

$descendants = $ou->descendants()->get();
```

The above example will execute a `listing` on your LDAP directory in
the distinguished name `ou=Accountants,ou=Users,dc=local,dc=com`. This effectively
pulls the descendants of the model.

#### Refreshing Models

To re-retrieve a new model from your directory, call the `fresh()` method.
Doing so will not affect the existing instance you already have:

```php
$user = User::where('cn', '=', 'jdoe')->first();

$freshUser = $user->fresh();
```

If you would like to re-retrieve the existing model, call the `synchronize()` method.
This will re-retrieve the models attributes from the directory:

```php
$user = User::where('cn', '=', 'jdoe')->first();

$user->synchronize();
```

### Collections

When you query your models, returned results will be contained inside of a
`LdapRecord\Query\Collection`. The `Collection` class directly extends
Laravel's collection. Be sure to check out its [documentation](https://laravel.com/docs/collections)
for all of the available helpful methods.

```php
<?php

$users = User::get();

$usersWithEmail = $users->filter(function (User $user) {
    return $user->hasAttribute('mail');
});
```

## Retrieving Single Models

If you would like to retrieve a single model from your directory, you can utilize
a variety of methods. Here is a list and usage of each:

| Method                                    |
| ----------------------------------------- |
| `first()`                                 |
| `find($distinguishedName)`                |
| `findBy($attributeName, $attributeValue)` |
| `findByAnr($attributeValue)`              |
| `findByGuid($objectGuid)`                 |

```php
// Retrieve the first model of a global LDAP search...
$user = User::first();

// Retrieve a model by its distinguished name...
$user = User::find('cn=John Doe,dc=acme,dc=org');

// Retrieve the first model that matches the attribute...
$user = User::findBy('cn', 'John Doe');

// Retrieve the first model that matches an array of ANR attributes...
$user = User::findByAnr('John Doe');

// Retrieve a model by its object guid...
$user = User::findByGuid('bf9679e7-0de6-11d0-a285-00aa003049e2');
```

#### Not Found Exceptions

Occasionally you may want to throw an exception if a specific record you're looking
for cannot be found on your directory. You can substitute the above methods
with `OrFail()` variants:

| Method                                          |
| ----------------------------------------------- |
| `firstOrFail()`                                 |
| `findOrFail($distinguishedName)`                |
| `findByOrFail($attributeName, $attributeValue)` |
| `findByAnrOrFail($attributeValue)`              |
| `findByGuidOrFail($objectGuid)`                 |

```php
try {
    // Retrieve the first model of a global LDAP search or fail...
    $user = User::firstOrFail();

    // Retrieve a model by its distinguished name or fail...
    $user = User::findOrFail('cn=John Doe,dc=acme,dc=org');

    // Retrieve the first model that matches the attribute or fail...
    $user = User::findByOrFail('cn', 'John Doe');

    // Retrieve the first model that matches an array of ANR attributes or fail...
    $user = User::findByAnrOrFail('John Doe');

    // Retrieve a model by its object guid or fail...
    $user = User::findByGuidOrFail('bf9679e7-0de6-11d0-a285-00aa003049e2');
} catch (\LdapRecord\Models\ModelNotFoundException $e) {
    // One of the models could not be located...
}
```

## Creating & Updating Models

### Creating

Before we begin it is paramount to know that LDAP objects require a Distinguished Name to be
created successfully in your LDAP directory. LdapRecord will always attempt to generate a
Distinguished Name for models that do not have one upon `save`. In addition, some LDAP
objects require **more** attributes to be set for successful creation.

For example, to create a `User` object in Active Directory, the `cn` (Common Name) attribute is required.
If you do not set this attribute, an exception will be thrown upon saving your LDAP model. For another
example, `OrganizationlUnit`'s must have the `ou` attribute set.

LdapRecord cannot validate this for you as LDAP objects differ widely in their attribute requirements.

To create a new record in your directory, create a new model instance and call the `save()` method.
Upon calling `save()`, if no Distinguished Name is set on a new model, one will be generated
based on your configured `base_dn` that you have set inside your connections configuration:

```php
$conn = new Connection([
    // ...
    'base_dn' => 'dc=local,dc=com',
]);

$conn->connect();

$user = new User();

$user->cn = 'John Doe';

// User will be saved with the DN: 'cn=John Doe,dc=local,dc=com
$user->save();
```

#### Dynamic Distinguished Name Generation

LdapRecord generates a models distinguished name via the model method `getCreatableRdn`.
This method is responsible for generating the "Relative Distinguished Name" which is
the true name of the object inside of your LDAP directory that does not include
your base Distinguished Name.

Since _most_ LDAP objects require a Common Name (`cn`) this is defaulted to:

```php
/**
 * Get a creatable RDN for the model.
 *
 * @return string
 */
public function getCreatableRdn()
{
    $name = $this->escape($this->getFirstAttribute('cn'))->dn();

    return "cn=$name";
}
```

> As you can see above, the attribute is escaped before being passed into the RDN string.
> **You must do this**, otherwise if commas or other reserved characters are inside the
> attribute you are using, it will generate a malformed distinguished name.

You may override this method to allow your models Distinguished Name's to be dynamically generated
rather than creating them yourself manually. For example, here is how we would set the Relative
Distinguished Name (RDN) for an Active Directory `OrganizationalUnit` model:

```php
public function getCreatableRdn()
{
    $name = $this->escape($this->getFirstAttribute('ou'))->dn();

    return "ou=$name";
}
```

This then gets prepended onto your connections configured `base_dn`, for a resulting "Full" Distinguished Name:

```text
ou=MyOrganizationalUnitName,dc=local,dc=com
```

You may set the base DN of where you would like the object to be created inside by
using the `inside()` method, rather than your `base_dn` from your configuration:

```php
$user = new User(['cn' => 'John Doe']);

$user->inside('ou=Users,dc=local,dc=com');

$user->save();
```

You may also pass in an LdapRecord `Model` instance. This is convenient so you
know the container / organizational unit distinguished name is valid:

```php
$ou = OrganizationalUnit::findOrFail('ou=Users,dc=acme,dc=org');

$user = new User(['cn' => 'John Doe']);

$user->inside($ou)->save();
```

The above examples will save the user inside the `Users` OU resulting in the full distinguished name:

```text
cn=John Doe,ou=Users,dc=acme,dc=org
```

#### Setting A Distinguished Name

To set the models distinguished name, call the `setDn()` method on your model and populate it
with any organization unit or container that you would like it to be created inside:

```php
$user = new User();

$user->cn = 'John Doe';

$user->setDn('cn=John Doe,ou=Users,dc=acme,dc=org');

$user->save();
```

### Updating

Updating models is as easy as creating them. When you have a model returned from a query,
set its attributes as you would for creating and call the `save()` method:

```php
$user = User::first();

$user->company = 'My Company';
$user->samaccountname = 'jdoe';
$user->department = 'Accounting';
$user->displayname = 'Johnathan Doe';

$user->save();
```

### Moving

To move existing models into Organizational Units or Containers, call the `move()` method:

> When moving a model is successful, the users distinguished name will be
> automatically updated to reflect its new location in your directory,
> so you may continue to run operations on it during the same request.

```php
$user = User::find('cn=Steve Bauman,dc=local,dc=com');

$ou = OrganizationalUnit::find('ou=Office Users,dc=local,dc=com');

$user->move($ou);

// Displays 'cn=Steve Bauman,ou=Office Users,dc=local,d=com'
echo $user->getDn();
```

### Renaming

To rename existing models, call the `rename()` method and supply the new objects RDN (relative distinguished name):

> When renaming is successful, the users distinguished name is automatically
> updated to reflect its new name in the directory, so you may run further
> operations on it during the same request.

```php
$user = User::find('cn=John Doe,dc=local,dc=com');

$user->rename('cn=Jane Doe');

// Displays 'cn=Jane Doe,dc=local,dc=com'
echo $user->getDn();
```

## Restoring Deleted Models

> **Important**: This feature is only possible when connecting to an Active Directory server.

To restore a deleted object, we must first query the directory
for deleted objects by using the `whereDeleted` method:

```php
use LdapRecord\LdapRecordException;
use LdapRecord\Models\ActiveDirectory\User;

$user = User::whereDeleted()
            ->where('mail', '=', 'sbauman@local.com')
            ->first();

try {
    $user->restore();

    // Successfully restored user.
} catch (LdapRecordException $ex) {
    // Failed restoring user.
}
```

If you're including deleted results in your queries using the `withDeleted` clause,
you can call the `isDeleted` method to check if an object has been deleted:

```php
$users = User::withDeleted()->get();

foreach ($users as $user) {
    if ($user->isDeleted()) {
        $user->restore();
    }
}
```

If you call `restore` on a non-deleted object, it will simply return `false`:

```php
$user = User::where('cn', '=', 'Steve Bauman')->first();

$result = $user->restore();

// Displays bool(false)
var_dump($result);
```

## Attributes

### Methods

There are several built-in methods on models you may like to utilize:

`Model::getAttributes()`

The `getAttributes` method returns all of the values on the model:

```php
$user = User::first();

$attributes = $user->getAttributes();

foreach ($attributes as $name => $values) {
    //
}
```

> In the above example, `$values` will always be an array.

`Model::getAttribute($name)`

The `getAttribute` method returns all of the values inside the given key. This will return an `array` if the attribute exists:

```php
$group = Group::first();

$members = $group->getAttribute('member');

if ($members) {
    foreach ($members as $member) {
        echo $member;
    }
}
```

`Model::getFirstAttribute($name)`

The `getFirstAttribute` method returns the first value of the given key. This will always return `null` or `string`:

```php
$group = Group::first();

$firstMember = $group->getFirstAttribute('member');
```

`Model::hasAttribute()`

The `hasAttribute` method determines whether the model contains the key in the models attributes:

```php
$user = User::first();

if ($user->hasAttribute('company')) {
    //
}
```

`Model::addAttributeValue($name, $value)`

To add a value to an attribute without clearing it, use the `addAttributeValue` method:

```php
$user = User::first();

$user->addAttributeValue('proxyaddresses', 'SMTP:sbauman@local.com');
```

`Model::countAttributes()`

The `countAttributes` method returns the number of attributes the model contains:

```php
$user = User::first();

echo $user->countAttributes();
```

### Array Conversion

Attributes you retrieve from an LdapRecord model will **always**
return and array. This is due to LDAP's multi-valued nature.

For example, if you would like to retrieve the users `mail`
attribute, you must request the first key from it:

```php
$user = User::find('cn=John Doe,dc=acme,dc=org');

// Get the users email address.
echo $user->mail[0];
```

However, the above will cause an exception if the attribute does not exist.

To work around this, we can use the `getFirstAttribute()` method:

```php
$user = User::find('cn=John Doe,dc=acme,dc=org');

// Get the users email address.
echo $user->getFirstAttribute('mail');
```

When setting attributes on models, they will automatically
be converted to an array for you if you do not provide one.

```php
$user = User::find('cn=John Doe,dc=acme,dc=org');

// Both approaches will set the attribute identically:
$user->mail = 'jdoe@acme.org';
$user->mail = ['jdoe@acme.org'];
```

Similarly, you can use the `setFirstAttribute()` method to
set the attributes first value in its array, even if
it does not currently exist on the model:

```php
$user = User::find('cn=John Doe,dc=acme,dc=org');

// Set the users email address.
$user->setFirstAttribute('mail', 'jdoe@acme.org');
```

### Determining Attribute Existence

To check if a model has an attribute, you can use the `hasAttribute()` method:

```php
if ($user->hasAttribute('mail')) {
    // This user has an email address.
}
```

As with all other attribute methods, this check is **case-insensitive**.
You may pass any type of casing of the attribute you are looking for:

```php
// Both will return 'true':
$user->hasAttribute('samaccountname');
$user->hasAttribute('sAMAccountname');
```

### Casing & Hyphens

#### Attribute Casing

LdapRecord automatically normalizes all attribute keys to lowercase. This
means when setting or getting model attributes, you can use alternate
casing and the same attribute will be set or retrieved.

This is extremely handy so you do not have to look up the casing of
each attribute every time you want to set or retrieve one. This
also means you can use your own attribute convention:

```php
$user = new User();

// Each will set the same attribute:
$user->samaccountname = 'John Doe';
$user->sAMAccountName = 'John Doe';
$user->samAccountName = 'John Doe';
```

#### Attribute Hyphens

Since LDAP does not support underscores in LDAP attributes but does
support using hyphens, anytime you would like to set an attribute
that contains a hypen, set it using an underscore instead.
LdapRecord will automatically convert the underscore
to a hyphen dynamically:

```php
$user = new User();

$user->some_attribute = 'Value';

// Returns 'Value'
echo $user->some_attribute[0];
echo $user->getAttribute('some-attribute')[0];
```

Similarly, when retrieving attributes that contain a hyphen, use
an underscore instead:

```php
$user = User::find('cn=John Doe,dc=acme,dc=org');

// Both will act identically:
echo $user->apple_user_homeurl[0];
echo $user->getAttribute('apple-user-homeurl')[0];
```

## Deleting Models

To delete a record from your directory, call the `delete()` method on a model you have retrieved:

```php
<?php

$user = User::first();

$user->delete();
```

> The account you have configured to bind to your LDAP server must have permission to delete the record
> you have retrieved. If it does not, you will receive an exception upon deletion.

#### Deleting Models By Distinguished Name

In the example above, we are retrieving the record from the directory prior to deletion. However,
if you'd like to simply delete a model by its distinguished name, call the `destroy()` method.
The _number_ of deleted models will be returned from this method:

```php
<?php

// Deleting single object...
$deleted = User::destroy('cn=John Doe,dc=acme,dc=org');

// Deleting multiple objects...
$deleted = User::destroy([
    'cn=John Doe,dc=acme,dc=org',
    'cn=Jane Doe,dc=acme,dc=org',
]);
```

> You may also pass in `true` into the second parameter to recursively delete
> leaf entries if a record is located by the distinguished name you have given.

#### Recursive Deleting

Sometimes you will be working with containers or organizational units that contain nested records
inside of them. Calling `delete()` on these records will generate an exception without first
deleting the records inside. If you would like to delete all records contained inside of
another model, pass in `true` in the first parameter of the `delete()` method:

```php
<?php

$ou = OrganizationalUnit::find('ou=Users,dc=acme,dc=org');

$ou->delete($recursive = true);
```

## Comparing Models

If you ever need to compare to models to see if they are the same, call the the `is()` method.
This method will determine if the models have the same distinguished name and connection:

```php
if ($user->is($anotherUser)) {
    //
}
```

To see if a model is contained inside an organizational unit or another type of
container, call the `isDescendantOf()` method:

```php
$ou = OrganizationalUnit::find('ou=User Accounts,dc=acme,dc=org');
$user = User::find('cn=John Doe,ou=User Accounts,dc=acme,dc=org');

if ($user->isDescendantOf($ou)) {
    // This user is contained inside this organizational unit.
}
```

You may also want to know whether a model is an ancestor of another. To
do so, call the `isAncestorOf()` method:

```php
$user = User::find('cn=John Doe,ou=User Accounts,dc=acme,dc=org');
$ou = OrganizationalUnit::find('ou=User Accounts,dc=acme,dc=org');

if ($ou->isAncestorOf($user)) {
    // This OU is an ancestor of this user.
}
```

> Calling `isDescendantOf()` or `isAncestorOf()` performs recursive checks. If
> a model is contained in a nested OU / container of the one you are checking,
> the methods will return `true`.

```php
$ou = OrganizationalUnit::find('ou=User Accounts,dc=acme,dc=org');
$user = User::find('cn=John Doe,ou=Accounting,ou=User Accounts,dc=acme,dc=org');

// This will return true:
if ($user->isDescendantOf($ou)) {
    //
}

// This will return true:
if ($ou->isAncestorOf($user)) {
    //
}
```

To perform non-recursive checks, such as checking if a model is a
**direct** child of another model, call the `isChildOf` method:

```php
$ou = OrganizationalUnit::find('ou=User Accounts,dc=acme,dc=org');
$user = User::find('cn=John Doe,ou=User Accounts,dc=acme,dc=org');

if ($user->isChildOf($ou)) {
    //
}
```

To perform the opposite, such as checking if a model is a parent of
another, call the `isParentOf` method:

```php
$officeOu = OrganizationalUnit::find('ou=Office,ou=User Accounts,dc=acme,dc=org');
$userAccountsOu = OrganizationalUnit::find('ou=User Accounts,dc=acme,dc=org');

if ($userAccountsOu->isParentOf($officeOu)) {
    //
}
```

## Events

LdapRecord models fire several different [event](/docs/core/v1/events) during the creation,
updating and deletion. Here is a list of all the events you can listen for:

| Event                               |
| ----------------------------------- |
| `LdapRecord\Models\Events\Creating` |
| `LdapRecord\Models\Events\Created`  |
| `LdapRecord\Models\Events\Updating` |
| `LdapRecord\Models\Events\Updated`  |
| `LdapRecord\Models\Events\Saving`   |
| `LdapRecord\Models\Events\Saved`    |
| `LdapRecord\Models\Events\Deleting` |
| `LdapRecord\Models\Events\Deleted`  |

To listen for these events, call the `getEventDispatcher()` on the `LdapRecord\Container`
to retrieve the dispatcher, then call `listen()` on the returned dispatcher:

```php
<?php

$dispatcher = \LdapRecord\Container::getEventDispatcher();

$dispatcher->listen(\LdapRecord\Models\Events\Creating::class, function ($event) {
    $model = $event->getModel();
});
```

> You will want to setup any listeners prior to making changes to models,
> otherwise your listener will not be executed due to them not existing yet.

## Serialization

All model instances can be converted to an array for JSON serialization. To serialize
a model instance, simply pass the model into `json_encode()`. This calls
`jsonSerialize()` on the model to retrieve is serializable data:

```php
<?php

$user = User::first();

echo json_encode($user);
```

### Hiding Attributes

You may want to exclude certain attributes from being included in the
serialization of your model, such as `userPassword` for OpenLDAP.

To do this, add a `$hidden` property to your model:

```php
use LdapRecord\Models\Model;

class User extends Model
{
    protected $hidden = ['userPassword'];
}
```

Now when you `json_encode($model)`, all attributes will be included **except** the `userPassword` attribute.

If you'd prefer a white-list of attributes, you can add a `$visible` property instead,
which will ensure only the attributes specified will be included in serialization:

```php
use LdapRecord\Models\Model;

class User extends Model
{
    protected $visible = ['cn', 'mail', 'sn'];
}
```

#### Converting Attributes to JSON

Depending on the type of LDAP directory and model you are working with, you may need
to convert some attributes to a string before it can be properly serialized.
For example, if you your model is from Active Directory, you will need to
convert the `objectguid` property to a string since it is in binary,
otherwise `json_encode()` will throw an exception.

This can be done by adding a `convertAttributesForJson()` method to your model:

> By default, the `objectguid` and `objectsid` attributes are
> converted for you when using the built-in Active Directory models.

```php
<?php

use LdapRecord\Models\Model;

class User extends Model
{
    protected function convertAttributesForJson(array $attributes = [])
    {
        if ($this->hasAttribute('objectguid')) {
            // If the model has a GUID set, we need to convert it due to it being in
            // binary. Otherwise we will receive a JSON serialization exception.
            return array_replace($attributes, [
                'objectguid' => [$this->getConvertedGuid()]
            ]);
        }

        return $attributes;
    }
}
```
