---
title: Importing Objects
description: LdapRecord-Laravel Overview
---

# Importing LDAP Objects

## Introduction

> If you are looking to import LDAP users into your application,
> view [this guide](/docs/laravel/v1/auth/importing) instead.

With LdapRecord-Laravel, you can easily import and synchronize LDAP objects into a database table
using a given Eloquent model. This is useful for importing groups, memberships and more.

## Getting Started

For this example, we will be importing LDAP groups into our applications database table `groups`.

Our groups table simply contains a `name` column, however to import LDAP objects into it,
we must add two extra database columns:

|  Column  | Reason                                                                                                                              |
| :------: | ----------------------------------------------------------------------------------------------------------------------------------- |
|  `guid`  | This is for storing your LDAP objects `objectguid`. It is needed for locating and synchronizing your LDAP object to the database.   |
| `domain` | This is for storing your LDAP objects connection name. It is needed for storing your configured LDAP connection name of the object. |

> For brevity, we will not be showing the creation of the `groups` database table migration.

## Creating the Migration

Generate a migration to add these columns onto our `groups` table:

```text
php artisan make:migration add_ldap_columns_to_groups_table
```

Then, we'll add the new required columns to the migration:

```php
class AddLdapColumnsToGroupsTable extends Migration
{
    /**
     * Run the migrations.
     */
    public function up()
    {
        Schema::table('groups', function (Blueprint $table) {
            $table->string('guid')->unique()->nullable();
            $table->string('domain')->nullable();
        });
    }

    /**
     * Reverse the migrations.
     */
    public function down()
    {
        Schema::table('groups', function (Blueprint $table) {
            $table->dropColumn(['guid', 'domain']);
        });
    }
}
```

After finishing setting up the above migration, make sure you run it:

```text
php artisan migrate
```

## Setting Up the Eloquent Model

For the importer to be able to properly interface with your Eloquent model, you must apply the
following trait and interface onto your Eloquent model you are using to perform the import.

|   Type    |                                         |
| :-------: | --------------------------------------- |
| Interface | `LdapRecord\Laravel\LdapImportable`     |
|   Trait   | `LdapRecord\Laravel\ImportableFromLdap` |

```php
// app/Group.php

namespace App;

use LdapRecord\Laravel\LdapImportable;
use LdapRecord\Laravel\ImportableFromLdap;

class Group extends Authenticatable implements LdapImportable
{
    use ImportableFromLdap;

    // ...
}
```

Your model is now ready for importing.

## Running the Import

### Defining Sync Attributes

Prior to running the import, you must define a configuration array. This configuration array must contain an
array of `sync_attributes`, that reference the database column (the key) and the LDAP attribute (the value):

```php
$config = [
    'sync_attributes' => [
        'name' => 'cn'
    ],
];
```

As with [importing LDAP users](/docs/laravel/v1/auth/importing), you may also use an [attribute handler](/docs/laravel/v1/auth/configuration/#attribute-handlers)
if you require extra logic when setting database values from the object.

```php
$config = ['sync_attributes' => \App\Ldap\AttributeHandler::class];
```

### Performing the Import

To perform the import, you must use the `LdapRecord\Laravel\LdapImporter` class.

This class accepts your Eloquent model class as the first parameter, and your configuration array into the second:

> **Important**: The importer will always return an **unsaved** Eloquent
> model. You must call `save()` after running the importer.

```php
use App\Group as EloquentGroup;
use LdapRecord\Laravel\LdapImporter;
use LdapRecord\Models\ActiveDirectory\Group as LdapGroup;

// Define the sync attributes.
$config = [
    'sync_attributes' => [
        'name' => 'cn'
    ],
];

// Create the importer.
$importer = new LdapImporter(EloquentGroup::class, $config);

// Import each group from the directory.
foreach (LdapGroup::get() as $group) {
    $importer->run($group)->save();
}
```

The above can easily be placed into a [scheduled job](https://laravel.com/docs/laravel/v1/scheduling)
if you'd prefer the import to be ran in the background of your application.
