---
title: Authentication Overview
description: Authenticating LDAP users into your application
---

# Authentication Overview

LdapRecord-Laravel comes with two ways to authenticate LDAP users into your application.
Read each use case below and select one that best suits your needs.

## Plain Authentication

- You don't need to attach any data to your LDAP users.
- You're okay with your application being inaccessible if your LDAP server is down.
- Your application critically depends on LDAP user roles and status (i.e. user enablement/disablement, group memberships).

[Plain Authentication Overview](/docs/laravel/v3/auth/plain/)

## Synchronized Database Authentication

- You need to attach data to your LDAP users.
- Your application may need to authenticate other registered non-LDAP users.
- Your application must be accessible if your LDAP server is unavailable or down.

[Synchronized Database Authentication Overview](/docs/laravel/v3/auth/database/)

## Configure Without Credentials

To configure LdapRecord-Laravel authentication without credentails your LDAP server much have anonymous binding enabled. When binding anonymously, your permissions must be open enough so that users who need to sign in to your Laravel application can be read from your ActiveDirectory server, along with the attribute you are using for authentication.

To bind anonymously to your LDAP server, set your `username` and `password` to `null` inside your configuration.

> **Important**: A base DN must still be configured for searches to return results.

If anonymous binding is disabled, you must configure a `username` and `password` to connect to your LDAP server.

### Why Does LdapRecord Require Credentials?

Think of it like a database connection to your application. LdapRecord needs credentials to search your directory for the user who is attempting to sign in to your Laravel application by the attribute of your choosing. Without this access, it cannot search. You would have to have users enter in their full distinguished name to be able to sign in. Once signed in, LDAP access would be lost as soon as the PHP request ends, leaving most of the features in LdapRecord-Laravel in a non-working state.
