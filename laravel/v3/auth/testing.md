---
title: Testing Authentication
description: LdapRecord-Laravel testing guide
---

# Testing

## Introduction

LdapRecord-Laravel prides itself on giving you a great and easy testing experience using
the [Directory Emulator](/docs/laravel/v3/testing#directory-emulator). Using it, we can
test authentication [rules](/docs/laravel/v3/auth/configuration#rules),
[scopes](/docs/laravel/v3/models#query-scopes) and group memberships.

## Getting Started

Before we begin, you must require the `doctrine/dbal` into your composers `require-dev` for testing.
This is due to the `$table->dropColumns(['guid', 'domain'])` call inside the additional
LdapRecord auth migration and that we are using SQLite in our test environment.

This package is required for modifying columns - as described in the
[Laravel documentation](https://laravel.com/docs/laravel/v3/migrations#modifying-columns).

To do so, run the following command:

```bash
composer require doctrine/dbal --dev
```

## Creating the test

Let's whip up a test by running the following command:

```bash
php artisan make:test LdapAuthenticationTest
```

inside our generated test, we'll make use of the following traits:

**DatabaseMigrations**

```text
Illuminate\Foundation\Testing\DatabaseMigrations
```

Using this trait will execute our migrations and ensure our database is ready to import our LDAP user.

**WithFaker**

```text
Illuminate\Foundation\Testing\WithFaker
```

Using this trait provides us with generating fake UUID's (great for creating mock "guids"), names and emails.

Let's add a `test_auth_works` method into the generated test:

```php
<?php

namespace Tests\Feature;

use Illuminate\Foundation\Testing\DatabaseMigrations;
use Illuminate\Foundation\Testing\WithFaker;
use Illuminate\Support\Facades\Auth;
use LdapRecord\Laravel\Testing\DirectoryEmulator;
use LdapRecord\Models\ActiveDirectory\User;
use Tests\TestCase;

class LdapAuthenticationTest extends TestCase
{
    use DatabaseMigrations, WithFaker;

    public function test_auth_works()
    {
        $fake = DirectoryEmulator::setup('default');

        $ldapUser = User::create([
            'cn' => $this->faker->name,
            'mail' => $this->faker->email,
            'objectguid' => $this->faker->uuid,
        ]);

        $fake->actingAs($ldapUser);

        $this->post('/login', [
            'email' => $ldapUser->mail[0],
            'password' => 'secret',
        ])->assertRedirect('/home');

        $user = Auth::user();

        $this->assertInstanceOf(\App\Models\User::class, $user);
        $this->assertEquals($ldapUser->mail[0], $user->email);
        $this->assertEquals($ldapUser->cn[0], $user->name);
    }
}
```

Let's deconstruct what's going on here step by step.

---

```php
$fake = DirectoryEmulator::setup('default');
```

This first line creates a new Directory Emulator for our LDAP connection named `default` inside
of our `config/ldap.php` file. It returns a fake LDAP connection that we can use to indicate
that the user we create in this fake directory will successfully pass LDAP authentication.

---

```php
$user = User::create([
    'cn' => $this->faker->name,
    'mail' => $this->faker->email,
    'objectguid' => $this->faker->uuid,
]);
```

On the second line, we're creating our fake LDAP user who will be signing in to our application.
You'll notice that we assign the attributes that are inside our `sync_attributes`
specified inside our `config/auth.php` file, as well as the users `objectguid`.

> If you're using OpenLDAP, the `objectguid` field may be `entryUUID` or `uid`.

This is a good place to test attribute synchronization.

---

```php
$fake->actingAs($user);
```

This third line, we are asserting that the user we have created will automatically pass
LDAP authentication. If we remove this line, attempting to authenticate as the
user will fail, as they are not allowed to bind using your fake connection.

---

```php
$this->post('/login', [
    'email' => $user->mail[0],
    'password' => 'secret',
])->assertRedirect('/home');
```

Fourth, we are sending a post request to our `login` page, with our LDAP users email address.
The password can be anything, since we asserted above (using the `actingAs()` method) that
the user **will** pass, regardless of what password we use.

If your application has [password synchronization](/docs/laravel/v3/auth/configuration/#database)
enabled, this is a good place to send various passwords and assert that the hashes
match after a successful login.

---

```php
$user = Auth::user();

$this->assertInstanceOf(\App\Models\User::class, $user);
$this->assertEquals($ldapUser->mail[0], $user->email);
$this->assertEquals($ldapUser->cn[0], $user->name);
```

Finally, we'll check to make sure we can retrieve the successfully authenticated
user and that their attributes were successfully synchronized into our Eloquent
database model.

## Scopes

To test scopes that you apply to the LdapRecord model you are using for authentication,
you will need to apply the attributes to the fake user you create to test that
they can be properly located during authentication.

For example, if you created a scope that enforces users to be inside an Organizational
Unit, then we must create our fake user inside that Organizational Unit for the
user to be located - as you would using a real LDAP directory.
Let's walk through this.

Below we have our scope that will enforce users to be located
inside an Organizational Unit named `Administrators`:

```php
namespace App\Ldap\Scopes;

use LdapRecord\Models\Model;
use LdapRecord\Models\Scope;
use LdapRecord\Query\Model\Builder;
use LdapRecord\Models\ActiveDirectory\OrganizationalUnit;

class AdministratorsScope implements Scope
{
    public function apply(Builder $query, Model $model)
    {
        $ou = OrganizationalUnit::where('ou', '=', 'Accounting')->first();

        $query->in($ou);
    }
}
```

And we have also added it into our model:

```php
namespace App\Ldap;

use LdapRecord\Models\Model;
use App\Ldap\Scopes\AdministratorsScope;

class User extends Model
{
    protected static function boot()
    {
        parent::boot();

        static::addGlobalScope(new AdministratorsScope());
    }
}
```

Now let's create our test. To do so, we'll set up everything as we have in the above test
example, but we will create our user inside the `Administrators` Organizational Unit:

```php
public function test_auth_works()
{
    $fake = DirectoryEmulator::setup('default');

    $ou = OrganizationalUnit::create(['ou' => 'Administrators']);

    $ldapUser = (new User)->inside($ou);

    $ldapUser->save([
        'mail' => $this->faker->email,
        'cn' => $this->faker->name,
        'objectguid' => $this->faker->uuid,
    ]);

    $fake->actingAs($ldapUser);

    $this->post('/login', [
        'email' => $ldapUser->mail[0],
        'password' => 'secret',
    ])->assertRedirect('/home');

    $user = Auth::user();

    $this->assertInstanceOf(\App\Models\User::class, $user);
    $this->assertEquals($ldapUser->mail[0], $user->email);
    $this->assertEquals($ldapUser->cn[0], $user->name);
}
```

To test the opposite of the above - such as a user who is not located inside the `Administrators`
OU, simply create them inside a different OU, or inside the root of your emulated directory:

```php
public function test_auth_fails()
{
    $fake = DirectoryEmulator::setup('default');

    $ldapUser = User::create([
        'cn' => $this->faker->name,
        'mail' => $this->faker->email,
        'objectguid' => $this->faker->uuid,
    ]);

    $fake->actingAs($ldapUser);

    $this->post('/login', [
        'email' => $ldapUser->mail[0],
        'password' => 'secret',
    ])->assertSessionHasErrors('email');

    $this->assertFalse(Auth::check());
}
```

Even though we have asserted that the user passes LDAP authentication (`$fake->actingAs($ldapUser)`),
authentication will fail due to the user not being able to be located due to our scope we have
created.

We have also modified our redirect assertion to instead validate that the `email` session
key contains errors. This key will contain the `Invalid credentials` message.

## Rules

As with testing scopes, to test rules we must either apply or omit data on
our fake user to test our LDAP authentication rules.

An authentication rule is great for checking if a user is a member of a certain group
before allowing them to authenticate. Let's walk through an example and test this.

Our application requires the user to be a member of a group called `Help Desk`.
With that requirement, here is our created authentication rule:

```php
<?php

namespace App\Ldap\Rules;

use LdapRecord\Laravel\Auth\Rule;
use LdapRecord\Models\Model as LdapRecord;
use LdapRecord\Models\ActiveDirectory\Group;
use Illuminate\Database\Eloquent\Model as Eloquent;

class HelpDeskEmployee implements Rule
{
    public function passes(LdapRecord $user, Eloquent $model = null): bool
    {
        $group = Group::where('name', '=', 'Help Desk')->first();

        return $this->user->groups()->exists($group);
    }
}
```

This rule has also been added into our providers configuration inside our `config/auth.php` file:

```php
// ...

'providers' => [
    // ...

    'ldap' => [
        // ...
        'rules' => [
            \App\Ldap\Rules\HelpDeskEmployee::class,
        ],
    ],
]
```

Now we can create our test to ensure only users who are members of the group can authenticate:

```php
public function test_auth_works()
{
    $fake = DirectoryEmulator::setup('default');

    $ldapGroup = Group::create(['cn' => 'Help Desk']);

    $ldapUser = User::create([
        'cn' => $this->faker->name,
        'mail' => $this->faker->email,
        'objectguid' => $this->faker->uuid,
        'memberof' => [$ldapGroup->getDn()],
    ]);

    $ldapGroup->members()->attach($ldapUser);

    $fake->actingAs($ldapUser);

    $this->post('/login', [
        'email' => $ldapUser->mail[0],
        'password' => 'secret',
    ])->assertRedirect('/home');

    $user = Auth::user();

    $this->assertInstanceOf(\App\Models\User::class, $user);
    $this->assertEquals($ldapUser->mail[0], $user->email);
    $this->assertEquals($ldapUser->cn[0], $user->name);
}
```

As you can see above, we created a `Help Desk` group, added the group into the users `memberof`
attribute (due to this field being virtual) and have attached them to the group.

Now let's create a test to ensure users who are not members of the group **can't** authenticate.

```php
public function test_auth_fails()
{
    $fake = DirectoryEmulator::setup('default');

    $ldapUser = User::create([
        'cn' => $this->faker->name,
        'mail' => $this->faker->email,
        'objectguid' => $this->faker->uuid,
    ]);

    $fake->actingAs($ldapUser);

    $this->post('/login', [
        'email' => $ldapUser->mail[0],
        'password' => 'secret',
    ])->assertSessionHasErrors('email');

    $this->assertFalse(Auth::check());
}
```

The above test passes because we have not added our LDAP user into any groups -
so the `exists()` check inside our rule returns `false`.

## SSO / Windows Authentication

To test Sigle-Sign-On (or Windows Authentication) for your Laravel application, you must
set the authenticating users [down-level logon name](https://docs.microsoft.com/en-us/windows/win32/secauthn/user-name-formats#down-level-logon-name)
as a server variable.

This server variable (typically `$_SERVER['AUTH_USER']`) is what the `WindowsAuthenticate`
middleware reads to locate the authenticated user from your LDAP directory.

To set server variables for upcoming requests inside your Laravel tests, use the `withServerVariables()` method:

```php
public function test_windows_authentication_works()
{
    DirectoryEmulator::setup('default');

    $ldapUser = User::create([
        'cn' => $this->faker->name,
        'mail' => $this->faker->email,
        'objectguid' => $this->faker->uuid,
        'samaccountname' => $this->faker->userName,
    ]);

    // Replace 'DOMAIN' with your domain from your configured LDAP
    // `base_dn`. For example, if your `base_dn` is equal to
    // 'dc=company,dc=com', then you would use 'COMPANY'.
    $authUser = implode('\\', [
        'DOMAIN', $ldapUser->getFirstAttribute('samaccountname')
    ]);

    // Set the server variables for the upcoming request.
    $this->withServerVariables([
        WindowsAuthenticate::$serverKey => $authUser
    ]);

    // Attempt accessing a protected page:
    $this->get('/dashboard')->assertOk();

    // Ensure the user was authenticated:
    $this->assertTrue(Auth::check());
}
```
