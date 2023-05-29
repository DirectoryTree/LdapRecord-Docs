---
title: Using LdapRecord-Laravel with Laravel Sanctum
description: Setting up LDAP authentication with Laravel Sanctum
---

# Laravel Sanctum

## Introduction

[Laravel Sanctum](https://laravel.com/docs/sanctum) provides a featherweight authentication system for SPAs and simple APIs.

Since LdapRecord-Laravel provides a database authentication driver, it integrates
with Sanctum directly, similarly to the built in `eloquent` driver.

> **Important**: Before getting started, please complete the below guides:
>
> - [Installation Guide](/docs/laravel/v3/auth/database/installation)
> - [Configuration Guide](/docs/laravel/v3/auth/database/configuration)
> - [Sanctum Installation Guide](https://laravel.com/docs/sanctum#installation)

## Preparing The User Eloquent Model

If you've followed the above guides, your Eloquent user model should resemble the below:

```php
// app/Models/User.php

// ...
use Laravel\Sanctum\HasApiTokens;
use LdapRecord\Laravel\Auth\HasLdapUser;
use LdapRecord\Laravel\Auth\LdapAuthenticatable;

class User extends Authenticatable implements LdapAuthenticatable
{
    // ...
    use HasApiTokens;
    use AuthenticatesWithLdap;
    // ...
}
```

## SPA Authentication

> **Important**:
> 
> Please read Laravel Sanctum's [SPA Authentication setup guide](https://laravel.com/docs/sanctum#spa-authentication) before proceeding.

SPA Authentication means that you have a frontend-based application
that will be sending requests to your own application's protected
route endpoints (via the `auth:sanctum` middleware).

### Preparing The Authentication Guard

Laravel Sanctum will utilize the `web` authentication guard specified in your `config/auth.php` file by default.

Make sure this guard exists and is utilizing the `session` driver:

```php
// config/auth.php

'guards' => [
    'web' => [
        'driver' => 'session',
        'provider' => 'users',
    ],
    
    // ...
],
```

If you want to change the guard Sanctum uses, publish its configuration file by running the below command:

> **Important**: As mentioned above, any custom guard must use a `session` driver for Sanctum to function.

```
php artisan vendor:publish --tag="sanctum-config"
```

Then, update the `guard` configuration option:

```php
// config/sanctum.php

'guard' => ['web'],
```

### Setting Up The Sanctum Middleware

If you're going to be sending requests to your `/api` endpoints from
your application's frontend, you must insert a Sanctum middleware
into your Laravel application's `api` middleware group for those
requests to be automatically authenticated.

This means that users who have logged into your frontend application
will not need to manually provide a Sanctum token to send requests
to your protected `/api` endpoints. This middleware is responsible
for booting up the session during API requests that are received
from your frontend:

> **Important**:
> 
> The position of this middleware is crucial. The `throttle:api`
> middleware will utilize a different throttle cache key
> for authenticated users than guests.
> 
> Since the `EnsureFrontendRequestsAreStateful` is inserted before `throttle:api`,
> the session will be started and an authenticated user will exist, allowing
> the `throttle:api` to access them and bind a unique throttle key to them.
>
> You may have to tweak this `throttle:api` middleware if your
> frontend application sends large amounts of API requests.

```php
// app/Http/Kernel.php

protected $middlewareGroups = [
    // ...

    'api' => [
        \Laravel\Sanctum\Http\Middleware\EnsureFrontendRequestsAreStateful::class, // <-- Inserted here.
        'throttle:api',
        \Illuminate\Routing\Middleware\SubstituteBindings::class,
    ],
],
```

How does Sanctum know that the request came from your frontend? Well, it does
this by checking the domain that the request was sent from to see if it
matches with your application server's URL, or one of the URL's
configured inside the `sanctum.php` configuration file:

```php
// config/sanctum.php

'stateful' => explode(',', env('SANCTUM_STATEFUL_DOMAINS', sprintf(
    '%s%s',
    'localhost,localhost:3000,127.0.0.1,127.0.0.1:8000,::1',
    Sanctum::currentApplicationUrlWithPort()
))),
```

> As you can see, various `localhost` domains are included, which is why we can test with Sanctum locally.

### Logging In

> **Important**:
> 
> It's recommended to use [Laravel Fortify](https://laravel.com/docs/fortify)
> as a starting point for authenticating users. This guide assumes you
> have set up authentication using one of the documented packages.

As mentioned in the Laravel Sanctum documentation, you must first initialize
a CSRF cookie by requesting one from an endpoint Sanctum integrates
into your application automatically (`/sanctum/csrf-token`). When
you send a request to this endpoint, your application will send
cookie headers back, containing the X-CSRF token.

Once a successful `204` (No content) response is received from the CSRF token
endpoint resulting in a new CSRF cookie containing the token, you may send a
login request to your application. This login request will initialize the
session by sending back a new session cookie in the header
(upon providing valid credentials).

When this session cookie is received by your web browser, `axios` (and
other HTTP JavaScript clients) should automatically send this cookie
along with any subsequent requests. Laravel Sanctum will read this
cookie that is sent along your request and authenticate the user
for you, allowing the user to access protected routes.

Here's how a login request via `axios` could be made:

```js
let credentials = {
    email: 'john@local.com',
    password: 'secret',
};

axios.get('/sanctum/csrf-cookie').then(response => {
    axios.post('/login', credentials).then('...');
});
```

## API Token Authentication

> **Important**:
> 
> Please read Laravel Sanctum's [API Token Authentication setup guide](https://laravel.com/docs/sanctum#api-token-authentication) before proceeding.

If your application is going to provide an API to end users or external
services, you will need to implement API token authentication.

### Issuing API Tokens

To start issuing new user API tokens, we will take Sanctum's
[default suggested endpoint for issuing tokens](https://laravel.com/docs/sanctum#issuing-mobile-api-tokens),
and then tweak it a little bit.

Open your `routes/api.php` file, and paste the below:

```php
// routes/api.php

use App\Models\User;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Route;
use Illuminate\Support\Facades\Auth;
use Illuminate\Validation\ValidationException;

Route::middleware('auth:sanctum')->get('/user', function (Request $request) {
    return $request->user();
});

Route::post('sanctum/token', function (Request $request) {
    $request->validate([
        'email' => 'required|email',
        'password' => 'required',
        'device_name' => 'required',
    ]);

    // Remember, an LDAP query will be executed on all the array
    // elements in the credentials array (excluding "password").
    // Here, we're locating a user via their "mail" attribute.
    $credentials = [
        'mail' => $request->email,
        'password' => $request->password,
    ];

    if (Auth::validate($credentials)) {
        $user = Auth::getLastAttempted();

        return [
            'token' => $user->createToken($request->device_name)->plainTextToken
        ];
    }

    throw ValidationException::withMessages([
        'email' => ['The provided credentials are incorrect.'],
    ]);
});
```

This is all that is needed to generate new API tokens, and to start
using those tokens to authenticate against your server with.

Let's make sure these endpoints work with some tests.

## Testing

### Tinkerwell Testing

To test your Sanctum endpoint with [Tinkerwell](https://tinkerwell.app), serve your Laravel application by running the below command:

```bash
php artisan serve

> Starting Laravel development server: http://127.0.0.1:8000
```

Then, send a post request to `api/sanctum/token`:

```php
use Illuminate\Support\Facades\Http;

$data = [
  'email' => 'johndoe@local.com',
  'password' => 'secret',
  'device_name' => 'browser',
];

$response = Http::baseUrl('http://127.0.0.1:8000')
  ->withHeaders(['accept' => 'application/json'])
  ->post('api/sanctum/token', $data)
  ->json();

// ['token' => '5|iSO2wH8W....']
dd($response);
```

To ensure the token works, capture a returned token from the above request, and send it with `Authorization` headers to `api/user`:

```php
$token = '5|iSO2wH8W....';

$response = Http::baseUrl('http://127.0.0.1:8000')
  ->withHeaders(['accept' => 'application/json'])
  ->withToken($token)
  ->get('api/user')
  ->body();

// ['id' => 1, 'name' => 'John Doe', ...]
dd($response);
```

### PHPUnit Testing

> **Important**: If you are using SQLite to test, remember to install
> `doctrine/dbal` before getting started, as mentioned in the
> [test guide](https://ldaprecord.com/docs/laravel/v3/auth/testing/#getting-started):
>
> ```bash
> composer require doctrine/dbal --dev
> ```

To begin, let's create a Sanctum test to make sure both our API endpoints are working:

```bash
php artisan make:test SanctumTokenTest
```

Open the new file and erase the existing example test:

```php
namespace Tests\Feature;

use Tests\TestCase;
use Illuminate\Foundation\Testing\WithFaker;
use Illuminate\Foundation\Testing\RefreshDatabase;

class SanctumTokenTest extends TestCase
{
}
```

We'll need to `use` the `DatabaseMigrations` trait to run our application's migrations:

```php
// ...
use Illuminate\Foundation\Testing\DatabaseMigrations;

class SanctumTokenTest extends TestCase
{
    use DatabaseMigrations;
}
```

Now let's add a test to ensure our LDAP user can authenticate and get an API token:

```php
namespace Tests\Feature;

use Tests\TestCase;
use App\Models\User;
use Laravel\Sanctum\PersonalAccessToken;
use Illuminate\Testing\Fluent\AssertableJson;
use LdapRecord\Laravel\Testing\DirectoryEmulator;
use Illuminate\Foundation\Testing\DatabaseMigrations;
use LdapRecord\Models\ActiveDirectory\User as LdapUser;

class SanctumTokenTest extends TestCase
{
    use DatabaseMigrations;

    protected function tearDown(): void
    {
        DirectoryEmulator::tearDown();

        parent::tearDown();
    }

    public function testLdapUserCanRetrieveToken()
    {
        // Set up the directory emulator to create a fake, existing LDAP user:
        $fake = DirectoryEmulator::setup();

        // Remember to supply the "sync_attributes" you have assigned inside
        // your config/auth.php. You will receive an exception if any
        // non-nullable SQL fields are not present on the LDAP user:
        $ldapUser = LdapUser::create([
            'cn' => 'John Doe',
            'mail' => 'john@local.com',
        ]);

        // Set the user to automatically pass LDAP authentication:
        $fake->actingAs($ldapUser);

        // Post the token endpoint and ensure a new token was created:
        $this->postJson('api/sanctum/token', [
            'email' => $ldapUser->mail[0],
            'password' => 'secret',
            'device_name' => 'browser',
        ])->assertJson(
            fn (AssertableJson $json) => $json->whereType('token', 'string')
        );

        // Ensure the user was imported:
        $this->assertDatabaseHas(User::class, [
            'email' => $ldapUser->mail[0],
            'name' => $ldapUser->cn[0],
        ]);

        // Ensure a token exists:
        $this->assertDatabaseCount(PersonalAccessToken::class, 1);
    }
}
```

Great, now let's add a test to ensure an existing LDAP user that fails authentication is not imported, and no token is created:

```php
class SanctumTokenTest extends TestCase
{
    // ...
        
    public function testLdapUserCannotRetrieveTokenWithInvalidPassword()
    {
        DirectoryEmulator::setup();

        $ldapUser = LdapUser::create([
            'cn' => 'John Doe',
            'mail' => 'john@local.com',
        ]);

        // Post the token endpoint and ensure a validation error is thrown:
        $this->postJson('api/sanctum/token', [
            'email' => $ldapUser->mail[0],
            'password' => 'secret',
            'device_name' => 'browser',
        ])->assertJsonValidationErrors([
            'email' => 'The provided credentials are incorrect.'
        ]);

        // Ensure the user was not imported:
        $this->assertDatabaseMissing(User::class, [
            'email' => $ldapUser->mail[0],
            'name' => $ldapUser->cn[0],
        ]);

        // Ensure no token was created:
        $this->assertDatabaseCount(PersonalAccessToken::class, 0);
    }
}
```

Nice, now let's add another to ensure our LDAP users can authenticate using their token:

```php
class SanctumTokenTest extends TestCase
{
    // ...

    public function testLdapUserCanUseTokenOnProtectedRoutes()
    {
        $fake = DirectoryEmulator::setup();

        $ldapUser = LdapUser::create([
            'cn' => 'John Doe',
            'mail' => 'john@local.com',
        ]);

        $fake->actingAs($ldapUser);

        // Post the token endpoint and grab the API token:
        $plainTextToken = $this->postJson('api/sanctum/token', [
            'email' => $ldapUser->mail[0],
            'password' => 'secret',
            'device_name' => 'browser',
        ])->json('token');

        // Attempt retrieving the user using the returned token:
        $this->getJson('api/user', [
            'Authorization' => "Bearer $plainTextToken",
        ])->assertJsonStructure([
            'id',
            'guid',
            'name',
            'email',
            'domain',
        ]);
    }
}
```

Great. Now we have our API Sanctum endpoints tested against our LDAP integration!
