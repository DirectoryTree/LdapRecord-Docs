---
title: Testing with LdapRecord
description: LdapRecord-Laravel testing guide
---

# Testing

## Introduction

Testing LDAP integration for PHP has always been quite difficult. Any type of integration
that is needed, you either need a real LDAP server to test against, or you mock
every response given and assume the logic you have in place will work until
you do live testing with a real LDAP server. This is finicky and hard to
test in an easy way.

That's where the LdapRecord **Directory Emulator** comes in.

## Directory Emulator

The Directory Emulator dynamically replaces the LDAP connection you specify with a fake
one. This fake connection sets up an SQLite database that resembles an LDAP directory
and allows you to store, update, delete, move, rename, and query LDAP objects through LdapRecord.

The SQLite database can be stored as file so you can utilize it in your application,
or you can use an in-memory database for running unit tests. It's the perfect
suite for testing the LDAP integration in your Laravel applications.

When you query a model that uses the connection you have setup with the Directory Emulator,
LdapRecord dynamically swaps query filters with Eloquent SQL queries, effectively
allowing you to query objects you create inside of your emulated directory.

## Getting Started

To begin, let's say we have an application that lists LDAP users inside of your configured directory.

Inside of our `config/ldap.php` file, we have defined our single `default` connection:

```php
// config/ldap.php

'default' => env('LDAP_CONNECTION', 'default'),

'connections' => [
    'default' => [
        // ...
        'base_dn' => 'dc=local,dc=com',
    ],
],
```

We have a `UsersController` that displays the LDAP users:

```php
use App\Ldap\User;

public class UsersController extends Controller
{
    public function index()
    {
        $users = User::get();

        return view('ldap.users', ['users' => $users]);
    }
}
```

And our view that simply lists all the users:

```html
<table>
  <thead>
    <tr>
      <th>Username</th>
      <th>Full Name</th>
    </tr>
  </thead>
  <tbody>
    @foreach($users as $user)
    <tr>
      <td>{{ $user->getFirstAttribute('samaccountname') }}</td>
      <td>{{ $user->getFirstAttribute('cn') }}</td>
    </tr>
    @endforeach
  </tbody>
</table>
```

To test this would involve a ton of mocking - which we want to avoid. With the Directory Emulator,
we can populate a fake LDAP server with objects utilizing our actual connection configuration.

Let's create a test for the `UserController@index` method. We'll create a Laravel test running the following command:

```bash
php artisan make:test LdapUserControllerTest
```

Now that we have our test, let's attempt to test our `index` method:

```php
use App\Ldap\User;
use LdapRecord\Laravel\Testing\DirectoryEmulator;

class LdapUserControllerTest extends TestCase
{
    public function test_index_works()
    {
        DirectoryEmulator::setup('default');

        $user = User::create([
            'cn' => 'John Doe',
            'samaccountname' => 'jdoe',
        ]);

        $this->assertEquals('cn=John Doe,dc=local,dc=com', $user->getDn());

        $this->visit('/ldap/users')
            ->assertSee($user->getFirstAttribute('cn'))
            ->assertSee($user->getFirstAttribute('samaccountname'));
    }
}
```

As with actual LDAP objects created in a live directory using LdapRecord models, when you create
LDAP objects in the emulated directory, they will use your connections configured `base_dn`
to create distinguished names.

## Using a SQLite File Database

To use a SQLite file database, you must supply an array to the second parameter
of the `DirectoryEmulator::setup` method and provide a file path using the
`database` key where you would like the SQLite file to be stored:

> If the file does not exist already, it will be created for you automatically.

```php
$file = storage_path('ldap_test_database.sqlite');

DirectoryEmulator::setup('default', ['database' => $file]);
```

## Emulated Queries

The Directory Emulator also emulates LDAP queries. However, there are
limitations. It currently does not support the following features:

- `raw` filters
- `anr` (Ambiguous Name Resolution) filters
- Virtual attributes (such as `memberof`, `entryUUID` and more)

Using the emulator, you can create a diverse LDAP object tree in your unit tests
and ensure your application is querying the proper objects.

> The examples below use LdapRecord models, but you may also use a raw LdapRecord `Connection`.

Let's update our example controller to retrieve users inside of a specific OU, and a company name:

```php
public class UsersController extends Controller
{
    public function index()
    {
        $ou = OrganizationalUnit::find('ou=Accounting,dc=local,dc=com');

        $users = User::in($ou)->where('company', '=', 'Acme')->get();

        return view('ldap.users', ['users' => $users]);
    }
}
```

Now we can update our test by creating an Organizational Unit and then creating a user
inside of that OU and assert that we only see the proper user:

```php
public function test_index_works()
{
    DirectoryEmulator::setup('default');

    $user = User::create([
        'cn' => 'John Doe',
        'samaccountname' => 'johndoe',
    ]);

    $ou = OrganizationalUnit::create([
        'ou' => 'Accounting',
    ]);

    $accountant = (new User)->inside($ou)->save([
        'cn' => 'Jane Doe',
        'samaccountname' => 'janedoe',
    ]);

    $this->visit('/ldap/users')
        ->assertSee($accountant->getFirstAttribute('cn'))
        ->assertSee($accountant->getFirstAttribute('samaccountname'))
        ->assertDontSee($user->getFirstAttribute('cn'))
        ->assertDontSee($user->getFirstAttribute('samaccountname'));
}
```

As you can see, this is extremely effective for testing your LDAP query integrations.

## Working with Relationships

### Has One

A `hasOne` relationship is easy to test. In this example, we will set the `manager` of another user:

```php
$user = User::create(['cn' => 'Jane']);

$manager = User::create(['cn' => 'John']);

$user->manager()->attach($manager);
```

Then, you can retrieve the users manager:

```php
$manager = $user->manager()->first();
```

### Has Many

Since some attributes are virtual in LDAP (such as the the `memberof` attribute on User
objects in Active Directory), you will have to populate some attributes manually
to mimic these virtual attributes. Let's walk through an example.

In our application, we want to test that a user is a member of a particular group.

First, we will create our group and user and add the user to the group:

```php
$user = User::create(['cn' => 'John']);

$group = Group::create(['cn' => 'Accounting']);

$user->groups()->attach($group);
```

Now, if we attempt to retrieve the `$group->members()` relationship, we won't receive
any results, but we will when using the `$user->groups()` relationship:

```php
// Empty collection returned!
$users = $group->members()->get();

// A collection containing 'Accounting' group returned.
$groups = $user->groups()->get();
```

The `$user->groups()` relationship works because it queries for groups that contain a `member`
attribute equal to the users distinguished name. This `member` attribute is set on the
`$group` instance that you pass into the `attach()` method.

The `$group->members()` relationship **does not work** because it queries for objects that contain
a `memberof` attribute to locate objects that are members. The `memberof` attribute is virtual,
so we must populate it manually to get our relationships working on both directions:

```php
$group = Group::create(['cn' => 'Accounting']);

$user = User::create([
    'cn' => 'John',
    'memberof' => [$group->getDn()],
]);

$user->groups()->attach($group);

// Returns the user 'John'.
$users = $group->members()->first();

// Returns 'Accounting' group.
$groups = $user->groups()->first();
```

### Has Many In

Similarly with the `hasMany` relationship, when using a `hasManyIn` relationship,
you must pre-populate a users virtual attribute for queries to properly locate
members of a group.
