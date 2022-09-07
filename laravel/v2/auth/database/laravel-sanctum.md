---
title: Using LdapRecord-Laravel with Laravel Sanctum
description: Setting up LDAP authentication with Laravel Sanctum
---

> **Important**: Before getting started, please complete the below guides:
>
> - [Installation Guide](/docs/laravel/v2/auth/database/installation)
> - [Configuration Guide](/docs/laravel/v2/auth/database/configuration)
> - [Sanctum Installation Guide](https://laravel.com/docs/sanctum#installation)

## Preparing Your User Eloquent Model

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

## Issuing API Tokens

Now that your application is configured for [database authentication](https://ldaprecord.com/docs/laravel/v2/auth/database),
we will need an API endpoint to start issuing new user API tokens.

To do this, we will take Sanctum's [default suggested endpoint for issuing tokens](https://laravel.com/docs/9.x/sanctum#issuing-mobile-api-tokens), and then tweak it a little bit.

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

    $credentials = [
        'mail' => $request->email,
        'password' => $request->password,
    ];

    if (Auth::validate($credentials)) {
        return ['token' => Auth::getLastAttempted()->createToken($request->device_name)->plainTextToken];
    }

    throw ValidationException::withMessages([
        'email' => ['The provided credentials are incorrect.'],
    ]);
});
```

This is all that is needed to generate new API tokens, and to start using those tokens to authenticate against your server with.

Let's make sure these endpoints work with some tests.

## Testing

### Tinkerwell Testing

To test your Sanctum endpoint with [Tinkerwell](https://tinkerwell.app), serve your Laravel application by running the below command:

```bash
php artisan serve
```

Then, send a post request to `api/sanctum/token`:

```php
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

> **Important**: If you are using SQLite to test, remember to install `doctrin/dbal` before getting started, as
> mentioned in the [test guide](https://ldaprecord.com/docs/laravel/v2/auth/testing/#getting-started):
>
> ```bash
> composer require doctrine/dbal --dev
> ```

To begin, let's create a sanctum test to make sure both our API endpoints are working:

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

We'll need to `use` the `DatabaseMigrations` trait to run our migrations:

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
        // Setup the directory emulator to create a fake, existing LDAP user:
        $fake = DirectoryEmulator::setup();

        // Remember to supply the "sync_attributes" you have assigned inside
        // of your config/auth.php. You will receive an exception if any
        // non-nullable fields are not present on the LDAP user:
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
        ])->assertJsonValidationErrors(['email' => 'The provided credentials are incorrect.']);

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
