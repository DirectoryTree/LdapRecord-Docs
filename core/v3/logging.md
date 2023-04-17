---
title: Logging
description: Logging events in LdapRecord
---

# Logging

LdapRecord includes an implementation of PSR's widely supported [Logger](https://github.com/php-fig/log) interface.

By default, all of LdapRecord's [events](/docs/core/v2/events) will call the logger you have set to utilize.

> LdapRecord does not include a file / text logger. You must implement your own.

## Registering & Enabling a Logger

To register a logger call `LdapRecord\Container::setLogger()`. The logger must implement the `Psr\Log\LoggerInterface`.

```php
\LdapRecord\Container::setLogger($myLogger);
```

## Disabling Logging

If you need to disable the event logger after a certain set of operations, simply pass in `null` and logging will be disabled:

```php
\LdapRecord\Container::setLogger($myLogger);

$connection = new \LdapRecord\Connection(['...']);

try {
    $connection->connect();

    // Disable logging anything else.
    \LdapRecord\Container::setLogger(null);
} catch (\LdapRecord\Auth\BindException $e) {
    //
}
```

## Logged Information

After enabling LdapRecord logging, the following events are logged:

### `LdapRecord\Auth\Events\Attempting`

```text
LDAP (ldap://192.168.1.1:389) - Operation: LdapRecord\Auth\Events\Attempting - Username: CN=Steve Bauman,OU=Users,DC=local,DC=com
```

### `LdapRecord\Auth\Events\Binding`

```text
LDAP (ldap://192.168.1.1:389) - Operation: LdapRecord\Auth\Events\Binding - Username: CN=Steve Bauman,OU=Users,DC=local,DC=com
```

### `LdapRecord\Auth\Events\Bound`

```text
LDAP (ldap://192.168.1.1:389) - Operation: LdapRecord\Auth\Events\Bound - Username: CN=Steve Bauman,OU=Users,DC=local,DC=com
```

### `LdapRecord\Auth\Events\Passed`

```text
LDAP (ldap://192.168.1.1:389) - Operation: LdapRecord\Auth\Events\Passed - Username: CN=Steve Bauman,OU=Users,DC=local,DC=com
```

### `LdapRecord\Auth\Events\Failed`

```text
LDAP (ldap://192.168.1.1:389) - Operation: LdapRecord\Auth\Events\Failed - Username: CN=Steve Bauman,OU=Users,DC=local,DC=com - Result: Invalid Credentials
```

### `LdapRecord\Models\Events\Saving`

```text
LDAP (ldap://192.168.1.1:389) - Operation: Saving - On: LdapRecord\Models\Entry - Distinguished Name: cn=John Doe,DC=local,DC=com
```

### `LdapRecord\Models\Events\Saved`

```text
LDAP (ldap://192.168.1.1:389) - Operation: Saved - On: LdapRecord\Models\Entry - Distinguished Name: cn=John Doe,DC=local,DC=com
```

### `LdapRecord\Models\Events\Creating`

```text
LDAP (ldap://192.168.1.1:389) - Operation: Creating - On: LdapRecord\Models\Entry - Distinguished Name: cn=John Doe,DC=local,DC=com
```

### `LdapRecord\Models\Events\Created`

```text
LDAP (ldap://192.168.1.1:389) - Operation: Created - On: LdapRecord\Models\Entry - Distinguished Name: cn=John Doe,DC=local,DC=com
```

### `LdapRecord\Models\Events\Updating`

```text
LDAP (ldap://192.168.1.1:389) - Operation: Updating - On: LdapRecord\Models\Entry - Distinguished Name: cn=John Doe,DC=local,DC=com
```

### `LdapRecord\Models\Events\Updated`

```text
LDAP (ldap://192.168.1.1:389) - Operation: Updated - On: LdapRecord\Models\Entry - Distinguished Name: cn=John Doe,DC=local,DC=com
```

### `LdapRecord\Models\Events\Deleting`

```text
LDAP (ldap://192.168.1.1:389) - Operation: Deleting - On: LdapRecord\Models\Entry - Distinguished Name: cn=John Doe,DC=local,DC=com
```

### `LdapRecord\Models\Events\Deleted`

```text
LDAP (ldap://192.168.1.1:389) - Operation: Deleted - On: LdapRecord\Models\Entry - Distinguished Name: cn=John Doe,DC=local,DC=com
```
