---
title: Model Searching
description: A list of all available LdapRecord model query builder methods.
---

# Available Model Query Methods (API)

> **Important**: The model query builder extends the base query builder. 
> 
> All of [its methods](/docs/core/v2/searching-api) are available on model queries.

## Method Listing

#### `appliedScopes`

Get an array of the global scopes that were applied to the model query:

```php
$scopes = User::query()->appliedScopes();
```

#### `findByAnr`

Find the first matching model by [ambiguous naming resolution](https://social.technet.microsoft.com/wiki/contents/articles/22653.active-directory-ambiguous-name-resolution.aspx):

> **Important**: If your LDAP server does not support ANR, an equivalent filter will be generated automatically.

```php
use LdapRecord\Models\ActiveDirectory\User;

if ($user = User::findByAnr('John Doe')) {
    // Found user!
} else {
    // Not found.
}
```

#### `findByAnrOrFail`

Find the first matching model by [ambiguous naming resolution](https://social.technet.microsoft.com/wiki/contents/articles/22653.active-directory-ambiguous-name-resolution.aspx) **or fail**:

> **Important**: If your LDAP server does not support ANR, an equivalent filter will be generated automatically.

```php
use LdapRecord\Models\ActiveDirectory\User;

try {
    $user = User::findByAnrOrFail('John Doe');
} catch (\LdapRecord\Models\ModelNotFoundException $ex) {
    // Not found.
}
```

#### `findByGuid`

Find a model by its string GUID:

```php
use LdapRecord\Models\ActiveDirectory\User;

$guid = 'f53c7b48-e8d1-425f-a23a-d1b98d7abfe8';

if ($user = User::findByGuid($guid)) {
    // Found user!
} else {
    // Not found.
}
```

#### `findByGuidOrFail`

Find a model by its string GUID **or fail**:

```php
use LdapRecord\Models\ActiveDirectory\User;

$guid = 'f53c7b48-e8d1-425f-a23a-d1b98d7abfe8';

try {
    $user = User::findByGuidOrFail($guid);
} catch (\LdapRecord\Models\ModelNotFoundException $ex) {
    // Not found.
}
```

#### `findManyByAnr`

Find multiple models using [ambiguous naming resolution](https://social.technet.microsoft.com/wiki/contents/articles/22653.active-directory-ambiguous-name-resolution.aspx) **or fail**:.

```php
use LdapRecord\Models\ActiveDirectory\User;

$users = User::findManyByAnr(['Jane', 'John', 'Jack', 'Josh']);
```

#### `removedScopes`

Get an array of global scopes that were removed from the model query:

```php
use LdapRecord\Models\ActiveDirectory\User;

$scopes = User::query()->withoutGlobalScope(
    OnlyAccountants::class
)->removedScopes();
```

#### `withGlobalScope`

Apply a new global scope on the model query:

```php
use LdapRecord\Models\ActiveDirectory\User;

// Using a closure...
$users = User::withGlobalScope('accountants', function ($query) {
    // ...
})->get();

// Using a scope class...
$users = User::withGlobalScope(
    Accountants::class, new Accountants
)->get();
```

#### `withoutGlobalScope`

Query a model without a registered global scope:

```php
use LdapRecord\Models\ActiveDirectory\User;

$users = User::withoutGlobalScope(Accountants::class)->get();
```

#### `withoutGlobalScopes`

Query a model without all registered global scopes:

```php
use LdapRecord\Models\ActiveDirectory\User;

$users = User::withoutGlobalScopes()->get();
```
