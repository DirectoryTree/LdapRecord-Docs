---
title: Model Relationships
description: Implementing and utilizing relationships in LdapRecord
---

# Models: Relationships

## Introduction

LDAP objects often contain attributes that reference other LDAP objects in your directory. An
example of this would be the `member` attribute on LDAP groups that contain a list of
distinguished names whom are members of the group.

Using LdapRecord relationships, we can define what models contain references to other objects
and easily retrieve the referenced models to perform operations upon. There are several
relationship types that LdapRecord supports:

| Relationship                            | Type                                                                                  |
| --------------------------------------- | ------------------------------------------------------------------------------------- |
| [Has One](#has-one)                     | Indicates a one-to-one relation, such as a user having one manager                    |
| [Has Many](#has-many)                   | Indicates a one-to-many relation, such as a user having many groups                   |
| [Has Many (Inverse)](#has-many-inverse) | Indicates an inverse one-to-many relation, such as a group having many members        |
| [Has Many In](#has-many-in)             | Indicates a one-to-many relation, but with virtual attributes that cannot be modified |

## Defining Relationships

### Has One

A has one relationship is a basic relationship to work with. An example of a "has one" relationship would be
a `User` having one `manager`. To define this relationship, we place a `manager()` method on our `User`
model, and call the `hasOne()` method and return the result:

```php
<?php

use LdapRecord\Models\Model;

class User extends Model
{
    /**
     * Retrieve the manager of the current user.
     */
    public function manager()
    {
        return $this->hasOne(User::class, 'manager');
    }
}
```

The first argument that is passed into the relation is the name of the related model.
The second is the LDAP attribute on the _current_ user that contains the
relationships distinguished name.

If the relationships attribute you are defining does not contain a distinguished name,
you can alter this and define a _foreign key_ using the third parameter. For example,
if our manager attribute actually contains a `uid`, we can change this so the
related model is retrieved by a UID, instead of a distinguished name:

```php
<?php

use LdapRecord\Models\Model;

class User extends Model
{
    /**
     * Retrieve the manager of the current user.
     */
    public function manager()
    {
        return $this->hasOne(User::class, 'manager', 'uid');
    }
}
```

### Has Many

Defining a has many relationship indicates that the model can be apart of
many of the given model.

For example, a `User` "has many" `groups`:

```php
<?php

use LdapRecord\Models\Model;

class User extends Model
{
    /**
     * Retrieve the groups the user is apart of.
     */
    public function groups()
    {
        return $this->hasMany(Group::class, 'member');
    }
}
```

In the above example, LdapRecord will construct a query to locate all of
the groups that the user is apart of using the users distinguished name.
This users distinguished name will automatically be escaped to
be able to properly locate all of the groups.

For example, this is the query filter that will be used in the search:

```text
(member=cn\3dJohn Doe\2cdc\3dacme\2cdc\3dorg)
```

If you're using an alternate LDAP server or a different attribute to locate
group membership, you may change the relation key. For example, you may
want to use `uniquemember` for this relationship:

```php
/**
 * Retrieve the groups the user is apart of.
 */
public function groups()
{
    return $this->hasMany(Group::class, 'uniquemember');
}
```

You may also define a _foreign key_ in third parameter if the attribute
you are using is not a distinguished name.

### Has Many (Inverse)

Now that we have setup a `User` model that can access of their groups,
lets define a `Group` model to be able to access its members.

Since an LDAP group can contain various types of objects (such as
contacts, users, and other groups), we must pass in an array of
models that are potential members of the group. This allows
the relationship to properly create the models that are
returned from the query results.

> LdapRecord will return plain `Entry` models when
> it cannot locate the correct model in the given array.

```php
<?php

use LdapRecord\Models\Model;

class Group extends Model
{
    /**
     * Retrieve the members of the group.
     */
    public function members()
    {
        return $this->hasMany([
            Group::class, User::class, Contact::class
        ], 'memberof')->using($this, 'member');
    }
}
```

> For brevity, we have not shown the creation of the `Contact` model.

You can see from the above example, we have passed an array of models
that are possible members of the group. The difference of this
definition is the usage of the `using()` method.

Since LDAP does not offer bi-directional relationships, we must add the
`using()` method. This method defines which model and attribute
to use for attaching and detaching related models.

In this case, we pass in `$this` to indicate that the current model
instance (the `Group`) contains the `member` attribute to add and
remove models you pass into the `attach()` and `detach()` methods.

This method is paramount to be able to properly utilize this relationship.

When querying the above relationship, LdapRecord will construct the following filter:

```text
(memberof=cn\3dAccounting\2cdc\3dacme\2cdc\3dorg)
```

### Has Many In

The has many in relationship allows you to retrieve related models from
the given parent models [virtual attribute](https://ldapwiki.com/wiki/Virtual%20Attribute)
such as `memberof`.

> Since this relationship uses virtual attributes, you cannot
> use `attach()` or `detach()` methods. This also means that
> for each entry that is contained in the virtual attribute,
> they will be queried for individually which can be very
> resource intensive depending on the group size.

Lets define a `groups()` relationship that utilizes the `hasManyIn()` method:

```php
<?php

use LdapRecord\Models\Model;

class User extends Model
{
    public function groups()
    {
        return $this->hasManyIn(Group::class, 'memberof');
    }
}
```

#### Important Note for Querying

When using the above relationship from query results, you must ensure
you select the LDAP property you have defined as the _foreign key_
in the relationship. This attribute contains the values needed to
locate the related models.

For example, the following relationship query below will return no results
because we have explicitly requested attributes **excluding** `memberof`:

```php
// Selecting only the 'cn', and 'sn' attributes:
$user = User::select(['cn', 'sn'])->find('cn=John Doe,dc=local,dc=com');

// Returns an empty collection.
$groups = $user->groups()->get();
```

## Returning Only Matching Models

> **Important**: This feature was added in v2.19.0.

When querying relationships on your LdapRecord models, you may receive
plain `LdapRecord\Models\Entry` instances if none of the models you
have provided in the relationship definition match the result's
object classes.

For example, an LDAP group may contain users, as well as other groups. To
explicitly return only users, you may call the method `onlyRelated()` to
filter the underlying query to match only `User` instances:

```php
class Group extends Model
{
    public function users()
    {
        return $this->hasMany(User::class, 'memberof')->onlyRelated();
    }
}
```

## Querying Relationships

LdapRecord relationships also serve as [query builders](/docs/core/v2/searching).
This means you can chain query builder methods onto relationship methods to add
constraints to the relationship query prior to retrieving the results from
your directory.

For example, lets define a `User` model that can be a member of many groups:

```php
<?php

use App\Group;
use LdapRecord\Models\Model;

class User extends Model
{
    /**
     * Retrieve groups that the current user is apart of.
     */
    public function groups()
    {
        return $this->hasMany(Group::class, 'member');
    }
}
```

Now, lets retrieve a user's groups, but only return those groups that have a common name starting with 'Admin':

```php
$user = User::find('cn=John Doe,dc=local,dc=com');

$adminGroups = $user->groups()->whereStartsWith('cn', 'Admin')->get();
```

> By default, querying relations will not include recursive results. More on this below.

### Recursive Queries

To request all of the relationships results, such as nested groups in groups, call
the `recursive()` method, prior to retrieving results via `get()`:

```php
$user = User::find('cn=John Doe,dc=local,dc=com');

$allGroups = $user->groups()->recursive()->get();
```

> Be careful when calling `recursive` on large sets of group memberships.
> If you are not careful, you could run out of memory due to thousands
> of models being returned.

The `recursive` method sets a flag on the LdapRecord relationship indicating
you would like recursive results included (groups of groups).

Recursive results are gathered by first retrieving the groups that the user is
a member of, then retrieving the groups that are members of each resulting
parent group. This means an LDAP search query is executed for each group
that your user is apart of.

Circular group dependencies are rejected automatically to prevent infinite looping.

## Attaching & Detatching Relationships

Using relationships you define, you can easily attach and detach related models from each other.
For example, you may want to attach a `Group` to a `User`, or vice-versa.

### Attaching

Using the above relationship examples, lets walk through attaching a user to a group:

```php
$user = User::find('cn=John Doe,dc=local,dc=com');
$group = Group::find('cn=Accounting,dc=local,dc=com');

// Attaching a group to a user:
$user->groups()->attach($group);

// Attaching a user to a group:
$group->members()->attach($user);
```

You may also use the `attachMany()` method to attach many models at once.

For this example, let's say we have an organizational unit that contains groups all new users must be apart of:

```php
$ou = OrganizationalUnit::find('ou=Groups,dc=local,dc=com');

$groups = Group::in($ou)->get();

$user = User::find('cn=John Doe,ou=Users,dc=local,dc=com');

$user->groups()->attachMany($groups);
```

As you can see above, we took a complex LDAP operation and completed it in just 4 lines of code.

### Detach

Using the above relationship examples, lets walk through detaching a user from a group:

```php
$user = User::find('cn=John Doe,dc=local,dc=com');

// Retrieve the first group that the user is apart of:
$group = $user->groups()->get()->first();

$user->groups()->detach($group);
```

You may also want to detach a user from all groups, if for example they are
leaving the company and you it is apart of your off-boarding process.

You may accomplish this task by using the `detachAll()` method:

```php
$user = User::find('cn=John Doe,ou=Users,dc=local,dc=com');

$user->groups()->detachAll();
```

## Checking Relationship Existence

To check if a model exists inside of a relationship, use the `exists()` relationship method.

> If you're using Active Directory and are simply looking to check if a user is
> inside of a particular group, utilize the `Model::whereMemberOf` method
> that is available on all Active Directory models to locate users
> whom are members of that group.

For example, lets determine if a `User` is a member of a `Group`:

```php
$user = User::find('cn=John Doe,dc=local,dc=com');
$group = Group::find('cn=Accounting,dc=local,dc=com');

if ($user->groups()->exists($group)) {
    // This user is a member of the 'Accounting' group.
}
```

This method can be used on **all** relationship types.

For another example, lets determine if a `User` is a `manager` of another:

```php
$user = User::find('cn=John Doe,dc=local,dc=com');
$manager = User::find('cn=Jane Doe,dc=local,dc=com');

if ($user->manager()->exists($manager)) {
    // Jane Doe is John Doe's manager.
}
```

You can also determine if the model has any groups or members by simply calling `exists()`:

```php
$user = User::find('cn=John Doe,dc=local,dc=com');

if ($user->manager()->exists()) {
    // This user has a manager.
}

if ($user->groups()->exists()) {
    // This user is a member of at least one group.
}
```
