# ch-actions

Centralized, reusable GitHub Actions workflows for all plugin repositories.

This repository provides a single source of truth for:

- Build workflows
- Release workflows
- WordPress.org deployment workflows
- Full test suite workflows (PHPUnit + Jest)
- Linting workflows
- Shared CI policies and conventions

By centralizing CI logic here, all plugin repositories stay clean, consistent, and easy to maintain. Updating a workflow in this repo automatically improves CI across every plugin that references it.

---

## 📁 Repository Structure

```
ch-actions/
  README.md
  VERSION_POLICY.md
  WORKFLOW_GUIDE.md
  .github/
    workflows/
      build-plugin.yml
      release-plugin.yml
      wporg-release.yml
      wporg-readme.yml
      test-plugin.yml
      lint-plugin.yml
```

All reusable workflows must live inside `.github/workflows/`.

---

## 📦 Available Reusable Workflows

| Workflow | Description |
|---------|-------------|
| `build-plugin.yml` | Builds plugin ZIP using `.zipignore` for testing or QA. |
| `release-plugin.yml` | Builds, tags, creates GitHub release, deploys ZIP to WP.org. |
| `wporg-release.yml` | WordPress.org release + deploy (standalone). |
| `wporg-readme.yml` | Updates plugin readme + assets on WP.org. |
| `test-plugin.yml` | Full test suite: PHPUnit (unit + integration) + Jest, with auto-detection. |
| `lint-plugin.yml` | PHPCS + PHPUnit + ESLint linting. |

---

## 🧩 How to Use These Workflows in a Plugin Repo

Reusable workflows **do not define triggers**.
Each plugin repository defines **when** workflows run, and this repo defines **how** they run.

### Example: Build workflow

```yaml
name: Build Plugin

on:
  push:
    branches:
      - testing
  workflow_dispatch:

jobs:
  build:
    uses: christophherr/ch-actions/.github/workflows/build-plugin.yml@main
```

### Example: Release workflow

```yaml
name: Release Plugin

on:
  push:
    branches:
      - release

jobs:
  release:
    uses: christophherr/ch-actions/.github/workflows/release-plugin.yml@main
    secrets:
      WP_UPLOADER_USERNAME: ${{ secrets.WP_UPLOADER_USERNAME }}
      WP_UPLOADER_PASSWORD: ${{ secrets.WP_UPLOADER_PASSWORD }}
```

### Example: Test workflow

```yaml
name: Test Plugin

on:
  pull_request:
  push:
    branches:
      - main
      - develop
      - testing

jobs:
  tests:
    uses: christophherr/ch-actions/.github/workflows/test-plugin.yml@main
```

---

## 🔐 Required Secrets (per plugin repo)

| Secret | Purpose |
|--------|---------|
| `WP_ORG_USERNAME` | WordPress.org username for deploys |
| `WP_ORG_PASSWORD` | WordPress.org password for deploys |
| `GITHUB_TOKEN` | Provided automatically by GitHub |

---

## 📁 Required Files in Plugin Repos

These workflows auto-detect tests based on file presence:

| File | Purpose |
|------|---------|
| `phpunit.unit.xml` | Enables PHPUnit unit tests |
| `phpunit.integration.xml` | Enables integration tests |
| `jest.config.js` | Enables JS tests |
| `package.json` | Enables npm + Jest |
| `.zipignore` | Required for build + release ZIPs |

If a file is missing, the corresponding job is skipped automatically.

---

## 🧭 Versioning & CI Policies

See:

- [VERSION_POLICY.md](VERSION_POLICY.md)
- [WORKFLOW_GUIDE.md](WORKFLOW_GUIDE.md)

---

## 🤖 Dependabot

All repositories should include a `.github/dependabot.yml` file to keep:

- GitHub Actions
- Composer dependencies
- npm dependencies

up to date automatically.

---

## 🔄 Updating Workflows

When a workflow in this repo is updated:

- All plugin repositories referencing `@main` automatically receive the update
- No changes are required in child repos
- Breaking changes must be documented in `CHANGELOG.md` (if present)

---

## 🛠 Contributing

Changes to workflows must be made via pull request.
All PRs must pass CI before merging.

---

## 📝 License

Internal use only. Not licensed for public distribution.

