# aisix — AI Gateway

> A single-binary, Rust-native AI gateway. OpenAI-compatible proxy + Admin API + Admin UI.
> Config lives in etcd. Lock-free reads. First-class streaming. >90% E2E coverage gate.

**Status:** scaffold (PR #1). Features are being delivered incrementally per [the plan](https://github.com/moonming/ai-gateway/issues).

## What it is

`aisix` is an AI inference gateway in the spirit of [LiteLLM](https://github.com/BerriAI/litellm) / [Portkey](https://github.com/Portkey-AI/gateway), rewritten in Rust for low cold-start, native streaming, and a single static binary.

- **Proxy API (`:3000`)** — OpenAI-compatible `/v1/chat/completions`, `/v1/embeddings`, `/v1/models`, `/v1/messages` (Anthropic native), plus passthrough
- **Admin API (`:3001`)** — CRUD for models, API keys, teams, budgets, credentials, guardrails, fallbacks; playground proxy; OpenAPI (Scalar) at `/openapi`; embedded SPA at `/ui`
- **Config store** — etcd with watch-driven, lock-free `ArcSwap` snapshot
- **Rate limiting** — two-phase (RPM pre-commit + TPM post-deduct) with concurrency semaphore
- **Observability** — Prometheus + OTLP (traces/metrics/logs) + structured access logs + Langfuse

## Workspace

```
crates/
├── aisix-core                 Config, Snapshot, ResourceEntry, errors
├── aisix-etcd                 ConfigProvider, watch supervisor
├── aisix-gateway              Hub & Bridge, SSE parser, provider trait
├── aisix-provider-openai
├── aisix-provider-anthropic
├── aisix-provider-gemini
├── aisix-provider-deepseek
├── aisix-proxy                /v1/* handlers + middleware
├── aisix-admin                CRUD + playground + UI embed + OpenAPI
├── aisix-obs                  tracing, metrics, access log
├── aisix-ratelimit            fixed-window + semaphore
├── aisix-cache                in-mem + redis + qdrant
├── aisix-guardrails           pre/during/post hooks
└── aisix-server               single binary — bootstrap + CLI
ui/                            React 18 + Vite + TS + Tailwind v4
```

## Development

Prerequisites: Rust toolchain (pinned in `rust-toolchain.toml`), Node 20+, pnpm, Docker (for etcd).

```bash
# Rust workspace
cargo check --workspace
cargo fmt --check
cargo clippy --workspace -- -D warnings
cargo test --workspace

# Admin UI
cd ui && pnpm install && pnpm dev

# Coverage (matches CI gate)
cargo install cargo-llvm-cov
cargo llvm-cov --workspace --lcov --output-path lcov.info

# Run (scaffold — full startup arrives in PR #5)
cargo run -p aisix-server -- --config config.example.yaml
```

## License

MIT — see [LICENSE](LICENSE).
