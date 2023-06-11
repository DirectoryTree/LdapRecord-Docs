---
title: Release Notes
description: LdapRecord-Laravel v3 Release Notes
---

# LdapRecord-Laravel v3.0.0 Release Notes

We're excited to announce the release of LdapRecord-Laravel v3.0.0! This major version
introduces changes and improvements, focusing on stricter typing, utilizing new PHP
features and methods, updated dependencies, and a new import and authentication 
configuration option. Please review the following release notes to understand
the new features and changes.

## Key Features and Changes

### PHP >= 8.1 Requirement

LdapRecord-Laravel v3 now requires PHP version 8.1 or higher to take advantage of the latest
language features, optimizations, and security improvements. The whole codebase has
been reviewed and refreshed with usage of new language features where applicable.

### Laravel >= 8.0 Requirement

LdapRecord-Laravel v3 now requires Laravel version 8.0 or higher to 
take advantage of the latest framework features, optimizations, 
and security improvements.

### Strict Types Implementation

We've implemented strict types across all classes and methods to enforce 
better type safety and catch potential errors during development. This 
will lead to more robust and maintainable code.

### New Import and Authentication Options

- A new [`min-users` command option](/docs/laravel/v3/auth/database/importing#min-users)
has been added to the `ldap:import` command, allowing to you set a minumum number 
of users to be returned by your LDAP server before importing users and 
performing soft-deletes.

- A new [`scopes` configuration option](/docs/laravel/v3/auth/database/configuration#scopes) 
has been made available to be added to your authentication provider in your 
`config/auth.php` file. This allows you to add [model query scopes]() to 
refine/restrict the users who may authenticate into your application.
