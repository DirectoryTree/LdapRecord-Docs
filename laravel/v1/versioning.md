---
title: Versioning
description: LdapRecord-Laravel Versioning
---

# Versioning

LdapRecord-Laravel is versioned under the [Semantic Versioning](http://semver.org/) guidelines as much as possible.

> Major versions of LdapRecord-Laravel will always be kept in sync with the core LdapRecord repository.
> This means when LdapRecord-Laravel hits `v2.0.0`, LdapRecord `^v2.0` will be used.

Releases will be numbered with the following format:

`<major>.<minor>.<patch>`

And constructed with the following guidelines:

- Breaking backward compatibility bumps the major and resets the minor and patch.
- New additions without breaking backward compatibility bumps the minor and resets the patch.
- Bug fixes and misc changes bumps the patch.

Minor versions are not maintained individually, and you're encouraged to upgrade through to the next minor version.

Major versions are maintained individually through separate branches.
