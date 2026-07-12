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
- Publish dry run — `cargo publish --dry-run` (library crates, opt-in)
- Release tag check — verifies the `Cargo.toml` version is unpublished on
  crates.io and higher than the highest existing git tag (opt-in)

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
      rust-toolchain: nightly-2026-01-22  # Optional: default is nightly-2026-01-22
      bevy-lint-enabled: true              # Optional: default is true
      bevy-lint-rev: 02112fe3d3de6d1369892ec7a8ea39a1143d2977  # Optional: default shown
      forbidden-attributes-check: true     # Optional: default is true
      publish-checks: true                 # Optional: default is false — enable for library crates
      crate-tag-prefix: v                  # Optional: default is "v" (=> tags like v1.2.3)
      release-tag-check-strict: false      # Optional: default is false — warn (not fail) when
                                           #   Cargo.toml version isn't above the highest git tag
```

The publish dry run and release tag check only run when `publish-checks: true`.
Enable them for **library crates** you publish to crates.io. The release tag
check hard-fails when the `Cargo.toml` version is already published on crates.io
(a genuine blocker), and — by default — only *warns* when the version isn't
higher than the highest existing git tag. Set `release-tag-check-strict: true`
to make that condition fail CI too.

### bevy-0.19-ci.yml

Same jobs as the 0.18 variant, but targeting Bevy 0.19.

Bevy 0.19 uses `cfg_select`, which stabilized during the Rust 1.95 cycle. This
variant therefore **defaults the main pipeline to `stable`** and keeps the
`bevy-lint` job on its own nightly `bevy-lint-toolchain`. Because no tagged
`bevy_lint` release supports Bevy 0.19 yet, `bevy-lint` is pinned to a
`TheBevyFlock/bevy_cli` **main revision** (with its matching
`nightly-2026-04-16` toolchain) that can compile Bevy 0.19. Bump
`bevy-lint-rev`/`bevy-lint-toolchain` to a tagged release once one ships.

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
    uses: MolecularSadism/msg_ci_templates/.github/workflows/bevy-0.19-ci.yaml@main
```

**Customization:**

```yaml
jobs:
  ci:
    uses: MolecularSadism/msg_ci_templates/.github/workflows/bevy-0.19-ci.yaml@main
    with:
      rust-toolchain: stable               # Optional: default is stable
      bevy-lint-enabled: true              # Optional: default is true
      bevy-lint-toolchain: nightly-2026-04-16  # Optional: nightly required by the pinned bevy_lint
      bevy-lint-rev: 6d3f8c9b2c98b57276b81eae5058ca31eaea279d  # Optional: bevy_cli main rev (Bevy 0.19)
      forbidden-attributes-check: true     # Optional: default is true
```

The same `publish-checks`, `crate-tag-prefix`, and `release-tag-check-strict`
inputs documented for the 0.18 variant also apply here.

### release.yaml

Tag and publish a release for a library crate. Reads the version from
`Cargo.toml`, validates it (unpublished on crates.io, higher than the highest
existing git tag, tag does not already exist), then **publishes to crates.io**,
**creates and pushes the tag** `<prefix><version>` (e.g. `v1.2.3`), and
**creates a GitHub release** for that tag.

Trigger it manually so releases are deliberate. Add a `CARGO_REGISTRY_TOKEN`
repository secret (your crates.io API token) before using it.

**Usage:**

```yaml
name: Release

on:
  workflow_dispatch:

jobs:
  release:
    uses: MolecularSadism/msg_ci_templates/.github/workflows/release.yaml@main
    secrets:
      CARGO_REGISTRY_TOKEN: ${{ secrets.CARGO_REGISTRY_TOKEN }}
    with:
      rust-toolchain: nightly-2026-01-22  # Optional: default is nightly-2026-01-22
      crate-tag-prefix: v                  # Optional: default is "v"
      all-features: true                   # Optional: default is true
```

To release: bump the version in `Cargo.toml`, merge to `main`, then run the
**Release** workflow from the Actions tab. It publishes the crate, creates
`vX.Y.Z`, and cuts the matching GitHub release.

## License

MIT OR Apache-2.0
