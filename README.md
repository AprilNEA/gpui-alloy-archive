# GPUI Alloy

A maintained downstream of [Zed GPUI](https://github.com/zed-industries/zed/tree/main/crates/gpui), with tracked patches and upstream contributions.

Alloy retains the complete Zed repository and history. GPUI crates keep their upstream names and paths. The initial baseline is `4c841aaf1c4fa613e89a5d77096523d0ff593b56`.

## Status

Repository initialization is complete locally. The Cupertino patches have not been migrated, and no validated Alloy snapshot is available. Keep using the existing Cupertino vendor until a snapshot passes both GPUI and consumer checks.

The default branch, `main`, contains the project README and maintenance ledger. The integration branch is `downstream/gpui`. Unmodified Zed commits are tracked through `upstream/main`. Existing upstream pull requests remain in [AprilNEA/zed](https://github.com/AprilNEA/zed).

See [ALLOY.md](ALLOY.md) for the maintenance policy, patch ledger, validation requirements, and initial push instructions. See the [upstream GPUI README](crates/gpui/README.md) for framework documentation.

## Licensing

Each crate retains its upstream license. GPUI uses [Apache-2.0](crates/gpui/LICENSE-APACHE); the full Zed repository also contains [GPL-3.0-or-later](LICENSE-GPL) code. Refer to each crate's manifest and license files.
