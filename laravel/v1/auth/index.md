---
title: Authentication Overview
description: Authenticating LDAP users into your application
---

# Authentication Overview

LdapRecord-Laravel comes with two ways to authenticate LDAP users into your application.

## Plain Authentication

If you have an application that doesn't require any user data to be synchronized to your database,
then you can utilize plain LDAP authentication.

When a user is successfully authenticated, the Laravel `Auth::user()` method will return an instance
of your configured **LdapRecord** model.

It is paramount to understand that upon every request a logged in user makes to your application,
an LDAP search request will be made to retrieve them. If you do not want this to occur, you must
use synchronized database authentication.

## Synchronized Database Authentication

Synchronized Database LDAP Authentication means that an LDAP user that has successfully passed LDAP authentication
is created & synchronized to your local applications database. This is helpful as you can attach typical
relational database information to them, such as blog posts, attachments, etc.

When a user is successfully authenticated, the Laravel `Auth::user()` method will return an instance of
your configured database **Eloquent** model. Subsequent requests to your application with logged in
users will retrieve the logged in user from your database, rather than your LDAP directory.

This works via the following process:

1. A user attempts to login to your application
2. LdapRecord attempts to locate the user in your LDAP directory
3. If a user is found, LDAP authentication now occurs and the users password is sent to your directory and validated
4. If authentication passes, a local database record is created in your `users` database table with the users attributes synchronized
5. The user is logged into your application
