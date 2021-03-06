---
title: User Management (Active Directory)
description: Managing users with LdapRecord.
---

# User Management (Active Directory)

## Creation

Let's walk through the basics of user creation for Active Directory. There
are some requirements you must know prior to creation:

| Requirement                                                                                               |
| --------------------------------------------------------------------------------------------------------- |
| You must connect to your server via TLS or SSL if you set the the users password (`unicodepwd`) attribute |
| You must connect to your server with an account that has permission to create users                       |
| You must set a common name (`cn`) for the user                                                            |
| You must set the `unicodePwd` attribute as a non-encoded string (more on this below)                      |
| To set the users `userAccountControl`, it must be set **after** the user has been created                 |

> Attributes that are set below can be cased in _any_ manor. They can be
> `UPPERCASED`, `lowercased`, `camelCased`, etc. Use whichever casing
> you prefer to be most readable in your application.

```php
<?php

use LdapRecord\Models\ActiveDirectory\User;

$user = (new User)->inside('ou=Users,dc=local,dc=com');

$user->cn = 'John Doe';
$user->unicodePwd = 'SecretPassword';
$user->samaccountname = 'jdoe';
$user->userPrincipalName = 'jdoe@acme.org';

$user->save();

// Enable the user.
$user->userAccountControl = 512;

try {
    $user->save();
} catch (\LdapRecord\LdapRecordException $e) {
    // Failed saving user.
}
```

> It is wise to encapsulate saving your user in a try / catch block, so if it
> fails you can determine if the cause of failure is due to your domains password policy.

## Password Management

### Setting Passwords

Using the included `LdapRecord\Models\ActiveDirectory\User` model, an attribute
[mutator](/docs/core/v1/model-mutators) has been added that assists in the setting
and changing of passwords on user objects. Feel free to take a
peek into the source code to see how it all works.

The password string you set on the users `unicodePwd` attribute is automatically encoded.
You do not need to encode it yourself. Doing so will cause an error or exception upon
saving the user.

Once you have set a password on a user object, this generates a modification
on the user model equal to a `LDAP_MODIFY_BATCH_REPLACE`:

```php
<?php

use LdapRecord\Models\ActiveDirectory\User;

$user = new User();

$user->unicodepwd = 'secret';

$modification = $user->getModifications()[0];

var_dump($modification);

// "attrib" => "unicodepwd"
// "modtype" => 3
// "values" => array:1 [
//    0 => ""\x00s\x00e\x00c\x00r\x00e\x00t\x00"\x00"
// ]
```

As you can see, a batch modification has been automatically generated for
the user. Upon calling `save()`, it will be sent to your LDAP server.

### Changing Passwords

To change a user's password, you must either:

1. Bind to your LDAP server with a user that has **permissions to reset passwords**
2. Or; **bind as the user** whose password you are trying to change.

There are some prerequisites you must know for changing passwords:

| Prerequisites                                                                                                        |
| -------------------------------------------------------------------------------------------------------------------- |
| You must provide the **correct user's old password**                                                                 |
| You must provide a new password that abides by your **password policy**, such as **history, complexity, and length** |
| You must set the `unicodepwd` attribute with an array containing **two** (2) values (old & new password)             |

Let's walk through an example:

> You must use a try / catch block upon saving. An `LdapRecord\LdapRecordException`
> will always be thrown when an incorrect old password has been given, or the new
> password does not abide by your password policy.

```php
<?php

use LdapRecord\Models\ActiveDirectory\User;

$user = User::find('cn=John Doe,ou=Users,dc=local,dc=com');

$user->unicodepwd = ['old-password', 'new-password'];

try {
    $user->save();

    // User password changed!
} catch (\LdapRecord\Exceptions\InsufficientAccessException $ex) {
    // The currently bound LDAP user does not
    // have permission to change passwords.
} catch (\LdapRecord\Exceptions\ConstraintException $ex) {
    // The users new password does not abide
    // by the domains password policy.
} catch (\LdapRecord\LdapRecordException $ex) {
    // Failed changing password. Get the last LDAP
    // error to determine the cause of failure.
    $error = $ex->getDetailedError();

    echo $error->getErrorCode();
    echo $error->getErrorMessage();
    echo $error->getDiagnosticMessage();
}
```

### Resetting Passwords

To reset a users password, you must be bound to your LDAP directory with a user whom has permission to do so on your directory.

You can perform a password reset by simply setting the users `unicodepwd` attribute as a string,
and then calling the `save()` method, similarly to how it is done during user creation:

```php
<?php

use LdapRecord\Models\ActiveDirectory\User;

$user = User::find('cn=John Doe,ou=Users,dc=local,dc=com');

$user->unicodepwd = 'new-password';

try {
    $user->save();

    // User password reset!
} catch (\LdapRecord\Exceptions\InsufficientAccessException $ex) {
    // The currently bound LDAP user does not
    // have permission to reset passwords.
} catch (\LdapRecord\Exceptions\ConstraintException $ex) {
    // The users new password does not abide
    // by the domains password policy.
} catch (\LdapRecord\LdapRecordException $ex) {
    // Failed resetting password. Get the last LDAP
    // error to determine the cause of failure.
    $error = $ex->getDetailedError();

    echo $error->getErrorCode();
    echo $error->getErrorMessage();
    echo $error->getDiagnosticMessage();
}
```

### Password Policy Errors

Active Directory will return diagnostic error codes when a password modification fails.

To determine the cause, you can check this diagnostic message to see if it contains any of the following codes:

| Code  | Meaning                                    |
| ----- | ------------------------------------------ |
| `525` | User not found                             |
| `52e` | Invalid credentials                        |
| `530` | Not permitted to logon at this time        |
| `531` | Not permitted to logon at this workstation |
| `532` | Password expired                           |
| `533` | Account disabled                           |
| `701` | Account expired                            |
| `773` | User must reset password                   |
| `775` | User account locked                        |

```php
<?php

use LdapRecord\Models\ActiveDirectory\User;

$user = User::find('cn=John Doe,ou=Users,dc=local,dc=com');

$user->unicodepwd = ['old-password', 'new-password'];

try {
    $user->save();

    // User password changed!
} catch (\LdapRecord\LdapRecordException $ex) {
    // Failed changing password. Get the last LDAP
    // error to determine the cause of failure.
    $error = $ex->getDetailedError();

    echo $error->getErrorCode(); // 49
    echo $error->getErrorMessage(); // 'Invalid credentials'
    echo $error->getDiagnosticMessage(); // '80090308: LdapErr: DSID-0C09042A, comment: AcceptSecurityContext error, data 52e, v3839'

    if (strpos($error->getDiagnosticMessage(), '52e')) {
        // This is an invalid credentials error.
    }
}
```

### Unlock Locked Out User Account

If a user has been locked out, set the `lockouttime` attribute to `0` (zero):

```php
$user = User::find('cn=John Doe,ou=Users,dc=local,dc=com');

$user->update(['lockouttime' => 0]);
```

### Extend User Password Expiration

Sometimes you may wish to extend a user's password expiration for the full duration of your domains password expiry time.

To do this, you must update the user's `pwdLastSet` time to `0`, then to `-1`:

```php
$user = User::find('cn=John Doe,ou=Users,dc=local,dc=com');

// Set password last set to 'Never':
$user->update(['pwdlastset' => 0]);

// Set password last set to the current date / time:
$user->update(['pwdlastset' => -1]);

// User password expiration successfully extended.
```

### User Must Change Password at Next Logon

To toggle the "_User Must Change Password at Next Logon_" checkbox that is
available in the Active Directory GUI - you must set the `pwdlastset`
attribute to one of the below values:

| Value | Meaning                                                                  |
| ----- | ------------------------------------------------------------------------ |
| `0`   | **Toggled on**. The user will be required to change their password.      |
| `-1`  | **Toggled off**. The user will not be required to change their password. |

> **Important**:
>
> - The `pwdlastset` attribute can only be modified by domain administrators
> - If toggled on, the Active Directory user **will not** pass LDAP authentication
>   until they visit a domain joined computer and update their password

```php
$user = User::find('cn=John Doe,ou=Users,dc=local,dc=com');

// The user must change their password on next login.
$user->pwdlastset = 0;

$user->save();
```

## User Account Control

A users `userAccountControl` attribute stores an integer value.

This integer value contains the sums of various integer flags. These flags control the
accessibility and behaviour of an Active Directory user account, such as account
disablement, password expiry, the ability to change passwords, and more.

For example, setting a users `userAccountControl` to `512` would mean that
the user account is a default account type that represents a typical user.
Setting it to `2`, would mean the account has been disabled.

Combining both to `514` (`512 + 2 = 514`) would mean the users
account is a typical user account, that is also disabled.

### Usage

You can manipulate a users `userAccountControl` manually by simply setting the
`userAccountControl` property on an existing user using the raw integer value,
or you can use the account control builder `LdapRecord\Models\Attributes\AccountControl`:

```php
<?php

use LdapRecord\Models\ActiveDirectory\User;
use LdapRecord\Models\Attributes\AccountControl;

$user = User::find('cn=John Doe,ou=Users,dc=local,dc=com');

// Setting the UAC value manually:
$user->userAccountControl = 512; // Normal, enabled account.

// Or, using the UAC builder:
$user->userAccountControl = (new AccountControl)->accountIsNormal();

$user->save();
```

Using the `AccountControl` builder, methods called will automatically sum the integer value.

For example, let's set an account control for a user with the following controls:

- The user account is normal
- The user account password does not expire
- The user account password cannot be changed

```php
$user = User::find('cn=John Doe,ou=Users,dc=local,dc=com');

$uac = new AccountControl();

$uac->accountIsNormal();
$uac->passwordDoesNotExpire();
$uac->passwordCannotBeChanged();

$user->userAccountControl = $uac;

$user->save();
```

The `AccountControl` builder also allows you to determine which flags are set.

This can be done with the `has` and `doesntHave` methods.

Create an `AccountControl` with the users `userAccountControl` value in the constructor:

```php
$user = User::find('cn=John Doe,ou=Users,dc=local,dc=com');

$uac = new AccountControl(
    $user->getFirstAttribute('userAccountControl')
);

if ($uac->has(AccountControl::LOCKOUT)) {
    // This account is locked out.
}

if ($uac->doesntHave(AccountControl::LOCKOUT)) {
    // The account is not locked out.
}
```

### Available Constants

The Account Control builder has constants for every possible value:

| Constant                                         | Value      |
| ------------------------------------------------ | ---------- |
| `AccountControl::SCRIPT`                         | `1`        |
| `AccountControl::ACCOUNTDISABLE`                 | `2`        |
| `AccountControl::HOMEDIR_REQUIRED`               | `8`        |
| `AccountControl::LOCKOUT`                        | `16`       |
| `AccountControl::PASSWD_NOTREQD`                 | `32`       |
| `AccountControl::PASSWD_CANT_CHANGE`             | `64`       |
| `AccountControl::ENCRYPTED_TEXT_PWD_ALLOWED`     | `128`      |
| `AccountControl::TEMP_DUPLICATE_ACCOUNT`         | `256`      |
| `AccountControl::NORMAL_ACCOUNT`                 | `512`      |
| `AccountControl::INTERDOMAIN_TRUST_ACCOUNT`      | `2048`     |
| `AccountControl::WORKSTATION_TRUST_ACCOUNT`      | `4096`     |
| `AccountControl::SERVER_TRUST_ACCOUNT`           | `8192`     |
| `AccountControl::DONT_EXPIRE_PASSWORD`           | `65536`    |
| `AccountControl::MNS_LOGON_ACCOUNT`              | `131072`   |
| `AccountControl::SMARTCARD_REQUIRED`             | `262144`   |
| `AccountControl::TRUSTED_FOR_DELEGATION`         | `524288`   |
| `AccountControl::NOT_DELEGATED`                  | `1048576`  |
| `AccountControl::USE_DES_KEY_ONLY`               | `2097152`  |
| `AccountControl::DONT_REQ_PREAUTH`               | `4194304`  |
| `AccountControl::PASSWORD_EXPIRED`               | `8388608`  |
| `AccountControl::TRUSTED_TO_AUTH_FOR_DELEGATION` | `16777216` |
| `AccountControl::PARTIAL_SECRETS_ACCOUNT`        | `67108864` |

### Available Methods

The Account Control builder has methods to apply every possible value:

| Method                                           | Constant Applied                                 |
| ------------------------------------------------ | ------------------------------------------------ |
| `AccountControl::runLoginScript()`               | `AccountControl::SCRIPT`                         |
| `AccountControl::accountIsDisabled()`            | `AccountControl::ACCOUNTDISABLE`                 |
| `AccountControl::homeFolderIsRequired()`         | `AccountControl::HOMEDIR_REQUIRED`               |
| `AccountControl::accountIsLocked()`              | `AccountControl::LOCKOUT`                        |
| `AccountControl::passwordIsNotRequired()`        | `AccountControl::PASSWD_NOTREQD`                 |
| `AccountControl::passwordCannotBeChanged()`      | `AccountControl::PASSWD_CANT_CHANGE`             |
| `AccountControl::allowEncryptedTextPassword()`   | `AccountControl::ENCRYPTED_TEXT_PWD_ALLOWED`     |
| `AccountControl::accountIsTemporary()`           | `AccountControl::TEMP_DUPLICATE_ACCOUNT`         |
| `AccountControl::accountIsNormal()`              | `AccountControl::NORMAL_ACCOUNT`                 |
| `AccountControl::accountIsForInterdomain()`      | `AccountControl::INTERDOMAIN_TRUST_ACCOUNT`      |
| `AccountControl::accountIsForWorkstation()`      | `AccountControl::WORKSTATION_TRUST_ACCOUNT`      |
| `AccountControl::accountIsForServer()`           | `AccountControl::SERVER_TRUST_ACCOUNT`           |
| `AccountControl::passwordDoesNotExpire()`        | `AccountControl::DONT_EXPIRE_PASSWORD`           |
| `AccountControl::accountIsMnsLogon()`            | `AccountControl::MNS_LOGON_ACCOUNT`              |
| `AccountControl::accountRequiresSmartCard()`     | `AccountControl::SMARTCARD_REQUIRED`             |
| `AccountControl::trustForDelegation()`           | `AccountControl::TRUSTED_FOR_DELEGATION`         |
| `AccountControl::doNotTrustForDelegation()`      | `AccountControl::NOT_DELEGATED`                  |
| `AccountControl::useDesKeyOnly()`                | `AccountControl::USE_DES_KEY_ONLY`               |
| `AccountControl::accountDoesNotRequirePreAuth()` | `AccountControl::DONT_REQ_PREAUTH`               |
| `AccountControl::passwordIsExpired()`            | `AccountControl::PASSWORD_EXPIRED`               |
| `AccountControl::trustToAuthForDelegation()`     | `AccountControl::TRUSTED_TO_AUTH_FOR_DELEGATION` |
| `AccountControl::accountIsReadOnly()`            | `AccountControl::PARTIAL_SECRETS_ACCOUNT`        |

## Group Management

If you are utilizing the included `LdapRecord\Models\ActiveDirectory\User` model, the
`groups()` relationship exists for easily removing / adding groups to users.

### Getting Groups

To get the groups that a user is a member of, call the `groups()` relationship method.
This will return the immediate groups that the user is a member of:

```php
<?php

use LdapRecord\Models\ActiveDirectory\User;

$user = User::find('cn=John Doe,ou=Users,dc=local,dc=com');

// Get immediate groups the user is apart of:
$groups = $user->groups()->get();

foreach ($groups as $group) {
    echo $group->getName();
}
```

You may also want to retrieve groups that are _members_ of groups that the user is apart of.
This is called a [recursive relationship query](/docs/core/v1/model-relationships/#recursive-queries).

To retrieve groups of groups, call the `recursive()` method following the `groups()` relation call:

```php
<?php

use LdapRecord\Models\ActiveDirectory\User;

$user = User::find('cn=John Doe,ou=Users,dc=local,dc=com');

// Get nested groups the user is apart of:
$groups = $user->groups()->recursive()->get();

foreach ($groups as $group) {
    echo $group->getName();
}
```

### Filtering Groups

Relations in LdapRecord act as query builders, so you can chain query methods on the `groups()` relation itself:

```php
<?php

use LdapRecord\Models\ActiveDirectory\User;

$user = User::find('cn=John Doe,ou=Users,dc=local,dc=com');

// Get all groups the user is apart of that contain 'Accounting':
$groups = $user->groups()->whereContains('cn', 'Accounting')->get();

// Get all groups the user is apart of that are members of the 'Office' group:
$groups = $user->groups()->whereMemberOf('cn=Office,ou=Groups,dc=local,dc=com')->get();
```

### Checking Existence

To check if a user is a member of _any_ group, call the `exists()` method on the `groups()` relationship:

```php
<?php

use LdapRecord\Models\ActiveDirectory\User;

$user = User::find('cn=John Doe,ou=Users,dc=local,dc=com');

if ($user->groups()->exists()) {
    // The user is a member of at least one group.
}
```

To check if a user is an immediate member of a specific group, pass a model into the `exists()` method:

```php
<?php

use LdapRecord\Models\ActiveDirectory\User;
use LdapRecord\Models\ActiveDirectory\Group;

$group = Group::find('cn=Accounting,dc=local,dc=com');

$user = User::find('cn=John Doe,ou=Users,dc=local,dc=com');

if ($user->groups()->exists($group)) {
    // The user is an immediate member of the 'Accounting' group.
}
```

To check if a user is a member of a group that could be nested in a sub-group, call
the `recursive()` method before calling `exists()`:

```php
<?php

use LdapRecord\Models\ActiveDirectory\User;
use LdapRecord\Models\ActiveDirectory\Group;

$group = Group::find('cn=Accounting,dc=local,dc=com');

$user = User::find('cn=John Doe,ou=Users,dc=local,dc=com');

if ($user->groups()->recursive()->exists($group)) {
    // The user is a member of the 'Accounting' group.
}
```

### Adding Groups

To add groups to a user, call the `groups()` relationship method, then `attach()`:

```php
<?php

use LdapRecord\Models\ActiveDirectory\User;
use LdapRecord\Models\ActiveDirectory\Group;

$group = Group::findOrFail('cn=Accounting,ou=Groups,dc=local,dc=com');

$user = User::find('cn=John Doe,ou=Users,dc=local,dc=com');

if ($user->groups()->attach($group)) {
    // Successfully added the group to the user.
}
```

### Removing Groups

To remove groups on user, call the `groups()` relationship method, then `detach()`:

```php
<?php

use LdapRecord\Models\ActiveDirectory\User;
use LdapRecord\Models\ActiveDirectory\Group;

$group = Group::findOrFail('cn=Accounting,ou=Groups,dc=local,dc=com');

$user = User::find('cn=John Doe,ou=Users,dc=local,dc=com');

if ($user->groups()->detach($group)) {
    // Successfully removed the group from the user.
}
```

> The `detach()` method will return `true` if the user is already not apart
> of the given group. This does not indicate that the user was previously a member.

You may want to locate groups on the user prior removal to ensure they are a member:

```php
<?php

use LdapRecord\Models\ActiveDirectory\User;

$user = User::find('cn=John Doe,ou=Users,dc=local,dc=com');

$group = $user->groups()->first();

if ($group && $user->groups()->detach($group)) {
    // Successfully removed the first group from the user.
}
```
