---
title: Group Management (Active Directory)
description: Managing groups with LdapRecord.
---

# Group Management (Active Directory)

## Creation

To create a new Active Directory group, only a common name is required (`cn`):

```php
<?php

use LdapRecord\Models\ActiveDirectory\Group;

$group = Group::create(['cn' => 'Accounting']);
```

To create a group inside of a particular Organizational Unit (OU), call the `inside()` method:

```php
$group = (new Group)->inside('ou=Office Groups,dc=local,dc=com');

$group->cn = 'Accounting';

$group->save();
```

## Members

When you create or locate a group on your directory, the `members` relationship
is available to you on the model instance.

### Getting Members

To get the immediate members of a group on your directory call the `members` relationship, and then `get()`:

```php
$group = Group::find('cn=Accounting,dc=local,dc=com');

$members = $group->members()->get();
```

> When getting members, a collection of various models may be returned, such as:
>
> - `LdapRecord\Models\ActiveDirectory\User`
> - `LdapRecord\Models\ActiveDirectory\Group`
> - `LdapRecord\Models\ActiveDirectory\Contact`
>
> To use different models, override the `members` relationship method.

#### Getting Members Recursively

Very often we use groups that are apart of other groups, that include members.

To retrieve these nested members, call the `recursive()` method, prior to `get()`:

```php
$group = Group::find('cn=Accounting,dc=local,dc=com');

$allMembers = $group->members()->recursive()->get();
```

### Adding Members

In Active Directory, valid group members are other groups, users and contacts.

To add members to a group, call the `members` relationship, and then the `attach()` method:

> You must provide a `Model` instance into the `attach()` method.

```php
$group = Group::find('cn=Accounting,dc=local,dc=com');

$user = User::find('cn=John Doe,dc=local,dc=com');

$group->members()->attach($user);
```

If the model was successfully added, or the model is already a member of the group,
the `attach()` method will return the model instance you passed into it:

```php
$office = $accounting->members()->attach(
    Group::create(['cn' => 'Office'])
);

// Displays 'cn=Office,dc=local,dc=com'
echo $office->getDn();
```

#### Adding Multiple Members

To add multiple members at once, provide an array of models to the `attachMany()` method:

```php
$accounting = Group::find('cn=Accounting,dc=local,dc=com');

$accountants = User::in('ou=Accountants,ou=Users,dc=local,dc=com')->get();

$accounting->members()->attachMany($accountants);
```

### Removing Members

To remove members on a group, call the `members` relationship, and then `detach()`:

> You must provide a `Model` instance into the `detach()` method.

```php
$group = Group::find('cn=Accounting,dc=local,dc=com');

$user = $group->members()->where('cn', '=', 'Steve Bauman')->first();

$group->members()->detach($user);
```

#### Removing Multiple Members

To remove multiple members at once, provide an array of models to the `detachMany()` method:

```php
$group = Group::find('cn=Accounting,dc=local,dc=com');

$members = $group->members()
                 ->where('department', 'contains', 'Office')
                 ->get();

$group->members()->detachMany($members);
```

#### Removing All Members

To remove **all** immediate members from the group, call the `detachAll()` method:

> A collection of all removed members will be returned.

```php
$group = Group::find('cn=Accounting,dc=local,dc=com');

$removed = $group->members()->detachAll();

foreach ($removed as $member) {
    echo "Removed: " . $member->getDn();
}
```

## Groups

When you create or locate a group on your directory, the `groups` relationship
is available to you on the model instance.

### Getting Groups

To get the immediate groups that a particular group is apart of on
your directory call the `groups` relationship, and then `get()`:

```php
$group = Group::find('cn=Accounting,dc=local,dc=com');

$groups = $group->groups()->get();
```

#### Getting Groups Recursively

When you have a group that is apart of many parent groups in a hierarchy, you may need to retrieve these.

Call the `recursive()` method, prior to `get()` to retrieve them:

```php
$group = Group::find('cn=Accounting,dc=local,dc=com');

$allGroups = $group->groups()->recursive()->get();
```

### Adding Groups

To add groups to a particular group, call the `attach()` method on the `groups` relation:

> You must provide a `Model` instance into the `attach()` method.

```php
$accounting = Group::find('cn=Accounting,dc=local,dc=com');

$office = Group::find('cn=Office,dc=local,dc=com');

$accounting->groups()->attach($office);
```

#### Adding Multiple Groups

To add multiple groups at once, provide an array of models to the `attachMany()` method:

```php
$accounting = Group::find('cn=Accounting,dc=local,dc=com');

$officeGroups = Group::in('ou=Office,ou=Groups,dc=local,dc=com')->get();

$accounting->groups()->attachMany($officeGroups);
```

### Removing Groups

To remove groups on a particular group, call the `groups` relationship, and then `detach()`:

> You must provide a `Model` instance into the `detach()` method.

```php
$accounting = Group::find('cn=Accounting,dc=local,dc=com');

$officeGroup = $accounting->groups()->where('cn', '=', 'Office')->first();

$accounting->groups()->detach($officeGroup);
```

#### Removing Multiple Groups

To remove multiple groups at once, provide an array of models to the `detachMany()` method:

```php
$accounting = Group::find('cn=Accounting,dc=local,dc=com');

$officeGroups = $accounting->groups()
                           ->in('ou=Office,ou=Groups,dc=local,dc=com')
                           ->get();

$accounting->groups()->detachMany($officeGroups);
```

#### Removing All Groups

To remove **all** immediate groups of a particular group, call the `detachAll()` method:

> A collection of all removed groups will be returned.

```php
$accounting = Group::find('cn=Accounting,dc=local,dc=com');

$removed = $accounting->groups()->detachAll();

foreach ($removed as $group) {
    echo "Removed: " . $group->getDn();
}
```
