---
title: Debugging
description: Debugging LdapRecord-Laravel
---

# Debugging

## Connectivity

LdapRecord-Laravel comes with a built-in command to test connectivity to your
LDAP servers. The exception message, error code, and diagnostic message are
displayed after a failure to bind to your LDAP server.

To test your connectivity, run the following command:

```bash
php artisan ldap:test
```

Then, the following will be output:

```text
Testing LDAP connection [default]...

+------------+------------+-----------------+-------------------------------------------------------------------------------------------------------------+---------------+
| Connection | Successful | Username        | Message                                                                                                     | Response Time |
+------------+------------+-----------------+-------------------------------------------------------------------------------------------------------------+---------------+
| default    | ✘ No       | user@local.com  | ldap_bind(): Unable to bind to server: Can't contact LDAP server. Error Code: [-1] Diagnostic Message: null | 5008.72ms     |
+------------+------------+-----------------+-------------------------------------------------------------------------------------------------------------+---------------+
```

The returned error codes and diagnostic messages can help you greatly
when attempting to debug SSL and TLS connectivity issues.

### TLS & SSL

TLS and SSL can be very tricky to get up and running. You will most likely have
to place an `ldap.conf` file onto your local / production server to indicate
that you would like to either _bypass_ TLS / SSL certificate verification,
or use a valid certificate that you have retrieved from your LDAP server.

This process is fully documented on the [configuration](/docs/core/v2/configuration#ssl-amp-tls)
documentation. It includes per operating system level instructions on where your `ldap.conf` file
is located (or where it must be created), as well as what it must contain.

> **Important**: You **must** restart your web server after making changes
> to the `ldap.conf` file. In some cases, you may even have to restart
> your workstation or server for the changes to take effect.

## Directory and Objects

LdapRecord-Laravel comes with a built-in command to browse and navigate through your LDAP directories interactively.

To browse your directory, use the `ldap:browse {connection}` command:

> **Note**: If no connection is specified, the browse command will connect to your default LDAP connection.

```bash
php artisan ldap:browse
```

## Logging In

To debug issues logging in, its recommended to first complete the following steps:

1. Enabled logging via the `logging` key inside of your `config/ldap.php` file <br/>
   (or by enabling it via your `.env` by using the `LDAP_LOGGING` key)
2. Clear your configurations cache (if enabled) by running the `php artisan config:clear` command
3. Add the `ListensForLdapBindFailure` [trait](/docs/laravel/v2/auth/setup/#displaying-ldap-error-messages) onto your `LoginController`
4. Attempt logging in again

After completing the above, the first thing to lookout for is whether a
red error message is being displayed underneath your username / email
field.

If you do not see any error message and are immediately returned back to
the login page, then you have likely changed the username field on your
`resources/views/auth/login.blade.php` but have not updated it inside
of your `LoginController`, or vice versa.

For example, if you want users to login by a `username` instead of their
`email`, make sure you've changed this via the `username` method,
and the `credentials` method on your `LoginController`

```php
// app/Http/Controllers/Auth/LoginController.php

use Illuminate\Http\Request;

public function username()
{
    // This is the name of the HTML 'input' inside
    // of our 'login.blade.php' view:
    return 'username';
}

protected function credentials(Request $request)
{
    // 'samaccountname' is the attribute we are using to
    // locate users in our LDAP directory with. The
    // value of the key must be the input name of
    // our HTML input, as shown above:
    return [
        'samaccountname' => $request->get('username'),
        'password' => $request->get('password'),
    ];
}
```

If you simply see an **Invalid Credentials**, or **Can't contact LDAP server**
error, refer to your log files inside of your applications `storage/logs`
directory to investigate further. With `logging` enabled, all LDAP
searches, binds, failures and exceptions will be reported there.
