# TPT Sovereign Agent Platform

An offline-first desktop platform for creating, running and composing autonomous agents that you fully own.

- Agents with attended memory, verifiable identity and capability-based permissions
- Agent-to-agent negotiation sessions
- Local database per colony, no cloud, no telemetry, no accounts
- Colony export and import as a signed, encrypted archive

**Status:** early development (skeleton only). See [todo.md](todo.md) for progress and [spec.txt](spec.txt) for the design.

## Build from source

Requirements:

- Rust 1.85 or newer (the toolchain is pinned in [rust-toolchain.toml](rust-toolchain.toml))
- Windows 10/11 or a modern Linux desktop
- Git (the foundation crates are fetched as git dependencies pinned to commit SHAs)

```sh
git clone https://github.com/tpt-solutions/tpt-app-sovereign-agent
cd tpt-app-sovereign-agent
cargo build --release --locked
```

## Workspace layout

| Crate | Responsibility |
|---|---|
| `sovereign-core` | Domain types: colony, agent, memory scope, session, activity event |
| `sovereign-llm` | `LlmBackend` trait and local OpenAI-compatible adapter |
| `sovereign-tools` | Filesystem, database and HTTP tools behind capability checks |
| `sovereign-store` | Keystone-backed colony storage and migrations |
| `sovereign-archive` | Signed, encrypted colony export and import |
| `sovereign-license` | Licence and identity helpers |
| `sovereign-ui` | AppFront native user interface |
| `sovereign-app` | Desktop binary |

## Licence

Dual-licensed under either of

- [MIT licence](LICENSE-MIT)
- [Apache licence, version 2.0](LICENSE-APACHE)

at your option. Dependencies must be compatible with both; this is enforced with `cargo deny check licenses` using [deny.toml](deny.toml).

## Support

Documentation and a quick-start guide are at https://tptsolutions.co.nz/tools
