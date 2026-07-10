# msg_ci_templates

[![CI](https://github.com/MolecularSadism/msg_ci_templates/workflows/CI/badge.svg)](https://github.com/MolecularSadism/msg_ci_templates/actions)
[![License](https://img.shields.io/badge/license-MIT%2FApache--2.0-blue.svg)](https://github.com/MolecularSadism/msg_ci_templates#license)
[![Bevy](https://img.shields.io/badge/Bevy-0.18-blue.svg)](https://bevyengine.org/)
[![Rust](https://img.shields.io/badge/rust-2024%20edition-orange.svg)](https://www.rust-lang.org/)

Reusable GitHub Actions CI workflows for Bevy projects.

## Available Workflows

### bevy-0.18-ci.yml

Comprehensive CI workflow for Bevy 0.18 projects.

**Jobs:**
- Format checking with `cargo fmt`
- Forbidden attributes checking (no_run, ignore)
- Documentation building with `cargo doc`
- Clippy linting (standard + pedantic)
- Bevy lint checks
- Tests (unit, integration, doc)

**Usage:**

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

concurrency:
  group: ${{ github.workflow }}-${{ github.ref || github.run_id }}
  cancel-in-progress: true

jobs:
  ci:
    uses: MolecularSadism/msg_ci_templates/.github/workflows/bevy-0.18-ci.yaml@main
```

**Customization:**

```yaml
jobs:
  ci:
    uses: MolecularSadism/msg_ci_templates/.github/workflows/bevy-0.18-ci.yaml@main
    with:
      rust-toolchain: stable               # Optional: default is stable
      cargo-features: '--all-features'     # Optional: feature flags for build/test/clippy/doc (use '' for default features)
      bevy-lint-enabled: true              # Optional: default is true (requires a nightly toolchain)
      bevy-lint-toolchain: nightly-2026-01-22  # Optional: nightly used only by the bevy-lint job
      bevy-lint-rev: 02112fe3d3de6d1369892ec7a8ea39a1143d2977  # Optional: default shown
      forbidden-attributes-check: true     # Optional: default is true
```

> **Note:** The main pipeline (format, docs, clippy, tests) runs on `stable` by
> default. The `bevy-lint` job requires a nightly toolchain (cranelift codegen
> backend), so it uses its own `bevy-lint-toolchain` input. Disable it with
> `bevy-lint-enabled: false` if you want a fully-stable pipeline.

## License

MIT OR Apache-2.0
