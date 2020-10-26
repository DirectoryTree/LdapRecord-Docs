---
title: Authentication Using Laravel UI
description: Setting up LDAP authenticating using Laravel UI
extends: _layouts.laravel-documentation
section: content
---

# Laravel UI

- [Introduction](#introduction)
- [Login Controller](#login-controller)
- [Using Usernames](#using-usernames)
- [Fallback Authentication](#fallback-auth)
- [Eloquent Model Binding](#model-binding)
- [Displaying LDAP Error Messages](#displaying-ldap-error-messages)

## Introduction {#introduction}

Now that we have updated our default authentication guard to use our new `ldap` provider, we will jump into
the default `LoginController` that is included with the [Laravel UI package](https://laravel.com/docs/authentication#introduction).

> Laravel UI is now deprecated (since Laravel 8.0) in favour of Laravel Jetstream.

## Login Controller {#login-controller}

For this example application, we will authenticate our LDAP users with their email address using the LDAP attribute `mail`.

For LdapRecord to properly locate the user in your directory during sign in,
we will override the `credentials` method in the `LoginController`:

```php
// app/Http/Controllers/Auth/LoginController.php

use Illuminate\Http\Request;

protected function credentials(Request $request)
{
    return [
        'mail' => $request->email,
        'password' => $request->password,
    ];
}
```

As you can see above, we set the `mail` key which is passed to the LdapRecord authentication provider.

A search query will be executed on your directory for a user that contains the `mail` attribute equal
to the entered `email` that the user has submitted on your login form. The `password`
key will not be used in the search.

If a user cannot be located in your directory, or they fail authentication, they will be redirected to the
login page normally with the "Invalid credentials" error message.

> You may also add extra key => value pairs in the `credentials` array to further scope the
> LDAP query. The `password` key is automatically ignored by LdapRecord.

## Using Usernames {#using-usernames}

In corporate environments, users are often used to signing into their computers with their username.
You can certainly keep this flow easy for them - we just need to change a couple things.

First, you will need to change the `email` column in the database migration that creates your `users`
table to `username`, as this represents what it will now contain:

```php
Schema::create('users', function (Blueprint $table) {
    // ...

    // Before...
    $table->string('email')->unique(); 
    
    // After...
    $table->string('username')->unique(); 
});
```

> Make sure you run your migrations using `php artisan migrate`.

Once we've changed the name of the column, we'll jump into the `config/auth.php` configuration and modify 
our LDAP user providers `sync_attributes` to synchronize this changed column.

In this example, we will use the users `sAMAccountName` as their username
which is common in Active Directory environments:

```php
// config/auth.php

'providers' => [
    // ...

    'ldap' => [
        // ...
        
        'database' => [
            // ...

            'sync_attributes' => [
                'name' => 'cn',
                'username' => 'samaccountname',
            ],
        ],
    ],
],
```

Now, since we have changed the way our users sign in to our application from the default `email` field,
we need to modify our HTML login form to reflect this. Let's jump into our `auth/login.blade.php`:

```html
<!-- resources/views/auth/login.blade.php -->

<!-- Before... -->
<input id="email" type="email" class="form-control @error('email') is-invalid @enderror" name="email" value="{{ old('email') }}" required autocomplete="email" autofocus>

<!-- After... -->
<input id="username" type="text" class="form-control @error('username') is-invalid @enderror" name="username" value="{{ old('username') }}" required autocomplete="username" autofocus>
```

After changing the HTML input, we now must modify our `LoginController` to use this new field.
We do this by overriding the `username` method, and updating our `credentials` method:

```php
// app/Http/Controllers/Auth/LoginController.php

use Illuminate\Http\Request;

public function username()
{
    return 'username';
}

protected function credentials(Request $request)
{
    return [
        'samaccountname' => $request->username,
        'password' => $request->password,
    ];
}
```

If we are using Fortify package, we must modify `Fortify::authenticateUsing` method in `App\Providers\AuthServiceProvider::boot`.

```php
// app\Providers\AuthServiceProvider.php

public function boot()
{
    Fortify::authenticateUsing(function ($request) {
        $validated = Auth::validate([
            // Before...
            'mail' => $request->email,
    
            // After...
            'samaccountname' => $request->username,
            'password' => $request->password
        ]);
    
        return $validated ? Auth::getLastAttempted() : null;
    });
}
```

You can now sign in to your application using usernames instead of email addresses.

## Fallback Authentication {#fallback-auth}

Database fallback allows the authentication of local database users if **LDAP
connectivity is not present**, or **an LDAP user cannot be found**.

To enable this feature, you must define a `fallback` array inside of the credentials
you return from the `credentials()` method inside of your `LoginController`:

```php
protected function credentials(Request $request)
{
    return [
        'mail' => $request->email,
        'password' => $request->password,
        'fallback' => [
            'email' => $request->email,
            'password' => $request->password,
        ],
    ];
}
```

For example, given the following `users` database table:

id | name | email | password | guid | domain |
--- | --- | --- | --- |
1 | Steve Bauman | sbauman@outlook.com | ... | `null` | `null` |

If a user attempts to login with the above email address and this user does
not exist inside of your LDAP directory, then standard Eloquent authentication
will be performed instead.

This feature is ideal for environments where:

- LDAP server connectivity may be intermittent
- Or; You have regular users registering normally in your application

> If you would like your LDAP users to be able to sign in to your application
> when LDAP connectivity fails or is not present, you must enable the
> [sync passwords](#database-password-sync) option, so your LDAP
> users can sign in using their last used password. 
> <br/><br/>
> If an LDAP users password has not been synchronized, they will not be able to sign in.

## Eloquent Model Binding {#model-binding}

If you are using [database synchronization](/docs/laravel/auth#database), model binding allows
you to access the **currently authenticated user's** LdapRecord model from their Eloquent
model. This grants you access to their LDAP data whenever you need it.

To begin, insert the `LdapRecord\Laravel\Auth\HasLdapUser` trait onto your User model:

```php
// app/User.php

// ...

use LdapRecord\Laravel\Auth\HasLdapUser;
use LdapRecord\Laravel\Auth\LdapAuthenticatable;
use LdapRecord\Laravel\Auth\AuthenticatesWithLdap;

class User extends Authenticatable implements LdapAuthenticatable
{
    use Notifiable, AuthenticatesWithLdap, HasLdapUser;

    // ...
}
```

Now, after an LDAP user logs into your application, their LdapRecord model will be
available on their Eloquent model via the `ldap` property:

> If their LDAP model cannot be located, the returned will be `null`.

```php
// Instance of App\User
$user = Auth::user();

// Instance of App\Ldap\User
$user->ldap;

// Get LDAP user attributes
echo $user->ldap->getFirstAttribute('cn');

// Get LDAP user relationships:
$groups = $user->ldap->groups()->get();
```

> This property uses deferred loading -- which means that the users LDAP model only
> gets requested from your server when you attempt to access it. This prevents
> loading the model unnecessarily when it is not needed in your application.

## Displaying LDAP Error Messages {#displaying-ldap-error-messages}

When a user fails LDAP authentication due to their password / account expiring, account
lockout, or their password requiring to be changed, specific error codes will be sent
back from your server. LdapRecord can interpret these for you and display
helpful error messages to users upon failing authentication.

To add this functionality, you must add the following trait to your `LoginController`:

```text
LdapRecord\Laravel\Auth\ListensForLdapBindFailure
```

Example:

```php
// app/Http/Controllers/Auth/LoginController.php

// ...

use LdapRecord\Laravel\Auth\ListensForLdapBindFailure;

class LoginController extends Controller
{
    use AuthenticatesUsers, ListensForLdapBindFailure;

    // ...
```

**However, this feature will only register automatically if your `LoginController` resides in the default
`App\Http\Controllers\Auth` namespace**. If you have changed the location of your `LoginController`,
you must modify the constructor and add the following method call to register the LDAP listener:

```php
// app/Http/Controllers/Auth/LoginController.php

// ...

use LdapRecord\Laravel\Auth\ListensForLdapBindFailure;

class LoginController extends Controller
{
    use AuthenticatesUsers, ListensForLdapBindFailure;

    public function __construct()
    {
        $this->middleware('guest')->except('logout');
    
        $this->listenForLdapBindFailure();
    }

    // ...
}
```

### Altering the Response

By default, when an LDAP bind failure occurs, a `ValidationException` will be thrown which will
redirect users to your login page and display the error. If you would like to modify this
behaviour, you will need to override the method `handleLdapBindError`.

This method will include the error message as the first parameter and the error code as the second.

This is useful for checking for specific Active Directory response codes and returning a response:

```php
// app/Http/Controllers/Auth/LoginController.php

// ...

class LoginController extends Controller
{
    // ...

    use ListensForLdapBindFailure {
        handleLdapBindError as baseHandleLdapBindError;
    }
    
    protected function handleLdapBindError($message, $code = null)
    {
        if ($code == '773') {
            // The users password has expired. Redirect them.
            abort(redirect('/password-reset'));
        }
    
        $this->baseHandleLdapBindError($message, $code);
    }
   
    // ...
}
```

> Refer to the [Password Policy Errors](/docs/active-directory/users/#password-policy-errors)
> documentation to see what each code means.

### Changing the Error Messages

If you need to modify the translations of these error messages, create a new translation
file named `errors.php` in your `resources` directory at the following path:

> The `vendor` directory (and each sub-directory) will have to be created manually.

<div class="files">
    <div class="ellipsis"></div>

    <div class="folder folder--open">
        resources

        <div class="ellipsis"></div>
        
        <div class="folder folder--open">
            lang

            <div class="ellipsis"></div>

            <div class="folder folder--open">
                vendor

                <div class="folder folder--open">
                    ldap

                    <div class="folder folder--open">
                        en

                        <div class="file">errors.php</div>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <div class="ellipsis"></div>
</div>

Then, paste in the following translations in the file and modify where necessary:

```php
<?php

return [
    'user_not_found' => 'User not found.',
    'user_not_permitted_at_this_time' => 'Not permitted to logon at this time.',
    'user_not_permitted_to_login' => 'Not permitted to logon at this workstation.',
    'password_expired' => 'Your password has expired.',
    'account_disabled' => 'Your account is disabled.',
    'account_expired' => 'Your account has expired.',
    'user_must_reset_password' => 'You must reset your password before logging in.',
    'user_account_locked' => 'Your account is locked.',
];
```
