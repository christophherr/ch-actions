# Workflow Guide

This guide explains each reusable workflow in the `ch-actions` repository, what it does, and how plugin repositories should use it. Each workflow is designed to be called from a plugin repository, which defines its own triggers (`on:`).

---

## 🧱 1. build-plugin.yml

### Purpose

Builds a plugin ZIP for testing or QA using `.distignore`.

### Includes

- Composer install (cached)
- npm install (cached)
- Asset build
- POT generation
- ZIP creation

### Inputs

None

### Secrets

None

### Usage (child repo)

```yaml
jobs:
  build:
    uses: honorswp/ch-actions/.github/workflows/build-plugin.yml@main
```

---

## 🚀 2. release-plugin.yml

### Purpose

Full release pipeline:

- Build
- Tag
- GitHub Release
- WordPress.org deploy

### Inputs

| Name | Description |
|------|-------------|
| `slug` | WP.org plugin slug |

### Secrets

| Secret | Purpose |
|--------|---------|
| `WP_ORG_USERNAME` | WP.org deploy username |
| `WP_ORG_PASSWORD` | WP.org deploy password |

### Usage (child repo)

```yaml
jobs:
  release:
    uses: honorswp/ch-actions/.github/workflows/release-plugin.yml@main
    secrets:
      WP_UPLOADER_USERNAME: ${{ secrets.WP_UPLOADER_USERNAME }}
      WP_UPLOADER_PASSWORD: ${{ secrets.WP_UPLOADER_PASSWORD }}
```

---

## 🌍 3. wporg-release.yml

### Purpose

Standalone WordPress.org deploy workflow.
Useful when you want to deploy without building a GitHub Release.

### Inputs

| Name | Description |
|------|-------------|
| `slug` | WP.org plugin slug |

### Secrets

Same as `release-plugin.yml`.

### Usage (child repo)

```yaml
jobs:
  deploy:
    uses: honorswp/ch-actions/.github/workflows/wporg-release.yml@main
    with:
      slug: my-plugin
    secrets:
      WP_ORG_USERNAME: ${{ secrets.WP_ORG_USERNAME }}
      WP_ORG_PASSWORD: ${{ secrets.WP_ORG_PASSWORD }}

```

---

## 📝 4. wporg-readme.yml

### Purpose

Updates plugin readme + assets on WP.org (no build, no release).

### Inputs

| Name | Description |
|------|-------------|
| `slug` | WP.org plugin slug |

### Secrets

Same as above.

### Usage (child repo)

```yaml
jobs:
  readme:
    uses: honorswp/ch-actions/.github/workflows/wporg-readme.yml@main
    with:
      slug: my-plugin
    secrets:
      WP_ORG_USERNAME: ${{ secrets.WP_ORG_USERNAME }}
      WP_ORG_PASSWORD: ${{ secrets.WP_ORG_PASSWORD }}

```

---

## 🧪 5. test-plugin.yml

### Purpose

Runs the full test suite:

- PHPUnit unit tests
- PHPUnit integration tests
- Jest JS tests

### Auto‑detection

Jobs only run if the plugin repo contains:

| File | Enables |
|------|---------|
| `phpunit.unit.xml` | Unit tests |
| `phpunit.integration.xml` | Integration tests |
| `jest.config.js` + `package.json` | JS tests |

If a file is missing, the corresponding job is skipped automatically.

### Includes

- Composer caching
- npm caching
- WP test suite caching
- Xdebug coverage
- Jest coverage (uploaded as artifact)

### Inputs

None

### Secrets

None

### Usage (child repo)

```yaml
jobs:
  tests:
    uses: honorswp/ch-actions/.github/workflows/test-plugin.yml@main
```

---

## 🔍 6. lint-plugin.yml

### Purpose

Runs:

- PHPCS
- PHPUnit (lint mode)
- ESLint

### Inputs

None

### Secrets

None

### Usage (child repo)

```yaml
jobs:
  lint:
    uses: honorswp/ch-actions/.github/workflows/lint-plugin.yml@main
```

---

## 🧩 Required Files in Plugin Repos

| File | Purpose |
|------|---------|
| `.distignore` | Required for build + release |
| `phpunit.unit.xml` | Optional |
| `phpunit.integration.xml` | Optional |
| `jest.config.js` | Optional |
| `package.json` | Optional |

---

## 🧭 Best Practices

- Keep workflows in plugin repos minimal
- Always reference reusable workflows via `@main`
- Let Dependabot update external actions weekly
- Never duplicate workflow logic in plugin repos
- Use `.distignore` to control what goes into release ZIPs
