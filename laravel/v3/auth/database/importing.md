---
title: Importing Users
description: Running the import command
---

# Importing LDAP Users

## Introduction

LdapRecord-Laravel allows you to import users from your LDAP directories into your local database.
This is done by executing the `php artisan ldap:import` command and is only available to LDAP
authentication providers you configure with [database synchronization](/docs/laravel/v3/auth/database).

As it is with signing users into your application, the Eloquent database model you specify in your
`config/auth.php` file is used for the creation and retrieval of users in your database.

## Attribute Synchronization

The `sync_attributes` you define inside your `config/auth.php` file for your provider will be used
for importing and synchronizing users.

Be sure to look at the [documentation](/docs/laravel/v3/auth/database/configuration#database-sync-attributes)
to get a further understanding on what is possible with this option.

## Syncing Existing Records

The `sync_existing` array you define inside your `config/auth.php` will be used to synchronize existing database records with your LDAP users.

Be sure to look at the [documentation](/docs/laravel/v3/auth/database/configuration#database-sync-existing)
to get a further understanding on what is possible with this option.

## Password Synchronization

The `sync_passwords` option you define inside your `config/auth.php` file is used when
importing and synchronizing users. However, there are some caveats you must be aware of:

- **Passwords cannot be retrieved from users who are being imported from your LDAP server.**
  <br/><br/>This would be a major security risk if this were possible. If a password is already
  set for the user being imported, it will be left untouched. This is to retain a
  possible synchronized password that was set upon login.<br/><br/>
- **Passwords will always be set to a hashed 16 character string if not already present.**
  <br/><br/>If the user being imported does not have a password, their password will be set to a
  hashed 16 character random string using `Str::random`.<br/><br/>
- **Passwords will not be set** if you have defined `false` for `password_column`.

## Running the command

To run the command you must insert the `provider` name that you have set up for LDAP database synchronization
inside your `config/auth.php` file. Let's walk through an example.

In our application we have a configured authentication provider named `ldap`:

```php
'providers' => [
    // ...

    'ldap' => [
        // ...
        'database' => [
            // ...
        ],
    ],
],
```

We will then insert the providers name into our import command and execute it:

```bash
php artisan ldap:import users
```

You will then be asked after a successful search in your directory:

```text
Found 2 user(s).

Would you like to display the user(s) to be imported / synchronized? (yes/no) [no]:
> y
```

A table will then be shown, so you can confirm the import of the located users:

```text
+-------------+-------------------+---------------------+
| Name        | Account Name      | UPN                 |
+-------------+-------------------+---------------------+
| John Doe    | johndoe           | johndoe@local.com   |
| Jane Doe    | janedoe           | janedoe@local.com   |
+-------------+-------------------+---------------------+
```

Then, you will be asked to import the users shown and the import will begin:

```text
 Would you like these users to be imported / synchronized? (yes/no) [no]:
 > y

  2/2 [============================] 100%

Successfully imported / synchronized 2 user(s).
```

## Scheduling the command

To run the import as a scheduled job, place the following in your `app/Console/Kernel.php` in the command scheduler:

```php
protected function schedule(Schedule $schedule)
{
    // Import LDAP users hourly.
    $schedule->command('ldap:import users', [
        '--no-interaction',
        '--restore',
        '--delete',
        '--filter' => '(objectclass=user)',
    ])->hourly();
}
```

The above scheduled import command will:

- Run without interaction and import new users as well as synchronize already imported users
- Restore user models who have been re-activated in your LDAP directory (if you're using [Eloquent Soft Deletes](https://laravel.com/docs/eloquent#soft-deleting))
- Soft-Delete user models who have been deactived in your LDAP directory (if you're using [Eloquent Soft Deletes](https://laravel.com/docs/eloquent#soft-deleting))
- Only import objects that have an `objectclass` containing `user`

> It's recommended to use [model query scopes](/docs/core/v3/models#query-scopes) instead of the `--filter`
> option on your configured authentication LdapRecord model so LDAP users signing in to your
> application are applied the same search filter.

## Programmatically Executing

You can call the `ldap:import` command using Laravel's [Artisan](https://laravel.com/docs/artisan#programmatically-executing-commands)
facade to programmatically execute the import inside your application wherever you'd like:

```php
Artisan::call('ldap:import', ['provider' => 'ldap', '--no-interaction']);
```

To use other arguments and options, include them as array values:

```php
Artisan::call('ldap:import', [
    'provider' => 'ldap',
    'user' => 'sbauman',
    '--no-interaction',
    '--restore' => true,
    '--delete' => true,
    '--delete-missing' => true,
    '--filter' => '(cn=John Doe)',
    '--scopes' => 'App\Ldap\Scopes\OnlyAdmins',
    '--attributes' => 'cn,mail,samaccountname',
]);
```

## Events

When executing the `ldap:import` command, LdapRecord-Laravel will fire various events that you may register listeners on:

> **Important**: Each event listed below has the parent namespace of `LdapRecord\Laravel\Events\Import\`.

| Event            | Fired                                                               | Occurrence                                                                                            |
| ---------------- | ------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------- |
| `Completed`      | When an import has fully completed.                                 | Once per `ldap:import` execution.                                                                     |
| `Deleted`        | When an import has soft-deleted a user.                             | Each time a user is soft-deleted during an `ldap:import` execution.                                   |
| `DeletedMissing` | When an import has soft-deleted missing users.                      | Once per `ldap:import` execution.                                                                     |
| `Imported`       | When a user has been imported.                                      | Each time a user is imported via `ldap:import` execution, or authentication.                          |
| `ImportFailed`   | When an exception occurs during import or synchronization.          | Each time a user fails to be synchronized or imported via `ldap:import` execution, or authentication. |
| `Importing`      | When a non-existent user is being imported.                         | Each time a non-existent user is imported via `ldap:import` execution, or authentication.             |
| `Restored`       | When a previously soft-deleted user is being restored (un-deleted). | Each time a soft-deleted user is restored via an `ldap:import` execution.                             |
| `Saved`          | When a user has been saved after import or synchronization.         | Each time a user is saved via `ldap:import` execution, or authentication.                             |
| `Started`        | When an import has been started.                                    | Once per `ldap:import` execution.                                                                     |
| `Synchronized`   | When a user has been synchronized with any defined sync attributes. | Each time a user is synchronized via an `ldap:import` execution, or authentication.                   |
| `Synchronizing`  | When a user is beginning to be synchronized.                        | Each time a user is synchronizing via an `ldap:import` execution, or authentication.                  |

## Command Arguments

### Provider

To execute the import command, you **must** supply an [authentication provider](/docs/laravel/v3/auth/database/configuration)
name. This will retrieve the users from your configured LdapRecord model, and import them using your configured Eloquent model.

For example, if you have kept the default `users` authentication provider name in your `config/auth.php` file, then you would execute:

```php
php artisan ldap:import users
```

### User

To import or synchronize a single user, insert one of their attributes (such as `mail`, `samaccountname`, `cn`)
and LdapRecord will try to locate the user for you using Ambiguous Name Resolution. If your LDAP server
does not support ANR, an equivalent query will be created automatically.

This argument is completely optional.

> **Important**: Do not use the `--delete-missing` option with this argument.
> Otherwise, other LDAP users that have been imported will be soft-deleted
> (if configured & enabled on your Eloquent model).

```text
php artisan ldap:import users jdoe@email.com

Found user 'John Doe'.

Would you like to display the user(s) to be imported / synchronized? (yes/no) [no]:
> y
```

## Command Options

### Chunk

The `--chunk` option allows you to import users by chunk.

The option takes a number that indicates how many users per-chunk you would like to import.

Use this option if you are running out of memory during large imports.

```text
php artisan ldap:import users --chunk 500
```

### Filter

The `--filter` option allows you to apply a raw filter to further narrow down the users who are imported:

> **Important**: If your filter contains commas, or other types of "escape" level LDAP search filter characters,
> you **must** escape the value with a backslash (`\`) before passing it into the search string. More on this below.

```text
php artisan ldap:import users --filter "(cn=John Doe)"
```

#### Escaping

In some cases, you may need to pass commas or other escape level characters into the search filter.

To do so, add a backslash (`\`) **before** the character to escape it properly:

```text
php artisan ldap:import users --filter "(cn=Doe\, John)"
```

If this is not done, you will receive a `Bad search filter` exception during import.

### Scopes

The `--scopes` option allows you to specify model query scopes that will
apply to the underlying LdapRecord query builder instance when
searching for users to import with your configured model.

This allows you to not have to extend the built-in models to apply global
scopes, as well as having scopes that only apply during import.

> **Note**: Since these scopes only apply during import, you may want to configure the
> [OnlyImported](/docs/laravel/v3/auth/restricting-login/#using-only-manually-imported-users)
> authentication rule so that only users who have been imported successfully
> with your configured scopes, can log into your application.

```text
php artisan ldap:import users --scopes "App\Ldap\Scopes\OnlyAdministrators"
```

You may also provide several scopes, via comma separation:

```text
php artisan ldap:import users --scopes "App\Ldap\Scopes\OnlyUsers,App\Ldap\Scopes\ExcludeComputerObjects"
```

### Attributes

The `--attributes` option allows you to specify the attributes that should be returned from your LDAP server.

This option is great for reducing memory usage for large imports, since all attributes will be returned from your LDAP server otherwise.

> **Important**: To use this option, you **must** comma separate each attribute in the
> command and include the attributes you have configured in your
> [authentication provider](/docs/laravel/v3/auth/configuration/#database-sync-attributes).

```text
php artisan ldap:import users --attributes "cn,mail,sn,givenname,samaccountname"
```

### Delete

> This option is only available on Active Directory models.

The `--delete` option allows you to soft-delete deactivated LDAP users. No users
will be deleted if your `User` Eloquent model does not have soft-deletes enabled.

```text
php artisan ldap:import users --delete
```

### Delete Missing

> This option is available for **all LDAP directories**.

The `--delete-missing` option allows you to soft-delete all LDAP users that
were missing from the import. This is useful when a user has been deleted
in your LDAP server, and therefore should be soft-deleted inside your
application, since they will not be returned in search results.

This option has been designed to have the utmost safety of user data in mind.
Here are some paramount things to understand with this option:

**No users will be deleted if soft-deletes are not enabled on your `User` eloquent model.**

Deletion will not occur. You must set up [Soft Deletes](https://laravel.com/docs/eloquent#soft-deleting)
on your `User` eloquent model.

**If no users have been successfully imported, no users will be soft-deleted.**

If an executed import does not successfully import any users, no users will be soft-deleted.

**Only users that belong to the domain you are importing will be soft-deleted.**

This means, all other users will be left untouched, such as local database
users that were not imported from an LDAP server, as well as users
that were imported from another domain.

**Soft-deleted users are reported in the log.**

When users are soft-deleted, a log entry will be created for each one:

```text
User with [id = 2] has been soft-deleted due to being missing from LDAP import.
User with [id = 5] has been soft-deleted due to being missing from LDAP import.
```

#### The DeletedMissing Event

A `DeletedMissing` event is fired in the event of any users being soft-deleted.

You may listen for this event and access the IDs of the deleted users, as well as the Eloquent model
that was used to perform the deletion, and the LdapRecord model that was used to perform the import.

Here is an example listener that accesses this event and its properties:

```php
// app/Listeners/UsersDeletedFromImport.php

namespace App\Listeners;

use LdapRecord\Laravel\Events\Import\DeletedMissing;

class UsersDeletedFromImport
{
    public function handle(DeletedMissing $event)
    {
        // \Illuminate\Support\Collection
        $event->deleted;

        // \LdapRecord\Models\ActiveDirectory\User
        $event->ldapModel;

        // \Illuminate\Database\Eloquent\Model
        $event->eloquentModel;
    }
}
```

### Restore

> This option is only available on Active Directory models.

The `--restore` option allows you to restore soft-deleted re-activated LDAP users.

```text
php artisan ldap:import users --restore
```

> Typically, the `--restore` and `--delete` options would be used together to
> allow full synchronization of user disablements and restoration.

### Min Users

The `--min-users` option allows you to specify a minimum number of users that should be returned
from your LDAP server before performing an import. This is useful in circumstances where you're 
using the `--delete-missing` flag, and you want to ensure that a possible query or replication 
issue in your environment does not perform a drastic soft-deletion to users not returned.

```text
php artisan ldap:import users --min-users=1000 --delete-missing
```

### No Logging

The `--no-log` option allows you to disable logging during the command.

```text
php artisan ldap:import users --no-log
```

By default, this is enabled, regardless if `logging` is disabled in your `config/ldap.php` file.

### No Interaction

To run the import command via a schedule, use the `--no-interaction` flag:

```text
php artisan ldap:import users --no-interaction
```

Users will be imported automatically with no prompts.

You can also call the command from the Laravel Scheduler, or other commands:

```php
// Importing one user
$schedule->command('ldap:import users sbauman', ['--no-interaction'])
            ->everyMinute();

// Importing all users
$schedule->command('ldap:import users', ['--no-interaction'])
            ->everyMinute();

// Importing users with a filter
$dn = 'CN=Accounting,OU=SecurityGroups,DC=local,DC=com';

$filter = sprintf('(memberof:1.2.840.113556.1.4.1941:=%s)', $dn);

$schedule->command('ldap:import users', ['--no-interaction', '--filter' => $filter])
    ->everyMinute();
```

### Additional Tips

- Users who already exist inside your database will be updated with your configured providers `sync_attributes`.
- Users **will never be force deleted** from the import command. You will need to delete users manually
  through your Eloquent model
- If you have a password mutator (setter) on your `User` Eloquent model, it will not override it.
  This allows you to hash the random 16 character passwords in your own way.
- Imported (new) users will be reported in your log files:

```text
[2020-01-29 14:51:51] local.INFO: Imported user johndoe
```

- Users that fail to be imported are also reported in your log files, alongside the message of the exception that caused the failure:

```text
[2020-01-29 14:51:51] local.ERROR: Unable to import user janedoe. SQLSTATE[23000]: Integrity constraint violation: 1048
```
