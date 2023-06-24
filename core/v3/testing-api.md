---
title: Testing API
description: The LdapRecord Testing API
---

# Available Test Utilties (API)

## `DirectoryFake`

### `setup`

Swap a `LdapRecord\Connection` in the `LdapRecord\Container` with an `LdapRecord\Testing\ConnectionFake`:

```php
use LdapRecord\Testing\DirectoryFake;

DirectoryFake::setup(); // LdapRecord\Testing\ConnectionFake
```

### `tearDown`

Revert the swapped connections in the `LdapRecord\Container` to their original `LdapRecord\Connection` connection instances.

This should typically be called in the `tearDown` of your test suite:

```php
use LdapRecord\Testing\DirectoryFake;

class TestCase extends PHPUnitTestCase
{
    public function tearDown(): void
    {
        DirectoryFake::tearDown();
        
        parent::tearDown();
    }
}
```

## `ConnectionFake` Method Listing

### `make`

Create a new `ConnectionFake` instance:

```php
use LdapRecord\Testing\ConnectionFake;

$config = ['...'];

$fake = ConnectionFake::make($config);
```

### `actingAs`

Set the user that is allowed to bind/authenticate to the `LdapFake`:

```php
ConnectionFake::make($config)->actingAs('cn=john,dc=local,dc=com');
```

You may also provide an `LdapRecord\Models\Model` instance:

```php
$user = User::make(['dn' => 'cn=john,dc=local,dc=com']);

ConnectionFake::make($config)->actingAs($user);
```

### `shouldBeConnected`

Set the connection fake to bypass bind attempts to the `LdapFake` as the user from the configuration:

```php
$fake = ConnectionFake::make($config)->shouldBeConnected();

$fake->isConnected(); // true
```

### `shouldNotBeConnected`

```php
$fake = ConnectionFake::make($config)->shouldNotBeConnected();

$fake->isConnected(); // false
```

## `LdapFake` Method Listing

### `operation`

Create a new `LdapExpectation` instance:

```php
use LdapRecord\Testing\LdapFake;

LdapFake::operation('read'); // LdapRecord\Testing\LdapExpectation
```

These expectations are typically added directly to an `LdapFake` instance using the `expect` method:

```php
DirectoryFake::setup()
    ->getLdapConnection() // LdapRecord\Testing\LdapFake
    ->expect(LdapFake::operation('...'))
```

### `expect`

Add an LDAP method expectation. It can receive an `LdapExpectation` instance or an array of expectations:

```php
use LdapRecord\Testing\LdapFake;
use LdapRecord\Testing\DirectoryFake;

$mockResults = ['...'];

// Single expectations...
DirectoryFake::setup()
    ->getLdapConnection()
    ->expect(LdapFake::operation('search')->andReturn($mockResults));

// Multiple expectations...
DirectoryFake::setup()
    ->getLdapConnection()
    ->expect([
        LdapFake::operation('bind')->andReturn(new \LdapRecord\LdapResultResponse),
        LdapFake::operation('search')->andReturn($mockResults),
    ]);

// Simple expectations using key-value pairs...
DirectoryFake::setup()
    ->getLdapConnection()
    ->expect([
        'bind' => new \LdapRecord\LdapResultResponse,
        'search' => $mockResults
    ]);
```

### `shouldAllowAnyBind`

Allow any bind attempt executed on the fake to succeed indefinitely:

```php
DirectoryFake::setup()
    ->getLdapConnection()
    ->shouldAllowAnyBind();
```

### `shouldAllowBindWith`

Allow a bind attempt from the given distinguished name to succeeed indefinitely:

```php
DirectoryFake::setup()
    ->getLdapConnection()
    ->shouldAllowBindWith('cn=john,dc=local,dc=com');
```
