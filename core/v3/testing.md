---
title: Testing
description: Testing with LdapRecord
---

# Testing

## Introduction

LdapRecord comes with robust 'fake' utilities to help you test your interactions with your LDAP server.

These test fakes allow you to add method expections and return mock results and responses,
allowing you to effectively test how your application behaves in various scenarios.

If you've ever used [Mockery](https://github.com/mockery/mockery) in your 
test suite before, you will see many similarities here.

## Setting Up

When you start using LdapRecord, you begin by adding a connection to the container
with its configuration to be able to interact with your LDAP server:

```php
// Adding an LDAP connection somewhere in your application...

\LdapRecord\Container::addConnection(
    new \LdapRecord\Connection(['...'])
);
```

Once a connection is added and is bootstrapped in your test environment, you may begin to test your interactions with the LDAP connection.

To begin, we can swap this LDAP connection out with the aforementioned 'fake', allowing us to add expectations to it, and mock responses.

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

The `setup` method accepts a connection name if you've added multiple connections, or have changed the name of the default LDAP connection:

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

## Tearing Down

Inside your test suite's tear down method (`tearDown` in PHPUnit's case), you should call the
`DirectoryFake::tearDown` method. This will put back all the replaced LDAP connections 
with their real equivalents, and perform additional method expectation assertions:

```php
 protected function tearDown(): void
{
    DirectoryFake::tearDown();

    parent::tearDown();
}
```

## Test Authentication / Bind

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

### Test Authentication / Bind Errors

To test bind errors, we can utilize `LdapFake`. This provides a way for us to
easily create an `LdapExpection` using the static `operation` method to deny
the bind attempt and include mock errors and diagnostic information:

```php
use LdapRecord\Testing\LdapFake;
use LdapRecord\Testing\DirectoryFake;

DirectoryFake::setup()
    ->getLdapConnection()
    ->expect(LdapFake::operation('bind')->once()->andReturnErrorResponse())
    ->shouldReturnErrorNumber(12)
    ->shouldReturnError('Some Error')
    ->shouldReturnDiagnosticMessage('Some Diagnostic Message');
```

```php
use LdapRecord\Container;
use LdapRecord\Connection;
use LdapRecord\BindException;

Container::addConnection(
    $connection = new Connection(['...'])
);

try {
    $connection->bind('cn=admin,dc=local,dc=com', 'secret');
} catch (BindException $e) {
    $error = $e->getDetailedError();
    
    $error->getErrorCode(); // 12
    $error->getErrorMessage(); // "Some Error"
    $error->getDiagnosticMessage(); // "Some Diagnostic Message"
}
```

## Operation Expectations

To test various LDAP operations to your LDAP server, LdapRecord provides a test utility called
`LdapFake`. This class extends the core `Ldap` class used for interacting with your server
and running the PHP `ldap_*` methods. The `LdapFake` class then overrides and intercepts 
these methods, providing the ability to add method expectations, similar to Mockery.

To retrieve the `LdapFake` instance, you may call the `getLdapConnection` method after setting up a `DirectoryFake`:

```php
use LdapRecord\Testing\DirectoryFake;

DirectoryFake::setup()->getLdapConnection(); // \LdapRecord\Testing\LdapFake
```

### Adding Expectations

To add expectations to the `LdapFake` connection instance, you may call 
the `expect` method, and provide a new `LdapExpectation` instance. The 
`LdapExpectation` class accepts the `LdapRecord\Ldap` method name to mock:

```php
use LdapRecord\Testing\LdapFake;
use LdapRecord\Testing\DirectoryFake;
use LdapRecord\Testing\LdapExpectation;

DirectoryFake::setup()
    ->getLdapConnection()
    ->expect((new LdapExpectation('read'))->once()->andReturn('...'));
```

You may also create an `LdapExpecation` using the static method `operation` on the `LdapFake` class:

```php
// Single expectations...
DirectoryFake::setup()
    ->getLdapConnection()
    ->expect(LdapFake::operation('search')->once()->andReturn($mockResults));

// Multiple expectations...
DirectoryFake::setup()
    ->getLdapConnection()
    ->expect([
        LdapFake::operation('bind')->once()->andReturn(new LdapResultResponse),
        LdapFake::operation('search')->once()->andReturn($mockResults),
    ]);

// Simple expectations using key-value pairs...
DirectoryFake::setup()
    ->getLdapConnection()
    ->expect([
        'bind' => new \LdapRecord\LdapResultResponse,
        'search' => $mockResults
    ]);

// Simple expectations using closures...
DirectoryFake::setup()
    ->getLdapConnection()
    ->expect([
        'bind' => function (LdapExpectation $bind) {
            $bind->with('cn=john,dc=local,dc=com', 'secret')->andReturnResponse();
        }),
    ]);
```

Use whichever method you find most suitable for your test.

### Expectation Arguments

Expectations can be provided with expected arguments that should
be passed to the LDAP operation using the `with` method:

```php
DirectoryFake::setup()
    ->getLdapConnection()
    ->expect(
        // Allow bind attempt matching the exact distinguished name and password.  
        LdapFake::operation('bind')->with('cn=john,dc=local,dc=com', 'secret')->andReturnResponse()
    );
```

Arguments are only validated if one is provided in its position:

```php
DirectoryFake::setup()
    ->getLdapConnection()
    ->expect(
        // Allow bind attempt with any password, but the distinguished name must match:
        LdapFake::operation('bind')->with('cn=john,dc=local,dc=com')->andReturnResponse()
    );
```

If no expected arguments are given to the expectation, then validation is not performed on them:

```php
DirectoryFake::setup()
    ->getLdapConnection()
    ->expect(
        // Allow bind attempts with any arguments.
        LdapFake::operation('bind')->andReturnResponse() 
    );
```

You may also provide a closure in each argument position to validate arguments. Each
closure should return a boolean to indicate whether the argument's value is valid:

```php
LdapFake::operation('bind')->with(
    fn ($dn) => $dn === 'cn=john,dc=local,dc=com'
    fn ($password) => $password === 'secret'
)->andReturnResponse()
```

When an expectation fails, an `LdapRecord\Testing\LdapExpectationException` exception will be thrown:

```text
LdapRecord\Testing\LdapExpectationException : Method [bind] should be called 1 times but was called 0 times.
```

### Expectation Order

Expectations that are added to the `LdapFake` are called in order (first to last). However, if 
an expectation is not given a number of times that it should be called, that means **it will 
be used indefinitely** for any calls to the method and return the same response each time:

```php
DirectoryFake::setup()
    ->getLdapConnection()
    ->expect([
        LdapFake::operation('bind')->with('cn=john,dc=local,dc=com')->andReturnResponse(),
        
        // ❌ Will never be reached due to above expectation.
        LdapFake::operation('bind')->with('cn=jane,dc=local,dc=com')->andReturnResponse(), 
    ]);
```

With the above example, the second `bind` expectation is never reached, because 
we have not provided a number of times that it should be executed.

Here is a fixed example, where we add a `once` to the expectation, meaning the 
expectation will be discarded the first time it is reached. The subsequent
`bind` call will be performed on the second expectation:

```php
DirectoryFake::setup()
    ->getLdapConnection()
    ->expect([
        LdapFake::operation('bind')->once()->with('cn=john,dc=local,dc=com')->andReturnResponse(),
        
         // ✅ Will be reached on subsequent bind attempt
        LdapFake::operation('bind')->once()->with('cn=jane,dc=local,dc=com')->andReturnResponse(),
    ]);
```

## Testing Operations

### Search

To test searching (`ldap_search`), we can add an expectation to our fake
LDAP connection on the `search` method, and return mock results:

> When returing mock LDAP results, you **must** return all attributes in an array, regardless
> if it's single valued, as this is how results are returned from your real LDAP server.

```php
use LdapRecord\Models\Entry;
use LdapRecord\Testing\LdapFake;
use LdapRecord\Testing\DirectoryFake;

$results = [
    [
        'mail' => ['john@local.com'], 
        'dn' => ['cn=John,dc=local,dc=com']
    ],
    [
        'mail' => ['jane@local.com'], 
        'dn' => ['cn=Jane,dc=local,dc=com']
    ],
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

### List

To test list (`ldap_list`), we can add an expectation to our fake
LDAP connection on the `list` method, and return mock results.

Since an `ldap_list` is the same as an `ldap_search` but without 
nested hierarchy searching, an almost identical test can be used:

```php
use LdapRecord\Models\Entry;
use LdapRecord\Testing\LdapFake;
use LdapRecord\Testing\DirectoryFake;

$results = [
    [
        'mail' => ['john@local.com'], 
        'dn' => ['cn=John,dc=local,dc=com']
    ],
    [
        'mail' => ['jane@local.com'], 
        'dn' => ['cn=Jane,dc=local,dc=com']
    ],
];

// Set up a connected fake:
$fake = DirectoryFake::setup()->shouldBeConnected();

// Expect a search and return a result:
$fake->getLdapConnection()->expect(
    LdapFake::operation('list')->andReturn($results)
);

// Execute a search and assert the expected results:
foreach (Entry::get() as $index => $user) {
    $this->assertEquals($results[$index]['mail'], $user->mail);
    $this->assertEquals($results[$index]['dn'][0], $user->getDn());
}
```

### Read

To test a read (`ldap_read`), we can add an expectation to our fake
LDAP connection on the `list` method, and return mock results.

> Similarly to the above list and search tests, the mock results here still 
> need to be provided in a nested array, but only one result needs to be 
> included, as this is what will be returned from your LDAP server.

```php
$results = [
    [
        'mail' => ['john@local.com'],
        'dn' => ['cn=John,dc=local,dc=com'],
    ],
];

// Set up a connected fake:
$fake = DirectoryFake::setup()->shouldBeConnected();

// Expect a search and return a result:
$fake->getLdapConnection()->expect(
    LdapFake::operation('read')->andReturn($results)
);

$user = Entry::find('cn=John,dc=local,dc=com');

$this->assertEquals($results[0]['mail'], $user->mail);
$this->assertEquals($results[0]['dn'][0], $user->getDn());
```

### Paginate

To test a pagianted query that returns more than one page of results, we need to add a
more complicated set of expectations on the `parseResult` method. This method is
used to retrieve details about the pagintation request (per page), and also
used to parse the result of a page:

```php
// Define the pages of mock results.
$pages = [
    [['count' => 1, 'objectclass' => ['foo'], 'dn' => ['cn=John,dc=local,dc=com']]],
    [['count' => 1, 'objectclass' => ['bar'], 'dn' => ['cn=Jane,dc=local,dc=com']]],
];

DirectoryFake::setup()
    ->shouldBeConnected()
    ->getLdapConnection()
    ->expect([
        // Return the first page of results.
        LdapFake::operation('search')->once()->andReturn($pages[0]),
    
        // Return the pagination response, indicating more pages to load.
        LdapFake::operation('parseResult')->once()->andReturnResponse(controls: [
            LDAP_CONTROL_PAGEDRESULTS => [
                'value' => [
                    'size' => 1,
                    
                    // Indicate more pages to load by returning a non-empty string as a cookie.
                    'cookie' => '1234',
                ],
            ],
        ]),
    
        // Return the parsed results from the first page of the pagination request.
        LdapFake::operation('parseResult')->once()->with(fn ($results) => (
            $results === $pages[0]
        ))->andReturnResponse(),
    
        // Return the next page of results.
        LdapFake::operation('search')->once()->andReturn($pages[1]),
        
        // Return the next pagination response, indicating *no more* pages to load.
        LdapFake::operation('parseResult')->once()->andReturnResponse(controls: [
            LDAP_CONTROL_PAGEDRESULTS => [
                'value' => [
                    'size' => 1,
                    
                    // Indicate that there are no more pages to load.
                    'cookie' => null, 
                ],
            ],
        ]),
    
        // Return the parsed results from the second page of the pagination request.
        LdapFake::operation('parseResult')->once()->with(fn ($results) => (
            $results === $pages[1]
        ))->andReturnResponse(), 
    ]);

$this->assertCount($users = User::paginate());
$this->assertTrue($users->contains('cn=John,dc=local,dc=com'));
$this->assertTrue($users->contains('cn=Jane,dc=local,dc=com'));
```

### Chunk

Similarly to the above pagination test, we can scaffold our chunk 
test nearly identically, but perform assertions differently, 
since the chunk method does not return results:

```php
// Define the pages of mock results.
$pages = [
    [['count' => 1, 'objectclass' => ['foo']]],
    [['count' => 1, 'objectclass' => ['bar']]],
];

DirectoryFake::setup()
    ->shouldBeConnected()
    ->getLdapConnection()
    ->expect([
        // Return the first page of results.
        LdapFake::operation('search')->once()->andReturn($pages[0]),
    
        // Return the pagination response, indicating more pages to load.
        LdapFake::operation('parseResult')->once()->andReturnResponse(controls: [
            LDAP_CONTROL_PAGEDRESULTS => [
                'value' => [
                    'size' => 1,
                    
                    // Indicate more pages to load by returning a non-empty string as a cookie.
                    'cookie' => '1234',
                ],
            ],
        ]),
    
        // Return the parsed results from the first page of the pagination request.
        LdapFake::operation('parseResult')->once()->with(fn ($results) => (
            $results === $pages[0]
        ))->andReturnResponse(),
    
        // Return the next page of results.
        LdapFake::operation('search')->once()->andReturn($pages[1]),
        
        // Return the next pagination response, indicating *no more* pages to load.
        LdapFake::operation('parseResult')->once()->andReturnResponse(controls: [
            LDAP_CONTROL_PAGEDRESULTS => [
                'value' => [
                    'size' => 1,
                    
                    // Indicate that there are no more pages to load.
                    'cookie' => null, 
                ],
            ],
        ]),
    
        // Return the parsed results from the second page of the pagination request.
        LdapFake::operation('parseResult')->once()->with(fn ($results) => (
            $results === $pages[1]
        ))->andReturnResponse(), 
    ]);

User::chunk(1, function ($results, $page) use ($pages) {
    // $page in the chunk callback starts at 1. We
    // will decrement it by one to retrieve the
    // correct mock results for the page. 
    $this->assertEquals($pages[--$page], $results);
});
```

### Create

To test an object creation (`ldap_add`), we can add an expectation to our fake
LDAP connection on the `add` method, validate the properties using the `with` 
method, and return true -- indicating creation was successful:

```php
DirectoryFake::setup()
    ->shouldBeConnected()
    ->getLdapConnection()
    ->expect(
        LdapFake::operation('add')->with('cn=John Doe,dc=local,dc=com', function (array $attributes) {
            return $attributes['cn'][0] === 'John Doe'
                && $attributes['mail'][0] === 'jdoe@local.com'
                && $attributes['objectclass'] === [
                    'top',
                    'person',
                    'organizationalperson',
                    'user',
                ];
        })->andReturnTrue()
    );

$model = new User();

$model->cn = 'John Doe';
$model->mail = 'jdoe@local.com';

$model->save();

$this->assertTrue($model->exists);

// You may also validate the model's properties after
// creation to ensure they have been set properly:
$this->assertEquals('cn=John Doe,dc=local,dc=com', $model->getDn());
$this->assertEquals('jdoe@local.com', $model->getFirstAttribute('mail'));
```

### Update

To test an object creation (`ldap_modify_batch`), we can add an expectation to our fake
LDAP connection on the `modifyBatch` method, validate the properties using the `with`
method, and return true -- indicating that the update was successful:

```php
DirectoryFake::setup()
    ->shouldBeConnected()
    ->getLdapConnection()
    ->expect([
        LdapFake::operation('modifyBatch')->once()->with('cn=john,dc=local,dc=com', function ($mods) {
            return count($mods) === 1
                && $mods[0]['attrib'] === 'mail'
                && $mods[0]['modtype'] === LDAP_MODIFY_BATCH_ADD
                && $mods[0]['values'] === ['john.doe@local.com'];
        })->andReturnTrue(),
    ]);

$user = new User();

$user->setRawAttributes([
    'dn' => ['cn=john,dc=local,dc=com'],
]);

$user->mail = 'john.doe@local.com';

$user->save();
```

### Attribute Add

To test adding an attribute to an LDAP object (`ldap_mod_add`), we can add an expectation to our fake
LDAP connection on the `modAdd` method, validate the properties using the `with`
method, and return true -- indicating that the update was successful:

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

### Attribute Remove

Similarly to the above test, to test removing an attribute to an LDAP object (`ldap_mod_del`), we can 
add an expectation to our fake LDAP connection on the `modDelete` method, validate the properties
using the `with` method, and return true -- indicating that the update was successful:

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

### Rename

To test renaming an LDAP object, we can add an expectation to 
the "rename" method that the current full distinguished 
name of the object, and the new RDN of the object: 

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

$model->setRawAttributes([
    'dn' => 'cn=John Doe,dc=acme,dc=org'
]);

$model->rename('Jane Doe');

$this->assertTrue($model->wasRecentlyRenamed);
$this->assertEquals('Jane Doe', $model->getName());
```

### Move

Similarly to the above test, moving an object performs 
a "rename"under the hood, but instead keeps the same
RDN, and is provided with a new base DN:

```php
DirectoryFake::setup()
    ->getLdapConnection()
    ->expect(
        LdapFake::operation('rename')
            ->with('cn=John Doe,dc=acme,dc=org', 'cn=John Doe', 'ou=Users,dc=acme,dc=org')
            ->andReturnTrue()
    );

$model = new Entry();

$model->setRawAttributes([
    'dn' => 'cn=John Doe,dc=acme,dc=org'
]);

$model->move('ou=Users,dc=acme,dc=org');

$this->assertTrue($model->wasRecentlyRenamed);
$this->assertEquals('cn=John Doe,ou=Users,dc=acme,dc=org', $model->getDn());
```
