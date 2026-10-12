# TPT Sovereign Agent Platform — TODO

Tick items off as you go (`- [x]`). Source of truth: [spec.txt](spec.txt), listing copy: [gumroad.md](gumroad.md).

## Decisions locked in
- **Foundation crates:** all in GitHub org `tpt-solutions` (tpt-anima, tpt-keystone-db, tpt-crypto, tpt-shatter, tpt-appfront, tpt-primitives). Consumed as **git deps pinned to commit SHA**; `cargo vendor` added for release builds.
- **Shell:** pure AppFront native (no Tauri). Fallback only if Phase 0 spike fails.
- **LLM:** `LlmBackend` trait + OpenAI-compatible localhost adapter (Ollama / llama.cpp / LM Studio). Embedded inference is post-v1.
- **Agent loop (MVP):** tool-call loop, SSE memory retrieval into prompt, streaming output in UI, deterministic mock backend for demos/CI.
- **No license key / no DRM:** the binary is not gated. Instead the app links to https://tptsolutions.co.nz/tools (About screen, README, guide) as a marketing lead. Link is a plain user-clicked link; no network calls from the app.
- **Platforms:** Windows + Linux only. macOS is out of scope.
- **tpt-concord:** dropped.
- **Licence:** app is MIT OR Apache-2.0; every dependency must be compatible with both.

---

## Phase 0 — Decisions, spikes, repo skeleton
- [x] Init git repo, `.gitignore`, `LICENSE-MIT`, `LICENSE-APACHE`, README stub (repo initialised, no commits yet)
- [x] Cargo workspace skeleton (`sovereign-core`, `sovereign-llm`, `sovereign-tools`, `sovereign-store`, `sovereign-archive`, `sovereign-license`, `sovereign-ui`, `sovereign-app`)
- [x] Pin Rust toolchain (`rust-toolchain.toml`, pinned to 1.97.0; MSRV 1.85 in workspace)
- [ ] Add git deps for the 6 foundation crates; confirm workspace compiles on Windows and Linux — *partial: 5 of 6 pinned on `sovereign-core` and compiling on Windows. `tpt-shatter` blocked (repo has no crate yet). Linux not verified.*
- [ ] Verify each foundation crate's actual license (Keystone shows "NOASSERTION" on GitHub; LICENSE-MIT + LICENSE-APACHE present) and record in `docs/dependency-audit.md` — *partial: 5 of 6 recorded; shatter blocked.*
- [ ] Add `cargo-deny` config enforcing MIT/Apache-2.0-compatible licenses (reject Apache-2.0-only, GPL, etc.) + run in CI — *partial: `deny.toml` written and `cargo deny check licenses` passes; CI not set up.*
- [ ] **Spike:** AppFront native desktop shell opens a window, renders a list, handles input on Windows and Linux (go/no-go for pure-AppFront)
- [ ] **Spike:** create an Anima agent end-to-end (CK lifecycle, SSE write/read, ZKPA identity) from our own code
- [ ] **Spike:** embedded Keystone DB opens a per-colony local file, runs a query, no server process or network
- [ ] **Spike:** streaming chat completion from Ollama via OpenAI-compatible endpoint
- [ ] Decide & document how much Anima surface the UI exposes (default: hide INP/ZKPA internals behind plain-language concepts)
- [ ] Write `docs/architecture.md` (layers, crate boundaries, data flow, threat model)
- [ ] CI: build + test + clippy + fmt + cargo-deny on Windows and Linux

## Phase 1 — Core domain & storage
- [ ] Domain types mapped onto `tpt-primitives` (Identity, Capability, Intent, Evidence, Claim): Colony, Agent, MemoryScope, Session, ActivityEvent
- [ ] Colony = one directory with one Keystone DB; open/create/close/lock semantics
- [ ] Schema + migrations for agents, scopes, sessions, activity log, settings
- [ ] Agent CRUD (create / edit / delete) with validation and soft-delete or cascade rules for memory
- [ ] Memory scopes: define scope model, assign scopes to agents, enforce at read/write via SSE
- [ ] Agent identity: ephemeral ZKPA identity issued per agent/session; provable via Anima
- [ ] Capability system: define capability set (fs read/write paths, Keystone query, HTTP host allowlist, memory scopes, negotiate)
- [ ] Capability grants stored per agent; deny-by-default; capability tokens via `tpt-shatter` where applicable
- [ ] Shatter integration for high-sensitivity fields (mark fields/scopes as sensitive; fragment at rest)
- [ ] Append-only, tamper-evident activity log (hash-chained) written for every tool call, grant, negotiation and lifecycle event
- [ ] Unit + property tests for permissions (no escalation, default deny)

## Phase 2 — Agent runtime
- [ ] `LlmBackend` trait (chat, streaming, tool-call schema, cancellation, timeouts)
- [ ] OpenAI-compatible localhost adapter (configurable base URL + model; model list discovery)
- [ ] Mock/scripted backend for deterministic demos, tests and CI
- [ ] Refuse non-loopback LLM endpoints unless user explicitly allows (keeps "no cloud" promise honest)
- [ ] Agent lifecycle via `tpt-anima-ck` (start, pause, resume, stop, crash recovery); scheduling & isolation
- [ ] Prompt assembly: system prompt + agent definition + retrieved attended memory (SSE) + conversation
- [ ] Memory write-back: what gets stored as attended memory after each turn
- [ ] Tool-call loop: parse tool calls → capability check → execute → feed result → continue; max-iteration and budget limits
- [ ] Tool: local filesystem (scoped to granted paths, canonicalised, no symlink/`..` escape)
- [ ] Tool: Keystone query (scoped to granted tables/scopes; read-only vs read-write)
- [ ] Tool: HTTP (every request user-approved; per-host allowlist option; show full URL/method/body in approval dialog)
- [ ] Approval flow primitive (pending → approved/denied → logged) reusable by UI
- [ ] Streaming token output surfaced to UI via channel/event stream

## Phase 3 — Multi-agent negotiation
- [ ] Negotiation session model (participants, goal/intent, rounds, outcome, status)
- [ ] Integrate `tpt-anima-inp` for intent expression and outcome negotiation
- [ ] Session runner: start/stop/timeout, max rounds, deadlock handling
- [ ] Capability checks on who may join/propose; all messages logged
- [ ] Persist session transcript + outcome to Keystone; link to activity log
- [ ] Integration tests with mock backend: 2-agent and 3-agent negotiations converge deterministically

## Phase 4 — Desktop UI (AppFront native)
- [ ] App shell: window, navigation, theming (dark default), keyboard navigation, accessibility basics
- [ ] First-run: create/open colony, choose data location, point at LLM endpoint, "test connection"
- [ ] Colony picker / recent colonies
- [ ] Agents view: list, create, edit, delete (with confirm)
- [ ] Agent editor: name, role/system prompt, model, memory scopes, capability grants (plain-language labels)
- [ ] Run view: chat with an agent, streaming output, tool-call display, stop button
- [ ] Approval dialogs (HTTP, risky fs ops) — clear, un-spoofable, cannot be auto-dismissed
- [ ] Negotiation sessions view: pick agents + goal, watch rounds live, see outcome
- [ ] Activity log / inspection UI: filter by agent/type/time, view tool calls, verify hash chain
- [ ] Memory inspector (basic list/search of an agent's attended memory per scope)
- [ ] Settings: LLM, data location, About + licence status
- [ ] Error states + empty states everywhere (no LLM running, DB locked, bad archive, etc.)
- [ ] Included example colony loads from a menu item
- [ ] Usability pass against success criterion: non-expert builds a useful multi-agent setup in < 30 min (test with 2–3 people)

## Phase 5 — Export / import (signed + encrypted archive)
- [ ] Archive format spec (`docs/archive-format.md`): manifest, version, chunking, signatures, KDF params
- [ ] Export: colony → archive; encrypt with user passphrase (KDF + AEAD via `tpt-crypto`); sign for integrity
- [ ] Import: verify signature/integrity before decrypting/applying; reject tampered or wrong-version archives with clear errors
- [ ] Import into new colony (never silently overwrite existing); handle identity re-issuance/rebinding
- [ ] Round-trip tests: export → import yields identical agents, memory, logs; corruption/bit-flip tests fail cleanly
- [ ] Cross-OS test: export on Windows, import on Linux (and reverse)
- [ ] UI for export/import with passphrase strength guidance and a "no recovery if forgotten" warning

## Phase 6 — Marketing link (replaces licensing)
- [ ] About screen with link to https://tptsolutions.co.nz/tools (opens in system browser on click only)
- [x] Same link in README (quick-start guide PDF pending, Gumroad listing already updated)
- [ ] Optional: "More TPT tools" item in Help menu
- [ ] Confirm no automatic requests to the site (no telemetry, no update check, no tracking params that identify users)
- [x] Use the plain URL https://tptsolutions.co.nz/tools everywhere (no ref/UTM params) — *checked in README and gumroad.md*

## Phase 7 — Hardening, performance, QA
- [ ] Threat-model review (agent escapes, prompt injection → tool abuse, path traversal, archive attacks)
- [ ] Prompt-injection mitigations for tool results (tagged untrusted content; capability checks independent of model output)
- [ ] Fuzz archive parser and tool-call parser
- [ ] Verify zero network traffic except user-approved HTTP tools and the configured loopback LLM (packet capture test)
- [ ] `cargo audit` + `cargo deny` clean; SBOM generated
- [ ] Measure binary size and cold start; set budgets and compare to comparable desktop tools
- [ ] Soak test: long-running multi-agent colony, memory/leak check
- [ ] Test on min spec (8 GB RAM, Windows 10/11, modern Linux)
- [ ] End-to-end offline test with network disabled

## Phase 8 — Packaging & release (Windows + Linux)
- [ ] Reproducible release build using vendored deps (`cargo vendor`, `--locked`)
- [ ] Windows: .exe + installer, code-signing certificate acquired and applied
- [ ] Linux: AppImage and/or .deb; checksums + detached signature
- [ ] Example agent colony (works with mock backend and with a local model)
- [ ] Quick-start guide PDF (install, point at Ollama, create agent, negotiation, export/import)
- [ ] Release zip layout: binaries + docs
- [x] **Update [gumroad.md](gumroad.md):** macOS and licence-key references removed, tools link added
- [ ] Confirm price ($279), refund policy, tags
- [ ] Create Gumroad product, upload zip, configure delivery, post-purchase email
- [ ] Define error-report submission channel (e.g. email address / form) referenced in listing & guide
- [ ] Open-source repo public: README with build-from-source steps, CONTRIBUTING, SECURITY.md, dual-licence notices
- [ ] Final clean-machine install test on Windows and Linux from the Gumroad zip
- [ ] Tag v1.0 and publish

## Post-v1 (no commitment — from spec)
- [ ] Visual negotiation graph editor
- [ ] Memory visualisation
- [ ] WASM tool sandboxing (Keystone WASM UDFs / Boxcar)
- [ ] Agent templates
- [ ] Headless / CLI mode
- [ ] Embedded LLM inference (e.g. via tpt-local-ai / llama.cpp)
- [ ] Stronger formal claims integration

## Open items to resolve while building
- [ ] Which Anima crates the app depends on directly vs via the umbrella `tpt-anima` crate
- [ ] Error-report channel and what "approved" means operationally
