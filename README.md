# Reusable CI/CD Workflows (`kazlim-code/ci-workflows`)

Public reusable `workflow_call` pipelines pinned by flat semver tags (e.g., `@v1.0.1`) so Dependabot (`package-ecosystem: "github-actions"`) can open automated PRs across all caller repositories.

> [!IMPORTANT]
> **Visibility Rule**: On personal GitHub accounts (`kazlim-code`), reusable workflows called by public repositories **must** reside in a **public** repository.

## Available Reusable Workflows (`@v1.0.1`)

| Workflow | Purpose | Required Caller Permissions |
| :--- | :--- | :--- |
| `.github/workflows/gleam-ci.yml@v1.0.1` | Gleam `format --check` + `test`, caching `~/.cache/gleam` & `build` on `manifest.toml` | `contents: read` |
| `.github/workflows/gleam-pages-deploy.yml@v1.0.1` | Two-job Gleam static MPA build + GitHub Pages deploy (`environment: github-pages` inside deploy job) | `contents: read`, `pages: write`, `id-token: write` |
| `.github/workflows/node-ci.yml@v1.0.1` | Node/TS lint, test & build reading `.tool-versions` or defaulting to `'lts/*'` | `contents: read` |

> [!NOTE]
> **Gleam Version & `rebar3` Inputs**: `gleam-ci.yml` and `gleam-pages-deploy.yml` have no hardcoded OTP/Gleam defaults (`otp_version` and `gleam_version` default to `''`). Supply `version_file: .tool-versions` (listing `erlang`, `gleam`, and `rebar` when Erlang/rebar3 deps exist) **or** explicit `otp_version` + `gleam_version` (e.g., `"28.0"` or `">0"`). `rebar3_version` defaults to `''` (skipped for pure-Gleam projects); pass `"3.x"` or an exact pin when compiling Erlang/rebar3 dependencies.

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
    uses: kazlim-code/ci-workflows/.github/workflows/gleam-ci.yml@v1.0.1
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
    uses: kazlim-code/ci-workflows/.github/workflows/gleam-pages-deploy.yml@v1.0.1
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
> **Immutable Tag Hygiene**: Enable a repository tag protection ruleset on `kazlim-code/ci-workflows` for `v*` so published `@v1.0.1` tags cannot be moved or deleted, and prefer 40-character commit SHAs for maximum cryptographic pinning.
