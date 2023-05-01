---
title: Release Notes
description: LdapRecord v3 Release Notes
---

# LdapRecord v3.0.0 Release Notes

We're excited to announce the release of LdapRecord v3.0.0! This major version introduces 
significant changes and improvements, focusing on stricter typing, utilizing new PHP 
features and methods, updated dependencies, and enhanced binding capabilities.
Please review the following release notes to understand the new features and changes.

## Key Features and Changes

### PHP >= 8.1 Requirement 

LdapRecord v3 now requires PHP version 8.1 or higher to take advantage of the latest 
language features, optimizations, and security improvements. The whole codebase has
been reviewed and refreshed with usage of new language features where applicable. 

### Strict Types Implementation

We've implemented strict types across all classes and methods to enforce better 
type safety and catch potential errors during development. This will lead to 
more robust and maintainable code.

### Dependency Update

The "Tightenco/Collect" package has been replaced with the "Illuminate/Collections"
package. This change provides a more modern, maintained package for handling 
collections, as "Tightenco/Collect" has discontinued support in favor of 
the core "Illuminate/Collections" pakage.

### SASL Binding Support

LdapRecord v3 now offers the ability to bind to your LDAP server using SASL 
(Simple Authentication and Security Layer). This enhancement provides more 
secure and flexible authentication options for connecting to LDAP servers.