---
title: Helpers
description: Various helper classes and utilities
---

# Helpers

LdapRecord provides some helper classes and utility functions you may find useful in your application.

## Distinguished Names

```php
use LdapRecord\Models\Attributes\DistinguishedName;
```

The `DistinguishedName` utility class allows you to parse
Distinguished Name's, and perform various operations.

> **Important**: All comparison based operations are **case insensitive**.

### `make`

Make a new Distinguished Name instance:

```php
$dn = DistinguishedName::make('cn=John Doe,dc=local,dc=com');
```

### `build`

Make a new [Distinguished Name Builder](#distinguished-name-building) instance:

```php
// Pre-populate a builder:
$builder = DistinguishedName::build('cn=John Doe,dc=local,dc=com');

// Start from scratch:
$builder = DistinguishedName::build();
```

### `of`

Alias of the `build` method:

```php
// Pre-populate a builder:
$builder = DistinguishedName::of('cn=John Doe,dc=local,dc=com');

// Start from scratch:
$builder = DistinguishedName::of();
```

### `isValid`

Determine if the given string is a valid Distinguished Name:

```php
// true
DistinguishedName::isValid('cn=John Doe,dc=local,dc=com');

// true
DistinguishedName::isValid('cn=John');

// false
DistinguishedName::isValid('String containing rdn cn=John');

// false
DistinguishedName::isValid(null);

// false
DistinguishedName::isValid('');
```

### `get`

Get the full value of the Distinguished Name:

```php
$dn = DistinguishedName::make('cn=John Doe,dc=local,dc=com');

// "cn=John Doe,dc=local,dc=com"
$dn->get();
```

### `set`

Set the value of the Distinguished Name:

```php
$dn = DistinguishedName::make('cn=other,dc=local,dc=com');

$dn->set('cn=John Doe,dc=local,dc=com');

// "cn=John Doe,dc=local,dc=com"
$dn->get();
```

### `explode`

Explode a Distinguished Name string:

```php
$dn = DistinguishedName::explode('cn=john doe,dc=local,dc=com');

// [
//   "cn=john doe",
//   "dc=local",
//   "dc=com",
// ]
var_dump($dn);
```

### `explodeRdn`

Explode a Relative Distinguished Name string:

```php
[$attribute, $value] = DistinguishedName::explodeRdn('cn=john doe');

// "cn"
echo $attribute;

// "john doe"
echo $value;
```

### `makeRdn`

Make a Relative Distinguished Name string:

```php
$rdn = DistinguishedName::makeRdn(['cn', 'john doe']);

// "cn=john doe"
echo $rdn;
```

### `unescape`

```php
$unescaped = DistinguishedName::unescape('\6a\6f\68\6e\2c\64\6f\65');

// "doe, john"
echo $unescaped;
```

### `name`

Get the Relative Distinguished Name's _value_:

```php
$dn = DistinguishedName::make('cn=John Doe,dc=local,dc=com');

// "John Doe"
$dn->name();
```

### `head`

Get the Relative Distinguished Name's _attribute_:

```php
$dn = DistinguishedName::make('cn=John Doe,dc=local,dc=com');

// "cn"
$dn->head();
```

### `relative`

Get the Relative Distinguished Name:

```php
$dn = DistinguishedName::make('cn=John Doe,dc=local,dc=com');

// "cn=John Doe"
$dn->relative();
```

### `parent`

Get the parent Distinguished Name:

```php
$dn = DistinguishedName::make('cn=John Doe,dc=local,dc=com');

// "dc=local,dc=com"
$dn->parent();
```

### `values`

Get the values of each DN component:

```php
$dn = DistinguishedName::make('cn=John Doe,dc=local,dc=com');

// array:3 [
//  0 => "John Doe"
//  1 => "local"
//  2 => "com"
// ]
$dn->values();
```

### `components`

Get the Relative Distinguished Name's of each DN component:

```php
$dn = DistinguishedName::make('cn=John Doe,dc=local,dc=com');

// array:3 [
//  0 => "cn=John Doe"
//  1 => "dc=local"
//  2 => "dc=com"
// ]
$dn->components();
```

### `assoc`

Get an associative array of the Distinguished Name component's, grouping them using their attribute name:

```php
$dn = DistinguishedName::make('cn=John Doe,dc=local,dc=com');

// array:2 [
//  "cn" => array:1 [
//    0 => "John Doe"
//  ]
//  "dc" => array:2 [
//    0 => "local"
//    1 => "com"
//  ]
// ]
$dn->assoc();
```

### `multi`

Split the Relative Distinguished Name's of each DN component into an associative array:

```php
$dn = DistinguishedName::make('cn=John Doe,dc=local,dc=com');

// array: 3 [
//   0 => ['cn', 'John'],
//   1 => ['ou', 'local'],
//   2 => ['dc', 'local'],
// ]
$dn->multi();
```

### `isEmpty`

Determine if the Distinguished Name has any values:

```php
// false
DistinguishedName::make('cn=John Doe,dc=local,dc=com')->isEmpty();

// false
DistinguishedName::make('cn=John Doe')->isEmpty();

// true
DistinguishedName::make(null)->isEmpty();

// true
DistinguishedName::make('')->isEmpty();
```

### `isParentOf`

Determine if the Distinguished Name is a _direct_ parent of the given child:

```php
$dn = DistinguishedName::make('ou=users,dc=local,dc=com');

// false
$dn->isParentOf(
  DistinguishedName::make('ou=accounting,dc=local,dc=com')
);

// true
$dn->isParentOf(
  DistinguishedName::make('ou=office,ou=users,dc=local,dc=com')
);
```

### `isChildOf`

Determine if the Distinguished Name is a _direct_ child of the given parent:

```php
$dn = DistinguishedName::make('cn=John Doe,dc=local,dc=com');

// false
$dn->isChildOf(
  DistinguishedName::make('ou=users,dc=local,dc=com')
);

// true
$dn->isChildOf(
  DistinguishedName::make('dc=local,dc=com')
);
```

### `isAncestorOf`

Determine if the Distinguished Name is an ancestor of the given descendant/child:

```php
$dn = DistinguishedName::make('ou=users,dc=local,dc=com');

// false
$dn->isAncestorOf(
  DistinguishedName::make('dc=local,dc=com')
);

// true
$dn->isAncestorOf(
  DistinguishedName::make('ou=accounting,ou=users,dc=local,dc=com')
);

// true
$dn->isAncestorOf(
  DistinguishedName::make('ou=other,ou=accounting,ou=users,dc=local,dc=com')
);
```

### `isDescendantOf`

Determine if the Distinguished Name is an descendant of the given ancestor/parent:

```php
$dn = DistinguishedName::make('cn=John Doe,ou=accounting,ou=users,dc=local,dc=com')

// false
$dn->isDescendantOf(
  DistinguishedName::make('ou=admin,dc=local,dc=com')
);

// true
$dn->isDescendantOf(
  DistinguishedName::make('ou=users,dc=local,dc=com')
);

// true
$dn->isDescendantOf(
  DistinguishedName::make('ou=accounting,ou=users,dc=local,dc=com')
);
```

## Distinguished Name Building

The Distinguished Name Builder allows you to build and transform Distinguished Names.

```php
use LdapRecord\Models\Attributes\DistinguishedNameBuilder;
```

> **Important**:
>
> -   All transformation methods can be chained.
> -   Values given to the `prepend` and `append` are escaped.
> -   Missing method calls are forwarded to a `DistinguishedName` instance.

### `components`

Get all of the components of the DN.

```php
$dn = DistinguishedName::build('cn=john doe,ou=users,dc=local,dc=com');

// array:4 [
//   0 => array:2 [
//     0 => "cn"
//     1 => "john doe"
//   ]
//   1 => array:2 [
//     0 => "ou"
//     1 => "users"
//   ]
//   2 => array:2 [
//     0 => "dc"
//     1 => "local"
//   ]
//   3 => array:2 [
//     0 => "dc"
//     1 => "com"
//   ]
// ]
$dn->components();
```

Get the components of a particular type:

```php
$dn = DistinguishedName::build('cn=john doe,ou=users,dc=local,dc=com');

// array:2 [
//   0 => array:2 [
//     0 => "dc"
//     1 => "local"
//   ]
//   1 => array:2 [
//     0 => "dc"
//     1 => "com"
//   ]
// ]
$dn->components('dc');
```

### `prepend`

Prepend an RDN onto the DN.

```php
$dn = DistinguishedName::build('dc=com');

// Use an attribute and value:
$dn->prepend('dc', 'local');

// Use an RDN:
$dn->prepend('cn=john');

// "cn=john,dc=local,dc=com"
$dn->get();
```

### `append`

Append an RDN onto the DN.

```php
$dn = DistinguishedName::build('cn=john');

// Use an attribute and value:
$dn->append('dc', 'local');

// Use an RDN:
$dn->append('dc=com');

// "cn=john,dc=local,dc=com"
$dn->get();
```

### `pop`

Pop an RDN off of the end of the DN.

```php
// "cn=john,dc=local"
DistinguishedName::build('cn=john,dc=local,dc=com')
  ->pop()
  ->get();

// "cn=john"
DistinguishedName::build('cn=john,dc=local,dc=com')
  ->pop(2)
  ->get();

// "cn=john"
DistinguishedName::build('cn=john,dc=local,dc=com')
  ->pop(2, $removed)
  ->get();

// array:2 [
//   0 => "dc=local"
//   1 => "dc=com"
// ]
var_dump($removed);
```

### `shift`

Shift an RDN off of the beginning of the DN.

```php
// "dc=local,dc=com"
DistinguishedName::build('cn=john,dc=local,dc=com')
  ->shift()
  ->get();

// "dc=com"
DistinguishedName::build('cn=john,dc=local,dc=com')
  ->shift(2)
  ->get();

// "dc=com"
DistinguishedName::build('cn=john,dc=local,dc=com')
  ->shift(2, $removed)
  ->get();

// array:2 [
//   0 => "cn=john"
//   1 => "dc=local"
// ]
var_dump($removed);
```

### `reverse`

Whether to output the DN in reverse.

```php
// "dc=com,dc=local,cn=john"
DistinguishedName::build('cn=john,dc=local,dc=com')
  ->reverse()
  ->get();
```

### Long Chain Example

```php
$dn = DistinguishedName::of('cn=John Doe,dc=local,dc=com')
    ->shift(1, $removed)
    ->prepend('ou', 'users')
    ->prepend($removed)
    ->pop(1, $removed)
    ->append('dc', 'org')
    ->append($removed)
    ->get();

// "cn=John Doe,ou=users,dc=local,dc=org,dc=com"
echo $dn;
```

## Passwords

```php
use LdapRecord\Models\Attributes\Password;
```

The Password helper allows you to create hashed passwords, as
well as encode them for transmission to your LDAP server.

> **Important**: You do not need to use any of these utilities
> when connecting to an Active Directory server. Password
> encoding is done for you with the included `User` model.

### `encode`

Make an encoded password for transmission over LDAP.

```php
// "\x00s\x00e\x00c\x00r\x00e\x00t\x00"\x00"
Password::encode('secret');
```

### `smd5`

Make a salted md5 password.

```php
// "{SMD5}i3f4A6FAN0MDFaaZU23fu8FcHw4="
Password::smd5('secret');

// "{SMD5}mc0uWpXVVe5747A4pKhGJXNhbHQ="
Password::smd5('secret', 'salt');
```

### `ssha`

Make a salted SHA password.

```php
// "{SSHA}L8EHaF8fyBVlTrvHbdE5/7MnCN1sR4az"
Password::ssha('secret');

// "{SSHA}gVK8WC9YyFT1gMsQHTGCgT3sSv5zYWx0"
Password::ssha('secret', 'salt');
```

### `ssha256`

Make a salted SSHA256 password.

```php
// "{SSHA256}f30+bbvnM24awEIG2iLZ12TcsjFT7e+OP3/fFmmqMZNdQRP/"
Password::ssha256('secret');

// "{SSHA256}+E+iFJ27Yu1ODPH1UNKUmzOmUT06dwfghQJRHHnMsO5zYWx0"
Password::ssha256('secret', 'salt');
```

### `ssha384`

Make a salted SSHA384 password.

```php
// "{SSHA384}x57dAvYd0LnqXDLxgmCqgrR585r2Ej4Lyxm+SQqY2fr1yzgIGz/t48MlKwEy+96jeShdcg=="
Password::ssha384('secret');

// "{SSHA384}BPdC1qPVnOtOWlZBhlNvMSsThLk7gG0moXRB2Ulg+UGkFToChXZ4jNzGfK5Uh3Otc2FsdA=="
Password::ssha384('secret', 'salt');
```

### `ssha512`

Make a salted SSHA512 password.

```php
// "{SSHA512}udY8kkohMXfh4YKmrMWbXk1CWf2xpzarDAOLTPBezod5JSwbgMvgCAjeJiYvmfrsfyHlVqc/4nmfaH7Hlvumo/cB2Jg="
Password::ssha512('secret');

// "{SSHA512}E491yrR9AdCoE7rbOPYS3EZgSuZpVE65AD9xko08s6floNesY/Zpe9zMVvLix4S2FiQSJ99RIkNvhHomNO9uL3NhbHQ="
Password::ssha512('secret', 'salt');
```

### `sha`

Make a non-salted SHA password.

```php
// "{SHA}5en6G6MezRroT3XKqkdPOmY/BfQ="
Password::sha('secret');
```

### `sha256`

Make a non-salted SHA256 password.

```php
// "{SHA256}K7gNU3sdo+OL0wNhqoVWhr3g6s1xYv72ol/pe/Unols="
Password::sha256('secret');
```

### `sha384`

Make a non-salted SHA384 password.

```php
// "{SHA384}WKd1ukESvjAFrkQHznV9iP2nHUBJe7gCbsrFTU4//HIyzo3jq1rLMK45dg/ufFPt"
Password::sha384('secret');
```

### `sha512`

Make a non-salted SHA512 password.

```php
// "SHA512}vSsar3708Jvp9Szi2NWZZ02Bqp1qRCFpbcTZPdBhnWgs5WtNZKnvCXdhztmeD2cmW192CF5bDufKRpayrW/isg=="
Password::sha512('secret');
```

### `md5`

Make a non-salted md5 password.

```php
// "{MD5}Xr4ilOzQ4PCOq3aQ0qbuaQ=="
Password::md5('secret');
```

### `md5Crypt`

Crypt password with an MD5 salt.

```php
// "{CRYPT}$1$hYstY89O$EHfOOWhp4qJ0.lDJ2owwb."
Password::md5Crypt('secret');

// "{CRYPT}saHW9GdxihkGQ"
Password::md5Crypt('secret', 'salt');
```

### `sha256Crypt`

Crypt password with a SHA256 salt.

```php
// "{CRYPT}$5$aRmWk3iiWxTdJ$iTy8QAfarSnilA3nM6SSH67qB2VpZiRbxXkA4FddfdD"
Password::sha256Crypt('secret');

// "{CRYPT}saHW9GdxihkGQ"
Password::sha256Crypt('secret', 'salt');
```

### `sha512Crypt`

Crypt a password with a SHA512 salt.

```php
// "{CRYPT}$6$GcGAYPV4NIvbC$odXh0cW4xldt2YUTqCaxjwFvRjiLA4CyDsQYaY1zLEfB4XXzsq6MFru9TAMbzR8hs0nJjmi5fFHrAB9hmaFF.."
Password::sha512Crypt('secret');

// "{CRYPT}saHW9GdxihkGQ"
Password::sha512Crypt('secret', 'salt');
```

## Utilities

```php
use LdapRecord\Utilities;
```

Provides methods for various LDAP related tasks.

### `explodeDn`

Converts a DN string into an array of RDNs. Returns `false` if an invalid DN is given.

```php
// array:3 [
//   0 => "john"
//   1 => "local"
//   2 => "com"
// ]
Utilities::explodeDn('cn=john,dc=local,dc=com');

// array:3 [
//   0 => "cn=john"
//   1 => "dc=local"
//   2 => "dc=com"
// ]
Utilities::explodeDn('cn=john,dc=local,dc=com', $removeAttributePrefixes = false);
```

### `unescape`

Un-escapes a hexadecimal string into its original string representation.

```php
$value = "\44\6f\65\2c\20\4a\6f\68\6e"

// "Doe, John"
Utilities::unescape($value);
```

### `binarySidToString`

Convert a binary SID to a string SID.

```php
Utilities::binarySidToString($binarySID);
```

### `binaryGuidToString`

Convert a binary GUID to a string GUID.

```php
Utilities::binaryGuidToString($binaryGUID);
```

### `stringGuidToHex`

Converts a string GUID to it's hex variant.

```php
$guid = '270db4d0-249d-46a7-9cc5-eb695d9af9ac';

// "\d0\b4\0d\27\9d\24\a7\46\9c\c5\eb\69\5d\9a\f9\ac"
Utilities::stringGuidToHex($guid);
```

### `convertWindowsTimeToUnixTime`

Round a Windows timestamp down to seconds and remove
the seconds between 1601-01-01 and 1970-01-01.

```php
Utilities::convertWindowsTimeToUnixTime($windowsTimestamp);
```

### `convertUnixTimeToWindowsTime`

Convert a Unix timestamp to Windows timestamp.

```php
Utilities::convertUnixTimeToWindowsTime($unixTimestamp);
```

### `isValidSid`

Validates that the inserted string is an object SID.

```php
// Returns "true"
Utilities::isValidSid('S-1-5-21-362381101-336104434-3030082-101');
Utilities::isValidSid('S-1-5-21-362381101-336104434');
Utilities::isValidSid('S-1-5-21-362381101');
Utilities::isValidSid('S-1-5-21');
Utilities::isValidSid('S-1-5');

// Returns "false"
Utilities::isValidSid('Invalid SID');
Utilities::isValidSid('S-1');
Utilities::isValidSid('');
```

### `isValidGuid`

Validates that the inserted string is an object GUID.

```php
// Returns "true"
Utilities::isValidGuid('59e5e143-a50e-41a9-bf2b-badee699a577');
Utilities::isValidGuid('8be90b30-0bbb-4638-b468-7aaeb32c74f9');
Utilities::isValidGuid('17bab266-05ac-4e30-9fad-1c7093e4dd83');

// Returns "false"
Utilities::isValidGuid('Invalid GUID');
Utilities::isValidGuid('17bab266-05ac-4e30-9fad');
Utilities::isValidGuid('');
```
