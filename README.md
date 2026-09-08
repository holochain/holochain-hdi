# holochain-hdi

The Holochain Deterministic Integrity SDK (`hdi`) and the crates it is built
from. These crates were split out of
[holochain/holochain](https://github.com/holochain/holochain) so that the HDI
can be versioned and released on its own
([holochain/holochain#5400](https://github.com/holochain/holochain/issues/5400)).

| Crate                        | Licence    | Purpose                                     |
| ---------------------------- | ---------- | ------------------------------------------- |
| `hdi`                        | CAL-1.0    | Integrity zome SDK, compiled to WASM        |
| `hdk_derive`                 | Apache-2.0 | Derive macros shared by the HDI and the HDK |
| `holochain_integrity_types`  | Apache-2.0 | Types available to integrity zomes          |
| `holo_hash`                  | Apache-2.0 | Hash types and hashing helpers              |
| `holochain_timestamp`        | Apache-2.0 | Microsecond timestamp type                  |
| `holochain_secure_primitive` | Apache-2.0 | Secure primitive macros                     |
| `holochain_util`             | Apache-2.0 | Shared utility functions                    |
| `fixt`                       | Apache-2.0 | Fixture generation used in tests            |

All crates share one version, set in the root `Cargo.toml`.

## Development

Install [cargo-make](https://github.com/sagiegurari/cargo-make) and
[taplo](https://taplo.tamasfe.dev/), then:

```bash
cargo make static   # formatting, clippy, docs, WASM build
cargo make test     # tests with all features
cargo make verify   # both
```

## Releasing

Releases are prepared and published by the
[holochain release integration](https://github.com/holochain/release-integration).
Run the "Prepare a release" workflow, review the PR it opens, and merge it.
Publishing to crates.io happens on merge.

## History

Commits before the split were extracted from `holochain/holochain` with
`git filter-repo`; blame and log work across the split.
