# Default CHANGELOG.md template

Use this only when the repository has no changelog and the user confirms this is the first one. Replace the project
and team names, then put the generated entry where the example entry is.

```markdown
# Changelog

All notable changes to this project are documented in this file.

## 1.0.1 - 2026-10-02

- Changed
  - Companies API returns `company_type` `radio_publisher` / `tv_publisher` for publisher companies, was empty.
- Fix
  - Radio campaign creation no longer answers 500 ("501 Invalid RCPT TO address provided") on a malformed spot
    provider e-mail such as `<name@domain.it`: e-mails get a real format check.
  - Frontend packs built with `NODE_ENV=production`; were development builds (unminified, "You are running Vue in
    development mode").
- Removed
  - Public API `tv|radio/competitors/alerts` (no caller).
- Tests
  - Request specs assert what their names claim: report job params, filters applied, 404 on foreign products.
```

Conventions of this style:

- **Heading:** `## X.Y.Z - YYYY-MM-DD`, newest version first.
- **Sections:** `Changed`, `Fix`, `Removed`, `Tests`, in that order. Leave out any section with no lines.
- **Bullets:** each section is a `- ` bullet and its lines are two-space-indented `- ` bullets under it.
- **Width:** wrap at 120 characters, with continuation lines indented under the bullet text.
- **Lines:** one short line per effect that a user or operator notices. Put exact error strings in quotes and code,
  paths and env vars in backticks.
