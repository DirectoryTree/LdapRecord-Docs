<p align="center">
    <img src="https://ldaprecord.com/logo.svg" width="300" alt="LdapRecord-Docs">
</p>

<p align="center">Versioned documentation for LdapRecord and its Laravel integration.</p>

<p align="center">
    <a href="https://ldaprecord.com">Documentation</a>
    <span> · </span>
    <a href="#contributing">Contributing</a>
    <span> · </span>
    <a href="https://github.com/DirectoryTree/LdapRecord.com">Website</a>
</p>

---

## Structure

Documentation pages are written in MDX and organized by package and major version:

- `core/v1` through `core/v4` contain the LdapRecord core documentation.
- `laravel/v1` through `laravel/v4` contain the Laravel integration documentation.

Edit the `page.mdx` file for the package and version your change applies to. Preserve the page's metadata and update older versions only when the change applies to them.

## Contributing

This repository contains documentation content. The [LdapRecord.com](https://github.com/DirectoryTree/LdapRecord.com) repository provides the Next.js website that renders it as a Git submodule at `src/app/docs`.

To preview documentation changes locally, use Node.js 22 and clone the website with its submodule:

```bash
git clone --recurse-submodules https://github.com/DirectoryTree/LdapRecord.com.git
cd LdapRecord.com
npm ci
npm run dev
```

Edit the documentation under `src/app/docs` and visit [http://localhost:3000](http://localhost:3000) to preview it. Create a branch in the `src/app/docs` repository before committing and submitting your changes; a newly initialized submodule starts with a detached HEAD.

For website component, layout, or navigation changes, submit a pull request to [LdapRecord.com](https://github.com/DirectoryTree/LdapRecord.com).
