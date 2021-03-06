---
title: Testing
description: Testing with LdapRecord
---

# Testing

## Introduction

LdapRecord comes with a utility that allow you to test bind attempts
against a fake server and return custom error codes & responses.

This allows you to test how your application responds to authentication failures and error messages.

## Test Case Setup

To begin, initialize the fake directory using the `DirectoryFake::setup` method. This method
accepts the name of your LDAP connection that you initialize in your application.

If you do not provide a name, your default LDAP connection name will be used.

Upon calling the `setup` method, your LDAP connection will be swapped out of the
connection `Container` and replaced with a `ConnectionFake`.

Let's walk through an example of testing an application that uses LDAP authentication.

Here's our example controller:

```php
class AuthController
{
    public function __construct()
    {
        Container::addConnection([
            'hosts' => ['10.0.0.1']
        ]);
    }

    public function login()
    {
        $connection = Container::getDefaultConnection();

        $username = $_POST['username'];
        $password = $_POST['password'];

        if ($connection->auth()->attempt($username, $password)) {
            return "Your password is valid!";
        }

        return "Username or password is incorrect.";
    }
}
```

Now let's test it:

```php
use LdapRecord\Testing\DirectoryFake;

class LoginTest extends TestCase
{
    public function test_login()
    {
        $user = 'cn=User,dc=local,dc=com';

        DirectoryFake::setup()->actingAs($user);

        // Execute HTTP post request somehow in your testing framework...

        $this->post('/login', [
            'username' => $user,
            'password' => 'secret',
        ])->assertSee("Your password is valid!");

        $this->post('/login', [
            'username' => 'invalid',
            'password' => 'secret',
        ])->assertSee("Username or password is incorrect.");
    }
}
```

This is a small example of how you can test bind attempts to your LDAP server.

## Responses and Error Codes

When testing connectivity to your LDAP server, you may wish to test error codes and
messages that may be returned when a bind attempt fails. To do this, you can
use the `ConnectionFake` to retrieve a fake LDAP connection to return
error codes when a bind attempt fails.

Let's walk through an example of an authentication controller that will
retrieve the last LDAP error and determine the cause of the bind
failure.

Let's walk through an example of testing an application that uses LDAP authentication.

Here's our example controller:

```php
class AuthController
{
    public function __construct()
    {
        Container::addConnection([
            'hosts' => ['10.0.0.1']
        ]);
    }

    public function login()
    {
        $connection = Container::getDefaultConnection();

        $username = $_POST['username'];
        $password = $_POST['password'];

        if ($connection->auth()->attempt($username, $password)) {
            return "Your password is valid!";
        }

        $error = $connection->getLdapConnection()->getDiagnosticMessage();

        if (strpos($error, '532') !== false) {
            return "Your password has expired.";
        } elseif (strpos($error, '533') !== false) {
            return "Your account is disabled.";
        } elseif (strpos($error, '701') !== false) {
             return "Your account has expired.";
        } elseif (strpos($error, '775') !== false) {
             return "Your account is locked.";
        }

        return "Username or password is incorrect.";
    }
}
```

You can see above that we are pulling the diagnostic message of the last failed bind attempt.

This diagnostic message contains an error code that you can use to tell the user why they failed logging in.

Here is how we would test the above controller:

```php
use LdapRecord\Testing\DirectoryFake;

class LoginTest extends TestCase
{
    public function test_login()
    {
        $user = 'cn=User,dc=local,dc=com';

        $fake = DirectoryFake::setup()->actingAs($user);

        $fake->getLdapConnection()->shouldReturnDiagnosticMessage('Failed: 775');

        // Execute HTTP post request somehow in your testing framework...

        $this->post('/login', [
            'username' => $user,
            'password' => 'secret',
        ])->assertSee("Your account is locked.");
    }
}
```
