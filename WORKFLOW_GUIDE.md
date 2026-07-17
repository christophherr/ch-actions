# Workflow Guide

This guide explains each reusable workflow in the `ea-actions` repository, what it does, and how plugin repositories should use it. Each workflow is designed to be called from a plugin repository, which defines its own triggers (`on:`).

---

## 🧱 1. build-plugin.yml

### Purpose

Builds a plugin ZIP for testing or QA using `.zipignore`.

### Includes

- Composer install (cached)
- npm install (cached)
- Asset build
- POT generation
- ZIP creation

### Inputs

| Name | Description | Default |
|------|-------------|---------|
| `php-version` | PHP version | `7.4` |
| `node-version` | Node version | `24` |
| `skip-build` | Skip npm/build step | `false` |

### Secrets

None

### Usage (child repo)

```yaml
jobs:
  build:
    uses: easily-amused/ea-actions/.github/workflows/build-plugin.yml@main
    with:
      skip-build: true # Optional: skips npm install and npm run build
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

| Name | Description | Default |
|------|-------------|---------|
| `php-version` | PHP version | `7.4` |
| `node-version` | Node version | `24` |
| `skip-build` | Skip npm/build step | `false` |

### Secrets

| Secret | Purpose |
|--------|---------|
| `WP_UPLOADER_USERNAME` | Internal site upload username |
| `WP_UPLOADER_PASSWORD` | Internal site upload password |

### Usage (child repo)
 
 ```yaml
 jobs:
   release:
     uses: easily-amused/ea-actions/.github/workflows/release-plugin.yml@main
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

| Name | Description | Default |
|------|-------------|---------|
| `slug` | WP.org plugin slug | (required) |
| `php-version` | PHP version | `7.4` |
| `node-version` | Node version | `24` |
| `skip-build` | Skip npm/build step | `false` |

### Secrets

Same as `release-plugin.yml`.

### Usage (child repo)

```yaml
jobs:
  deploy:
    uses: easily-amused/ea-actions/.github/workflows/wporg-release.yml@main
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

| Name | Description | Default |
|------|-------------|---------|
| `slug` | WP.org plugin slug | (required) |
| `ignore-other-files` | Only update assets/readme | `true` |

### Secrets

Same as above.

### Usage (child repo)

```yaml
jobs:
  readme:
    uses: easily-amused/ea-actions/.github/workflows/wporg-readme.yml@main
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
 | `phpunit.ci.xml` | Integration tests |
 | `jest.config.js` + `package.json` | JS tests |

If a file is missing, the corresponding job is skipped automatically.

### Includes

- Composer caching
- npm caching
- WP test suite caching
- Xdebug coverage
- Jest coverage (uploaded as artifact)

### Inputs

| Name | Description | Default |
|------|-------------|---------|
| `slug` | Plugin slug | (required) |
| `php-version` | PHP version | `7.4` |
| `wp-version` | WordPress version | `latest` |
| `node-version` | Node version | `24` |
| `skip-unit` | Skip PHPUnit unit tests | `false` |
| `skip-integration` | Skip PHPUnit integration tests | `false` |
| `skip-js` | Skip Jest tests | `false` |

### Secrets

None

### Usage (child repo)

```yaml
jobs:
  tests:
    uses: easily-amused/ea-actions/.github/workflows/test-plugin.yml@main
    with:
      slug: my-plugin
      wp-version: "6.5"
```

---

## 🔍 6. lint-plugin.yml

### Purpose

Runs:

- PHPCS
- PHPUnit (lint mode)
- ESLint

### Inputs

| Name | Description | Default |
|------|-------------|---------|
| `php-version` | PHP version | `7.4` |
| `node-version` | Node version | `24` |
| `skip-php-lint` | Skip PHPCS | `false` |
| `skip-js-lint` | Skip ESLint | `false` |
| `phpcs-standard` | PHPCS standard file | `phpcs.xml` |

### Secrets

None

### Usage (child repo)

```yaml
jobs:
  lint:
    uses: easily-amused/ea-actions/.github/workflows/lint-plugin.yml@main
    with:
      skip-js-lint: true
```

---

## 🧩 Required Files in Plugin Repos
 
 | File | Purpose |
 |------|---------|
 | `.zipignore` | Required for build + release |
 | `phpunit.unit.xml` | Optional |
 | `phpunit.ci.xml` | Optional |
 | `jest.config.js` | Optional |
 | `package.json` | Optional |

---

## 🧭 Best Practices

- Keep workflows in plugin repos minimal
- Always reference reusable workflows via `@main`
- Let Dependabot update external actions weekly
- Never duplicate workflow logic in plugin repos
- Use `.zipignore` to control what goes into release ZIPs

---

## 📁 .zipignore Format

The `build-plugin.yml` and `release-plugin.yml` workflows use the `zip -x@.zipignore` command to exclude files from the final archive. 

### Format Rules:
1. **One pattern per line.**
2. **Standard glob patterns** are supported (e.g., `*.log`, `temp/*`).
3. **Directory exclusion:** To exclude a directory and all its contents, use `dir_name/*`.
4. **Recursive matching:** Patterns usually match relative to the root of the ZIP.
5. **No special keywords:** Unlike `.gitattributes`, do NOT include `export-ignore`. Only the path/pattern itself.

### Moving from .gitattributes to .zipignore

If you have a `.gitattributes` file like this:
```
.github export-ignore
node_modules export-ignore
.gitignore export-ignore
```

Your `.zipignore` file should look like this:
```
.github/*
node_modules/*
.gitignore
```
*Note: Using `/*` ensures all contents of the directory are excluded.*
