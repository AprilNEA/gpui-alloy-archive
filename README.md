> [!IMPORTANT]
> Remove this line to confirm you've reviewed this PR before submitting.

# GPUI Alloy

A maintained downstream of [Zed GPUI](https://github.com/zed-industries/zed/tree/main/crates/gpui), with tracked patches and upstream contributions.

Alloy retains the complete Zed repository and history. GPUI crates keep their upstream names and paths. The initial baseline is `4c841aaf1c4fa613e89a5d77096523d0ff593b56`.

## Status

The first locally validated snapshot is `gpui-alloy/20261003.1`. All eleven patch topics are integrated and imported into Cupertino. Branches, records, and the snapshot tag have not been pushed. See [ALLOY.md](ALLOY.md#首个已验收快照) for fixed commits, checks, and platform limits.

The default branch, `main`, contains the project README and maintenance ledger. The integration branch is `downstream/gpui`. Unmodified Zed commits are tracked through `upstream/main`. Existing upstream pull requests remain in [AprilNEA/zed](https://github.com/AprilNEA/zed).

See [ALLOY.md](ALLOY.md) for the maintenance policy, patch ledger, validation requirements, and initial push instructions. See the [upstream GPUI README](crates/gpui/README.md) for framework documentation.

## Source exports

Use Python 3.12 or later and Git. Export to a new directory. This example exports the validated snapshot:

```sh
ALLOY_U=4c841aaf1c4fa613e89a5d77096523d0ff593b56
ALLOY_CANDIDATE=19fc9968f689ccce2b9e31c2eb9a76ec442333dc
python3 script/gpui_snapshot.py export --upstream "$ALLOY_U" --revision "$ALLOY_CANDIDATE" --snapshot /tmp/gpui-alloy-20261003.1
```

The export retains original crate names and paths. `ALLOY-SNAPSHOT.json` records both commits, the exporter hash, package paths, and file hashes. See [ALLOY.md](ALLOY.md#导出工具和消费方来源检查) for the consumer check and dependency configuration limits.

## Licensing

Each crate retains its upstream license. GPUI uses [Apache-2.0](crates/gpui/LICENSE-APACHE); the full Zed repository also contains [GPL-3.0-or-later](LICENSE-GPL) code. Refer to each crate's manifest and license files.
