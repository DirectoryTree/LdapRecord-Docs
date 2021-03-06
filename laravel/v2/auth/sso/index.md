---
title: Pass-through / SSO Authentication
description: Overview of up pass-through / SSO authentication
---

# Pass-through / SSO Authentication

Pass-through authentication allows your users to be automatically signed in when they access your
application on a Windows domain joined computer. This feature is ideal for in-house corporate
environments.

However, this feature assumes that you have enabled Windows Authentication in IIS, or have enabled
it in some other means with Apache. LdapRecord does not set this up for you. To enable Windows
Authentication, visit the [IIS configuration guide](https://www.iis.net/configreference/system.webserver/security/authentication/windowsauthentication/providers/add).

When you have it enabled on your server, and a user visits your application from a domain joined computer,
the users `sAMAccountName` becomes available on a PHP server variable (`$_SERVER['AUTH_USER']`).

LdapRecord provides a middleware that you apply to your stack which retrieves this username
from the request, attempts to locate the user in your directory, then logs the user in.
