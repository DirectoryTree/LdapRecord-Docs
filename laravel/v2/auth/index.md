---
title: Authentication Overview
description: Authenticating LDAP users into your application
extends: _layouts.laravel.page
section: content
---

# Authentication Overview {#overview}

LdapRecord-Laravel comes with two ways to authenticate LDAP users into your application.
Read each use case below and select one that best suits your needs.

## Plain Authentication

- You don't need to attach any data to your LDAP users.
- You're okay with your application being inaccessible if your LDAP server is down.
- Your application critically depends on LDAP user roles and status (i.e. user enablement/disablement, group memberships).

[Plain Authentication Overview](/docs/laravel/v2/auth/plain/)

## Synchronized Database Authentication

- You need to attach data to your LDAP users.
- Your application may need to authenticate other registered non-LDAP users.
- Your application must be accessible if your LDAP server is unavailable or down.

[Synchronized Database Authentication Overview](/docs/laravel/v2/auth/database/)