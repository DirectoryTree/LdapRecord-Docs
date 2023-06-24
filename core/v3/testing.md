---
title: Testing
description: Testing with LdapRecord
---

# Testing

## Introduction

LdapRecord comes with robust 'fake' utilities to help you test your interactions with your LDAP server.

These test fakes allow you to add method expections and return mock results and responses,
allowing you to effectively test how your application behaves in various scenarios.

## Getting Started

When you begin using LdapRecord, you first add a connection to the container
with its configuration to be able to interact with your LDAP server.

```php
// Adding an LDAP connection somewhere in your application...

\LdapRecord\Container::addConnection($connection);
```

Once a connection is added and is bootstrapped in your test environment, you may begin to test your interactions with the LDAP connection.

To begin, we must swap this LDAP connection out with a fake, allowing us to add expectations to it, and mock responses.

This swapping of the LDAP connection with a fake is done with the `DirectoryFake::setup` method:

```php
use LdapRecord\Testing\DirectoryFake;

class LdapTest extends TestCase
{
    public function test()
    {
        // Swap default connection with a fake...
        $fake = DirectoryFake::setup();
        
        // Returns instanceof \LdapRecord\Testing\ConnectionFake
        // \LdapRecord\Container::getDefaultConnection();
        
        // ...
    }
}
```

The `setup` method accepts a connection name, if you've added multiple connections, or have changed the name of the default LDAP connection:

```php
// Somewhere in your application...

\LdapRecord\Container::addConnection($connection, 'alpha');
```

```php
use LdapRecord\Testing\DirectoryFake;

public function test()
{
    $fake = DirectoryFake::setup('alpha');
    
    // Returns instanceof \LdapRecord\Testing\ConnectionFake
    // \LdapRecord\Container::getConnection('alpha');
}
```

## Test Bind

The `DirectoryFake` provides a convenient mechanism for mocking bind attempts to your LDAP server.

To permit a bind attempt to your server underneath a particular user, call the `actingAs` method:

> The `actingAs` method permits any password to be used, as long as the distinguished name given matches.

```php
DirectoryFake::setup()->actingAs('cn=admin,dc=local,dc=com');
```

```php
\LdapRecord\Container::addConnection(
    $connection = new \LdapRecord\Connection(['...'])
);

// Successful.
$connection->bind('cn=admin,dc=local,dc=com', 'secret');
```

### Test Bind Errors

To test bind errors, we can utilize `LdapFake`, which provides a way for us
to easily create an `LdapExpection` to deny the bind attempt and include
mock errors and diagnostic information:

```php
use LdapRecord\Testing\LdapFake;
use LdapRecord\Testing\DirectoryFake;

DirectoryFake::setup()
    ->getLdapConnection()
    ->expect(LdapFake::operation('bind')->andReturnErrorResponse())
    ->shouldReturnErrorNumber(12)
    ->shouldReturnError('Some Error')
    ->shouldReturnDiagnosticMessage('Some Diagnostic Message');
```

```php
\LdapRecord\Container::addConnection(
    $connection = new \LdapRecord\Connection(['...'])
);

try {
    $connection->bind('cn=admin,dc=local,dc=com', 'secret');
} catch (\LdapRecord\BindException $e) {
    $error = $e->getDetailedError();
    
    $error->getErrorCode(); // 12
    $error->getErrorMessage(); // "Some Error"
    $error->getDiagnosticMessage(); // "Some Diagnostic Message"
}
```

## Test Search

To test searching, we can ret

> When returing mock LDAP results, you **must** return all attributes in an array, regardless
> if it's single valued, as this is how results are returned from your real LDAP server.

```php
use LdapRecord\Models\Entry;
use LdapRecord\Testing\LdapFake;
use LdapRecord\Testing\DirectoryFake;

$results = [
    ['mail' => ['john@local.com'], 'dn' => ['cn=John,dc=local,dc=com']],
    ['mail' => ['jane@local.com'], 'dn' => ['cn=Jane,dc=local,dc=com']],
];

// Set up a connected fake:
$fake = DirectoryFake::setup()->shouldBeConnected();

// Expect a search and return a result:
$fake->getLdapConnection()->expect(
    LdapFake::operation('search')->andReturn($results)
);

// Execute a search and assert the expected results:
foreach (Entry::get() as $index => $user) {
    $this->assertEquals($results[$index]['mail'], $user->mail);
    $this->assertEquals($results[$index]['dn'][0], $user->getDn());
}
```

## Test List

## Test Read

```php
$result = [
    [
        'mail' => ['john@local.com'],
        'dn' => ['cn=John,dc=local,dc=com'],
    ],
];

DirectoryFake::setup()
    ->getLdapConnection()
    ->expect(['read' => $result]);

$user = Entry::find('cn=John,dc=local,dc=com');

$this->assertEquals($result[0]['mail'], $user->mail);
$this->assertEquals($result[0]['dn'][0], $user->getDn());
```

## Test Pagination

## Test Create

## Test Update

## Test Attribute Add

```php
$model = new Entry();

$model->setRawAttributes(['dn' => 'cn=John Doe,dc=acme,dc=org']);

DirectoryFake::setup()
    ->getLdapConnection()
    ->expect(
        LdapFake::operation('modAdd')
            ->with($model->getDn(), ['mail' => ['jdoe@local.com']])
            ->andReturnTrue()
    );

$model->addAttribute('mail', 'jdoe@local.com');

$this->assertEquals('jdoe@local.com', $model->getFirstAttribute('mail'));
```

## Test Attribute Remove

```php
$model = new Entry();

$model->setRawAttributes([
    'dn' => ['cn=John Doe,dc=acme,dc=org'],
    'mail' => ['jdoe@local.com'],
]);

$this->assertEquals('jdoe@local.com', $model->getFirstAttribute('mail'));

DirectoryFake::setup()
    ->getLdapConnection()
    ->expect(
        LdapFake::operation('modDelete')
            ->with($model->getDn(), ['mail' => ['jdoe@local.com']])
            ->andReturnTrue()
    );

$model->removeAttribute('mail', 'jdoe@local.com');

$this->assertNull($model->getFirstAttribute('mail'));
```

## Test Rename

```php
use LdapRecord\Testing\LdapFake;
use LdapRecord\Testing\DirectoryFake;

DirectoryFake::setup()
    ->getLdapConnection()
    ->expect(
        LdapFake::operation('rename')
            ->with('cn=John Doe,dc=acme,dc=org', 'cn=Jane Doe')
            ->andReturnTrue()
    );

$model = new Entry();

$model->setRawAttributes(['dn' => 'cn=John Doe,dc=acme,dc=org']);

$model->rename('Jane Doe');

$this->assertTrue($model->wasRecentlyRenamed);
$this->assertEquals('Jane Doe', $model->getName());
```