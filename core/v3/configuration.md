---
title: Configuration
description: Configuring LdapRecord
---

# Configuration

To configure your LDAP connections, you must provide an array to the
`Connection` class with key-value pairs to set various options.

Below is a list of all available options:

```php
use LdapRecord\Connection;

$connection = new Connection([
    // Mandatory Configuration Options
    'hosts'            => ['192.168.1.1'],
    'base_dn'          => 'dc=local,dc=com',
    'username'         => 'cn=admin,dc=local,dc=com',
    'password'         => 'password',

    // Optional Configuration Options
    'port'             => 389,
    'use_ssl'          => false,
    'use_tls'          => false,
    'use_sasl'         => false,
    'version'          => 3,
    'timeout'          => 5,
    'follow_referrals' => false,

    // Custom LDAP Options
    'options' => [
        // See: http://php.net/ldap_set_option
        LDAP_OPT_X_TLS_REQUIRE_CERT => LDAP_OPT_X_TLS_HARD
    ],
    
    // See: https://www.php.net/manual/en/function.ldap-sasl-bind.php
    'sasl_options' => [
        'mech' => null,
        'realm' => null,
        'authc_id' => null,
        'authz_id' => null,
        'props' => null,
    ],
]);
```

### Hosts

The hosts option is an array of IP addresses or host names located on your network that serve an LDAP directory.

You insert as many or as little as you'd like depending on your forest (with the minimum of one of course).

The first host in the array will always be used as the primary server. This means, all operations will take place underneath this host.

If the primary host fails to complete an operation (bind, query, modification, etc.), or the server does not
respond in the configured `timeout`, the same operation will be attempted on the following host in the array.

This automated fail-over process will continue for each host address, until a successful response is received.

> **Important**:
>
> - Do not append your port (`:389`, `:636`, etc.) to your hosts. <br/> Use the `port` configuration option instead.
> - Do not prepend your protocol (`ldap://` or `ldaps://`) to your hosts. <br/> Use the `use_ssl` configuration option instead.

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
as the root. This would allow all objects _below_ it to be retrieved from results.

> **Important**:
>
> - If you do not define a base DN, you will not retrieve any search results from queries.
> - Your base DN is **case-insensitive**. You do not need to worry about incorrect casing.

### Username & Password

To connect to your LDAP server, you must provide a username and password to be able to query and run operations on your server(s).

> **Important**:
>
> - The `username` option **must** be a users [Distinguished Name](https://docs.microsoft.com/en-us/previous-versions/windows/desktop/ldap/distinguished-names)
> - If however you are connecting to an Active Directory server, you may use:
>   - [userPrincipalName](https://docs.microsoft.com/en-us/windows/win32/secauthn/user-name-formats#user-principal-name) (`username@domain.com`)
>   - [Down-Level Logon Name](https://docs.microsoft.com/en-us/windows/win32/secauthn/user-name-formats#down-level-logon-name) (`DOMAIN\\username`)
> - To run administration level operations, such as resetting passwords, this account **must** have the permissions to do so on your directory.

### Port

The port option is used for opening a connection and binding to your LDAP server.

Default ports are already used for non SSL and SSL connections (`389` and `636`).

Only insert a port if your LDAP server uses a unique port.

> **Important**:
>
> - If enabling SSL, and `port` is set to `389`, it will be automatically overridden to use `636`.
> - If enabling TLS, you must use the default port for your LDAP server (`389`). SSL ports cannot be used.

### SSL & TLS

These boolean options enable an TLS or SSL connection to your LDAP server:

```php
$config = [
    // ...
    'use_tls' => false,
    'use_ssl' => false,
];
```

It is recommended to use _one_ of these options if you have the ability to. This ensures secure connectivity.

> **Important**:
>
> - Only **one** can be set to `true`. You must choose either or.
> - You **must** enable SSL or TLS to set / change / reset passwords in Active Directory.
> - **TLS is recommended over SSL**. SSL is labelled as a deprecated mechanism for securely running LDAP operations.

### SASL

This boolean option enables connecting to your LDAP server using SASL (Simple Authentication and Security Layer) via `ldap_sasl_bind`:

```php
$config = [
    // ...
    'use_sasl' => false,
];
```

SASL allows you to authenticate a user with various mechanisms, such as `DIGEST-MD5`, `CRAM-MD5`, `GSSAPI`, etc.

#### SASL Options

**`mech`** (string): This specifies the SASL mechanism to use for authentication. Common 
mechanisms include `DIGEST-MD5`, `CRAM-MD5`, and `GSSAPI`. If not specified, the server 
will choose the best available mechanism based on the client and server capabilities.

**`realm`** (string): This is the authentication realm or domain. It is used by some
SASL mechanisms to group users into different security domains or to map usernames 
to distinguished names. If not specified, the server will use its default realm.
Here's an example:

```php
$config = [
    // ...
    'sasl_options' => [
        'realm' => 'example.com',
    ],
];
```

**`authc_id`** (string): This is the authentication ID, which is used by some SASL 
mechanisms to identify the user during the authentication process. It may be a 
simple username or a more complex identifier (such as distinguished name), 
depending on the mechanism being used. Here's an example:

```php
$config = [
    // ...
    'sasl_options' => [
        'authc_id' => 'uid=jdoe,ou=admin,dc=example,dc=com'
    ]
];
```

**`authz_id`** (string): This is the authorization ID, which is used by some SASL 
mechanisms to identify the user for authorization purposes after authentication 
has been completed. It may be the same as the authentication ID or a different 
identifier (such as a distinguished name), depending on the mechanism being used. 
Here's an example:

```php
$config = [
    // ...
    'sasl_options' => [
        'authc_id' => 'jdoe'
    ]
];
```

**`props`** (string): This parameter allows you to specify additional security 
properties for the SASL mechanism being used. These properties can control 
various aspects of the security layer, such as encryption strength, integrity 
protection, and more. The format and available properties depend on the 
specific SASL mechanism being used.

When using the `DIGEST-MD5` SASL mechanism, you may use `props` to specify the
quality of protection (qop) and cipher options. The property string would be a
comma-separated list of key-value pairs, with keys and values separated by an 
equals sign (=). Here's an example:

```php
$config = [
    // ...
    'sasl_options' => [
        'props' => 'qop=auth-conf,cipher=rc4-56'
    ]
];
```

When using the GSSAPI SASL mechanism, you may use `props` to specify the 
`GSSAPI` flags, such as mutual authentication, delegation, and more.
Here's an example:

```php
$config = [
    // ...
    'sasl_options' => [
        'props' => 'gssapi_flags=mutual_required,delegate_cred'
    ]
];
```

#### Debugging

If you're having connectivity issues over SSL or TLS, you may have to
create an `ldap.conf` file and add the following inside:

```text
TLS_REQCERT never
```

The `ldap.conf` file will likely not exist by default. Create it inside the location for your OS:

| OS      | Location                                           |
|---------|----------------------------------------------------|
| Windows | `C:\OpenLDAP\sysconf\ldap.conf`                    |
| Linux   | `/etc/ldap/ldap.conf` or `/etc/openldap/ldap.conf` |
| macOS   | `/usr/local/etc/openldap/ldap.conf`                |

The above directories may not exist - you will need to create them in such case.

> **Important**:
>
> - You **must** restart your web server after making changes to the `ldap.conf` file.
> - In some cases, you may even have to restart your workstation or server for the changes to take effect.

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

The default is five (`5`) seconds.

> **Important**: If the timeout is reached performing an LDAP operation and
> you have specified multiple hosts in your configuration,
> the same timeout will be used for each host.
> </br></br>
> This means, if you have three (3) hosts in your configuration and two (2) of
> them do not respond (or fail), the operation will take ten (10) seconds +
> the amount of time the third (3rd) host takes to respond.

### Version

The LDAP version to use for your connection.

Must be an integer, and can either be two (`2`) or three (`3`).

> **Important**: It's heavily recommended to use version three (`3`). You may experience issues using version two (`2`).

### Follow Referrals

The follow referrals option is a boolean to tell Active Directory to
follow a referral to another server on your network if the server
queried knows the information your asking for exists, but does
not yet contain a copy of it locally.

This option is defaulted to `false`.

> **Important**: Disable this option if you're experiencing search / connectivity issues.
> </br></br>
> For more information, visit: [Microsoft Docs - LDAP Referrals](https://technet.microsoft.com/en-us/library/cc978014.aspx)

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
