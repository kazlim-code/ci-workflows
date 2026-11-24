# Reusable CI/CD Workflows (`kazlim-code/ci-workflows`)

Public reusable `workflow_call` pipelines pinned by flat semver tags (e.g., `@v1.0.0`) so Dependabot (`package-ecosystem: "github-actions"`) can open automated PRs across all caller repositories.

> [!IMPORTANT]
> **Visibility Rule**: On personal GitHub accounts (`kazlim-code`), reusable workflows called by public repositories **must** reside in a **public** repository.

## Available Reusable Workflows (`@v1.0.0`)

| Workflow | Purpose | Required Caller Permissions |
| :--- | :--- | :--- |
| `.github/workflows/gleam-ci.yml@v1.0.0` | Gleam `format --check` + `test`, caching `~/.cache/gleam` & `build` on `manifest.toml` | `contents: read` |
| `.github/workflows/gleam-pages-deploy.yml@v1.0.0` | Two-job Gleam static MPA build + GitHub Pages deploy (`environment: github-pages` inside deploy job) | `contents: read`, `pages: write`, `id-token: write` |
| `.github/workflows/node-ci.yml@v1.0.0` | Node/TS lint, test & build reading `.tool-versions` or defaulting to `'lts/*'` | `contents: read` |

## Example: Gleam CI + Tag-Triggered GitHub Pages (`dev_ing_website`)

### `.github/workflows/ci.yml`
```yaml
name: CI
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: read

jobs:
  verify:
    uses: kazlim-code/ci-workflows/.github/workflows/gleam-ci.yml@v1.0.0
    with:
      version_file: .tool-versions
      run_format_check: true
```

### `.github/workflows/pages-release.yml`
```yaml
name: Release Pages
on:
  push:
    tags:
      - 'v*.*.*'
  workflow_dispatch:

permissions:
  contents: read

concurrency:
  group: pages
  cancel-in-progress: false

jobs:
  deploy-pages:
    if: startsWith(github.ref, 'refs/tags/v')
    uses: kazlim-code/ci-workflows/.github/workflows/gleam-pages-deploy.yml@v1.0.0
    permissions:
      contents: read
      pages: write
      id-token: write
    with:
      version_file: .tool-versions
      export_module: "dev_ing_website/export"
      base_path_env: "DEV_ING_BASE_PATH"
      output_dir: "dist"
```

> [!TIP]
> **Immutable Tag Hygiene**: Enable a repository tag protection ruleset on `kazlim-code/ci-workflows` for `v*` so published `@v1.0.0` tags cannot be moved or deleted, and prefer 40-character commit SHAs for maximum cryptographic pinning.
