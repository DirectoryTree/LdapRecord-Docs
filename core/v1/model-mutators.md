---
title: Model Accessors & Mutators
description: Utilizing Accessors & Mutators in LdapRecord
---

# Models: Accessors & Mutators

## Introduction

Accessors and mutators allow you to modify attribute values when you
retrieve or set them on model instances. If you'd ever used
[Laravel accessors or mutators](https://laravel.com/docs/eloquent-mutators),
you'll feel right at home.

## Accessors & Mutators

### Defining An Accessor

For an example, lets say we are working with Active Directory and we
want to encode the `thumbnailPhoto` attribute whenever we retrieve it
from our `User` model.

To define an accessor for this attribute, we define a method named
`getThumbnailphotoAttribute()`:

```php
<?php

use LdapRecord\Models\Model;

class User extends Model
{
    public function getThumbnailphotoAttribute($value)
    {
        // Due to LDAP's multi-valued nature, all values will be
        // contained inside of an array. We will attempt to
        // retrieve the first one, or supply a default.
        $data = $value[0] ?? file_get_contents('images/default_photo.jpg');

        $image = base64_encode($data);

        $mime = 'image/jpeg';

        if (function_exists('finfo_open')) {
            $finfo = finfo_open();

            $mime = finfo_buffer($finfo, $data, FILEINFO_MIME_TYPE);

            return "data:$mime;base64,$image";
        }

        return "data:$mime;base64,$image";
    }
}
```

As you can see from the above, the attribute name we want to create
an accessor for, must be between `get` and `Attribute`.

> The casing of `get` and `Attribute` are very important.
> This casing difference is how LdapRecord detects accessor
> and mutator methods.

If your attribute contains a hyphen, use must use alternate casing
to indicate this. For example, lets create an accessor for the
`apple-user-homeurl` attribute:

```php
<?php

use LdapRecord\Models\Model;

class User extends Model
{
    public function getAppleUserHomeurlAttribute($value)
    {
        // Do something with its value.
        return $value;
    }
}
```

As you can see, alternate casing indicates to LdapRecord that
the attribute we are looking for contains hyphens.

### Defining A Mutator

A mutator does the opposite of an accessor. A mutator is a function
you define that accepts the value of the attribute you are setting
so you can transform it before it is set onto the model.

To define a mutator, we use the above accessor syntax with `set` instead of `get`.

For example, let's define a `unicodepwd` mutator that automatically
encodes a password by setting the attribute:

```php
<?php

use LdapRecord\Utilities;
use LdapRecord\Models\Model;

class User extends Model
{
    public function setUnicodepwdAttribute($password)
    {
        $this->attributes['unicodepwd'] = [Utilities::encodePassword($password)];
    }
}
```

Now once we set the attribute, it will automatically encode the
password we are setting on the `User` model:

```php
$user = new User();

$user->unicodepwd = 'secret';
```

### Date Mutators

By default, LdapRecord will convert the attributes `createtimestamp` and
`modifytimestamp` to instances of [Carbon](https://github.com/briannesbitt/Carbon).

> If you extend from `ActiveDirectory` models, the attributes
> `whenchanged` and `whencreated` will be converted instead.

When you define an attribute as a date, you can set its value to an
instance of `DateTime` / `Carbon` instance, a UNIX timestamp, or
a date string (`Y-m-d`). Upon saving your model, these will
be converted properly to be stored in your directory.

To define a mutator for an attribute that contains a timestamp,
we must set the `$dates` property on the model. However, since
LDAP directories have different timestamp formats for
some attributes, we must tell LdapRecord what kind
of format to use for proper conversion.

For example, let's define a date mutator for the `accountexpires`
attribute that exists on Active Directory. To do so, we must set
the `$dates` property to a key / value pair, where the key is
the attribute that contains the timestamp and the value is
the type of LDAP format to convert to and from:

```php
<?php

use LdapRecord\Models\Model;

class User extends Model
{
    protected $dates = [
        'accountexpires' => 'windows-int',
    ];
}
```

Now lets have our user's account expire at the same time tomorrow:

```php
$user = User::find('cn=John Doe,dc=acme,dc=org');

$user->accountexpires = new \DateTime('+1 day');

$user->save();
```

Once we've saved the model, the attribute will now automatically
be converted to a [Carbon](https://github.com/briannesbitt/Carbon)
instance so you can use any of Carbon's methods on the attribute:

```php
$user = User::find('cn=John Doe,dc=acme,dc=org');

if ($user->accountexpires->isPast()) {
    // The user account is expired.
}
```

#### Available Types

Currently, there are 3 built-in date mutator types. They are:

- `ldap`
- `windows`
- `windows-int`

#### LDAP Type

The `ldap` type is the most common format for LDAP timestamps -
outside of Active Directory. This format converts LDAP timestamps
in the format of `YYYYMMDDHHMMSST`. T is the time zone which
is usually 'Z' (Zulu Time Zone = UTC/GMT).

#### Windows Type

The `windows` type is similar to the `ldap` type, however it
differs slightly so it requires its own conversion type. Its
timestamp is in the format of `YYYYMMDDHHMMSS.0T`. T is the
time zone which is usually 'Z' (Zulu Time Zone = UTC/GMT).

#### Windows Integer Type

The `windows-int` type handles the 18-digit Active Directory timestamp
format, also named 'Windows NT time format', 'Win32 FILETIME or
SYSTEMTIME' or NTFS file time. An example of this would be
the `accountexpires` attribute that exists on users:

```text
132131246410000000
```

Which equals:

```text
Monday, September 16, 2019 4:24:01 PM
```

## Attribute Casting

Similarly with Laravel's Eloquent, the `$casts` property on your model provides
a convenient method of converting attributes to common data types. The `$casts`
property should be an array where the key is the name of the attribute being
cast and the value is the type you wish to cast the column to.

The supported cast types are:

- `integer`
- `real`
- `float`
- `double`
- `decimal:<digits>`
- `string`
- `boolean`
- `object`
- `array`
- `collection`
- `datetime:<ldap/windows/windows-int>`

To demonstrate attribute casting, let's cast the `msExchHideFromAddressList` Active Directory attribute,
which determines whether a user account is shown in the Global Address List in Outlook.

This attribute is stored as a string in Active Directory, with the value `TRUE` or `FALSE`.

```php
namespace App\Models\Ldap;

use LdapRecord\Models\ActiveDirectory\User as BaseUser;

class User extends BaseUser
{
    protected $casts = [
        'msExchHideFromAddressList' => 'boolean',
    ];
}
```

Then, we can utilize it when we retrieve users from our directory:

```php
$user = User::find('cn=John Doe,dc=local,dc=com');

if ($user->msExchHideFromAddressList) {
    // This user is being hidden from the Global Address list.
}
```
