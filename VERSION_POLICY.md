# GitHub Actions Versioning Policy

This document defines how we manage versions of GitHub Actions and reusable workflows across all plugin repositories.

---

## 🎯 Goals

- Keep CI stable and predictable
- Avoid breaking changes from upstream actions
- Ensure all plugin repos stay consistent
- Minimize maintenance overhead

---

## 1. Pin External Actions to Major Versions

Always pin external actions like this:

```
actions/checkout@v6
actions/setup-node@v4
actions/cache@v4
shivammathur/setup-php@v2
```

Why:

- Major versions are stable
- Dependabot can safely bump them
- Avoids unexpected breaking changes

---

## 2. Never Pin Reusable Workflows

Always reference reusable workflows like this:

```
uses: honorswp/ch-actions/.github/workflows/test-plugin.yml@main
```

Why:

- All plugin repos automatically inherit improvements
- No version drift
- No manual updates across dozens of repos

---

## 3. Dependabot Policy

Dependabot runs weekly and updates:

- GitHub Actions
- Composer dependencies
- npm dependencies

See `.github/dependabot.yml` in each repo.

---

## 4. Workflow Change Requirements

- All workflow changes must be made via PR
- All PRs must pass CI before merging
- Breaking changes must be documented in the repo root `CHANGELOG.md`

---

## 5. When to Create Versioned Workflow Tags

Only create versioned tags (e.g., `v1`, `v2`) if:

- You need to support legacy plugin repos
- You introduce a breaking change
- You want long-term stability for a specific workflow version

Otherwise, `@main` is preferred.

---

## 6. Security Requirements

- No unpinned third-party actions
- No actions from unknown authors
- All secrets must be passed explicitly via `secrets:` block

---

## 7. Deprecation Policy

When workflows are replaced:

- Mark old workflows as deprecated in the README
- Provide migration instructions
- Remove deprecated workflows after 90 days

---

This policy ensures CI remains stable, scalable, and easy to maintain across all HonorsWP plugin repositories.

