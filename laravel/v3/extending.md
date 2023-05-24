---
title: Extending
description: Extending default classes and authentication functionality
---

# Extending

## Introduction

> **Important**: Extendability has been added in v2.3.0.

LdapRecord relies on Laravel's application container for making most of the
instances that control all of the major features of LdapRecord. This
allows you to modify and/or adjust core methods with ease.

Please be mindful of what you override, as certain events may not be fired
and things may not work as documented, depending on your own custom
implementation of features and what has been changed.

### Authentication

To override the class that attempts authentication against your LDAP server,
call the `LdapRecord::authenticateUsersUsing()` method, and provide
a class name or closure, which extends the built-in authenticator.

Create the authenticator:

```php
namespace App\Ldap;

use LdapRecord\Models\Model;
use LdapRecord\Laravel\LdapUserAuthenticator;

class UserAuthenticator extends LdapUserAuthenticator
{
    /**
     * Attempt authenticating against the LDAP domain.
     *
     * @param Model  $user
     * @param string $password
     *
     * @return bool
     */
    public function attempt(Model $user, $password)
    {
        // Attempt authentication...

        // $user->getConnection()->auth()->attempt($user->getDn(), $password)...
    }
}
```

Register the binding:

```php
// app/Providers/AppServiceProvider.php

use App\Ldap\UserAuthenticator;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Register application services.
     *
     * @return void
     */
    public function register()
    {
        LdapRecord::authenticateUsersUsing(UserAuthenticator::class);
    }
}
```

### User Repository

To override the class that queries your LDAP server for users during authentication
and importing, call the `LdapRecord::locateUsersUsing()` method, and provide
a class name or closure, which extends the built-in user repository.

Create the class:

```php
namespace App\Ldap;

use LdapRecord\Laravel\LdapUserRepository;

class UserRepository extends LdapUserRepository
{
    /**
     * Retrieve a user by the given credentials.
     *
     * @param array $credentials
     *
     * @return \LdapRecord\Models\Model|null
     */
    public function findByCredentials(array $credentials = [])
    {
        // Locate the user by their credentials...

        // $this->query()->where(['...'])
    }
}
```

Register the binding:

```php
// app/Providers/AppServiceProvider.php

use App\Ldap\UserRepository;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Register application services.
     *
     * @return void
     */
    public function register()
    {
        LdapRecord::locateUsersUsing(UserRepository::class);
    }
}
```

### User Synchronizer

To override the class that synchronizes your LDAP users during authentication and
importing, call the `LdapRecord::synchronizeUsersUsing()` method, and provide
a class name or closure, which extends the built-in user sychronizer.

Create the class:

```php
namespace App\Ldap;

use LdapRecord\Laravel\Import\UserSynchronizer;

class Synchronizer extends UserSynchronizer
{
    /**
     * Synchronize the Eloquent database model with the LDAP model.
     *
     * @param LdapModel     $object
     * @param EloquentModel $eloquent
     * @param array         $data
     *
     * @return EloquentModel
     */
    public function synchronize(LdapModel $object, EloquentModel $eloquent, array $data = [])
    {
        // Set attributes onto the users Eloquent model...

        // $eloquent->name = $object->getFirstAttribute('cn');
    }
}
```

Register the binding:

```php
// app/Providers/AppServiceProvider.php

use App\Ldap\Synchronizer;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Register application services.
     *
     * @return void
     */
    public function register()
    {
        LdapRecord::synchronizeUsersUsing(Synchronizer::class);
    }
}
```
