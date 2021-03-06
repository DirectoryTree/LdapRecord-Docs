---
title: Configuration
description: Configuring LdapRecord
---

# Configuration

To configure your LDAP connections, provide an array with key-value pairs to set various parameters.

Here is a list of all parameters.

```php
$config = [
    // Mandatory Configuration Options
    'hosts'            => ['192.168.1.1'],
    'base_dn'          => 'dc=local,dc=com',
    'username'         => 'cn=admin,dc=local,dc=com',
    'password'         => 'password',

    // Optional Configuration Options
    'port'             => 389,
    'use_ssl'          => false,
    'use_tls'          => false,
    'version'          => 3,
    'timeout'          => 5,
    'follow_referrals' => false,

    // Custom LDAP Options
    'options' => [
        // See: http://php.net/ldap_set_option
        LDAP_OPT_X_TLS_REQUIRE_CERT => LDAP_OPT_X_TLS_HARD
    ]
];

$connection = new Connection($config);
```

### Hosts

The hosts option is an array of IP addresses or host names located on your network that serve an LDAP directory.

You insert as many servers or as little as you'd like depending on your forest (with the minimum of one of course).

> Do not append your port to your IP addresses or host names. Use the `port` configuration option instead.

### Base Distinguished Name

A 'Distinguished Name' is a string based identifier in LDAP that is used to indicate hierarchy.

Each object in your domain is assigned a Distinguished Name.

An example Distinguished Name would be `cn=John Doe,ou=Users,dc=local,dc=com`.

The above can be broken into the following 'Relative Distinguished Names' (RDN for short):

| RDN               | Meaning                                                 |
| ----------------- | ------------------------------------------------------- |
| `cn=John Doe`     | The object has a 'Common Name' of `John Doe`            |
| `ou=Users`        | The object resides in the 'Organizational Unit' `Users` |
| `dc=local,dc=com` | The object resides in the 'Domain' `local.com`          |

A 'Base Distinguished Name' is the distinguished name that you would like to
be used as the root of all searches and object creations using LdapRecord.

An example base DN would be `dc=local,dc=com`.

This means, that all searches executed with LdapRecord will start at `dc=local,dc=com`
as the root. This would allow all records _below_ it to be retrieved from results.

If you do not define a base DN, you will not retrieve any search results from queries.

> Your base DN is **case insensitive**. You do not need to worry about incorrect casing.

### Username & Password

To connect to your LDAP server, a username and password is required to be able to query and run operations on your server(s).

| Additional Tips                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| The `username` option must be a users [Distinguished Name](https://docs.microsoft.com/en-us/previous-versions/windows/desktop/ldap/distinguished-names). If you are connecting to an Active Directory server, you may use a users [userPrincipalName](https://docs.microsoft.com/en-us/windows/win32/secauthn/user-name-formats#user-principal-name) (`username@domain.com`) or [Down-Level Logon Name](https://docs.microsoft.com/en-us/windows/win32/secauthn/user-name-formats#down-level-logon-name) (`DOMAIN\\username`) instead. |
| To run administration level operations, such as resetting passwords, this account **must** have the permissions to do so on your directory.                                                                                                                                                                                                                                                                                                                                                                                            |

### Port

The port option is used for authenticating and binding to your LDAP server.

The default ports are already used for non SSL and SSL connections (`389` and `636`).

Only insert a port if your LDAP server uses a unique port.

### SSL & TLS

These boolean options enable an SSL or TLS connection to your LDAP server.

It is recommended to use one of these options if you have the ability to. This ensures secure connectivity.

| Requirements & Tips                                                                                              |
| ---------------------------------------------------------------------------------------------------------------- |
| Only **one** can be set to `true`. You must chose either or.                                                     |
| You **must** enable SSL or TLS to set / change / reset passwords in Active Directory.                            |
| **TLS is recommended over SSL**. SSL is labelled as a deprecated mechanism for securely running LDAP operations. |

If you're having connectivity issues over SSL or TLS, you may have to
create an `ldap.conf` file and add the following inside:

```text
TLS_REQCERT never
```

The `ldap.conf` file will likely not exist by default. Create it inside the location for your OS:

| OS      | Location                            |
| ------- | ----------------------------------- |
| Windows | `C:\OpenLDAP\sysconf\ldap.conf`     |
| Linux   | `/etc/ldap/ldap.conf`               |
| macOS   | `/usr/local/etc/openldap/ldap.conf` |

The above directories may not exist - you will need to create them in such case.

> **Important**: You **must** restart your web server after making changes
> to the `ldap.conf` file. In some cases, you may even have to restart
> your workstation or server for the changes to take effect.

If you can connect using `TLS_REQCERT never` inside of your `ldap.conf` file,
you may want to copy your domain CA certificate to your web server, as it
can be a bit of a security risk as it will ignore invalid certificates.

Copy your domain CA certificate to the following location:

| OS            | Location              |
| ------------- | --------------------- |
| Windows       | `C:\OpenLDAP\sysconf` |
| Linux / macOS | `/etc/ssl/certs`      |

Then, reference it in your `ldap.conf` with the full file path using (replace
`my-custom-path` with the location of the file):

```text
TLS_CACERT my-custom-path/ca.pem
TLS_REQCERT hard
```

Windows Example:

```text
TLS_CACERT C:\OpenLDAP\sysconf\ca.pem
TLS_REQCERT hard
```

Linux / macOS Example:

```text
TLS_CACERT /etc/ssl/certs/ca.pem
TLS_REQCERT hard
```

### Timeout

The timeout option allows you to configure the amount of seconds to wait
until your application receives a response from your LDAP server.

The default is `5` seconds.

### Version

The LDAP version to use for your connection.

Must be an integer and can either be `2` or `3`.

### Follow Referrals

The follow referrals option is a boolean to tell Active Directory to follow a referral to another server on your network if the server queried knows the information your asking for exists, but does not yet contain a copy of it locally.

This option is defaulted to `false`.

Disable this option if you're experiencing search / connectivity issues.

For more information, visit:
[https://technet.microsoft.com/en-us/library/cc978014.aspx](https://technet.microsoft.com/en-us/library/cc978014.aspx)

### Options

Arbitrary options can be set for the connection to fine-tune TLS and connection behavior.

> **Important**: The following options will be ignored if set:
>
> - `LDAP_OPT_PROTOCOL_VERSION`
> - `LDAP_OPT_NETWORK_TIMEOUT`
> - `LDAP_OPT_REFERRALS`
>
> These are instead set with the `version`, `timeout` and `follow_referrals` keys.

Valid LDAP options are listed in the [ldap_set_option](http://php.net/ldap_set_option) PHP documentation.
