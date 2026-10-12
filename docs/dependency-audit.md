# Dependency audit

Checked against the licensing rule in [spec.txt](../spec.txt) §5: every dependency must be compatible with both MIT and Apache-2.0. Enforced by `cargo deny check licenses` ([deny.toml](../deny.toml)).

Last checked: 2026-10-12, Windows 11, Rust 1.97.0.

## Foundation crates

All are pinned by commit SHA in [crates/sovereign-core/Cargo.toml](../crates/sovereign-core/Cargo.toml).

| Crate | Repo | Pinned SHA | Licence (Cargo manifest) | Status |
|---|---|---|---|---|
| `tpt-anima` (umbrella) | tpt-anima | `408c2e00` | MIT OR Apache-2.0 (workspace) | Compiles on Windows |
| `tpt-anima-ck`, `-sse`, `-inp`, `-zkpa` | tpt-anima | `408c2e00` | MIT OR Apache-2.0 (transitive) | Compiles on Windows |
| `tpt-keystone` | tpt-keystone-db | `b861edd6` | MIT OR Apache-2.0 | Compiles on Windows |
| `tpt-crypto` (+ `-core`, `-aead`, `-kem`, `-sig`, and others) | tpt-crypto | `a3bd3e31` | MIT OR Apache-2.0 (workspace) | Compiles on Windows |
| `tpt-primitives` | tpt-primitives | `8f4e27c0` | MIT OR Apache-2.0 (workspace) | Compiles on Windows |
| `tpt-appfront-core` | tpt-appfront | `61465269` | MIT OR Apache-2.0 (workspace) | Compiles on Windows. Provisional: the shell spike must confirm this is the right crate. |
| `tpt-shatter` | tpt-shatter | n/a | n/a | **Blocked.** The repo has only `spec.txt` and `todo.md`, with no crate to depend on. |

Notes:

- `tpt-keystone-db` has no root manifest. The crate is in `tpt-keystone/`, and it is referenced as a git dependency with `package = "tpt-keystone"`.
- The `tpt-keystone-db` GitHub repo reports `NOASSERTION` as its licence. The LICENSE-MIT and LICENSE-APACHE files are present, and the crate manifest declares MIT OR Apache-2.0. Re-check the repo-level licence metadata before a public release.
- Linux builds of the foundation crates have not been verified yet.

## Result of `cargo deny check licenses`

Passes. The only output is two warnings about allowances that no current dependency uses (`0BSD`, `Unicode-DFS-2016`). These are kept on purpose.
