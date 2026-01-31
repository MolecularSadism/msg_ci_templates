# msg_ci_templates

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
    uses: MolecularSadism/msg_ci_templates/.github/workflows/bevy-0.18-ci.yml@main
```

**Customization:**

```yaml
jobs:
  ci:
    uses: MolecularSadism/msg_ci_templates/.github/workflows/bevy-0.18-ci.yml@main
    with:
      rust-toolchain: nightly-2026-01-22  # Optional: default is nightly-2026-01-22
      bevy-lint-enabled: true              # Optional: default is true
      bevy-lint-rev: 02112fe3d3de6d1369892ec7a8ea39a1143d2977  # Optional: default shown
      forbidden-attributes-check: true     # Optional: default is true
```

## License

MIT OR Apache-2.0
