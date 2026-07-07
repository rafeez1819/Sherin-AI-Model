# SHERIN

**Edge-native, offline-first AI operating system — one model, one user, no cloud dependency.**

SHERIN is a personal AI architecture built around a simple bet: that a single-user AI system doesn't need to be a general-purpose frontier model to be genuinely useful. It needs to be *fast*, *fully local*, *auditable*, and able to grow its own knowledge base without manual re-engineering.

This repo documents the actual architecture, the reasoning behind each design decision, and — just as importantly — where the system's real strengths and real limits are. Nothing here is marketing copy.

---

## What SHERIN actually is

SHERIN is **not** a transformer-style foundation model. It's a hierarchical, deterministic routing and retrieval system:

- A spherical mesh of **Core → Layers → Areas**, each holding pre-harvested knowledge units (KUs)
- Tasks are decomposed and tagged with a compact **DNA-style identifier** that routes them through the mesh
- Only **KU-ID references** move through the reasoning pipeline — never raw content — until final assembly
- A local **KuLexicon engine** resolves KU-IDs to language on the user's own device
- The knowledge base **grows its own capacity** (new Areas, new Layers) as data fills existing structure, without manual redesign

If you're picturing GPT-style generalization to arbitrary novel queries, that's the wrong mental model. SHERIN's real strength is a domain it has actually harvested: near-instant, fully offline, privacy-preserving retrieval and composition. Outside a harvested domain, it has nothing to route to — this is a known, structural limitation, not a bug to be patched later.

---

## Core design principles

1. **Zero payload, precisely defined.** Only fixed-size IDs traverse the reasoning mesh. Content is fetched and rendered into language exactly once, at the final handoff to the user's device.
2. **Deterministic over probabilistic.** Every routing decision is auditable — you can trace exactly which Layer, Area, and KU produced a given output.
3. **Self-evolving structure.** The knowledge mesh expands ring-by-ring, Area-by-Area, only when existing capacity is actually full. No pre-provisioned, unused capacity.
4. **Single model, single user.** No multi-tenant serving, no shared cloud instance. The whole system is scoped to one person's harvested knowledge and one person's device.
5. **Honest benchmarking.** Every performance number in this repo is labeled with exactly which stage of the pipeline it measures — routing, lookup, generation, or end-to-end — because these have very different cost profiles and conflating them produces misleading claims.

---

## Repository map

| Doc | Covers |
|---|---|
| [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) | Core/Layer/Area mesh, the 3 Layer-1 bots (Safety/Planning/Execution), ring growth mechanic |
| [`docs/DNA_REQUEST_PROTOCOL.md`](docs/DNA_REQUEST_PROTOCOL.md) | How a user task becomes a routed, tagged, multi-outcome request |
| [`docs/LEXICON_ENGINE.md`](docs/LEXICON_ENGINE.md) | KuLexicon v2.0 — ID format, hash lookup, template-based generation, multilingual plan |
| [`docs/BENCHMARKS.md`](docs/BENCHMARKS.md) | What's actually been measured, what hasn't, and how to benchmark it properly |
| [`docs/ROADMAP.md`](docs/ROADMAP.md) | Open questions and what's left to build/prove |

---

## Status

This is an active, single-developer research build. Components exist at different levels of maturity:

- ✅ **KuLexicon core lookup** — implemented, benchmarked (O(1) hash lookup, sub-microsecond)
- ✅ **KuLexicon template generators** (Email/Poem/Script/Story/Essay) — implemented, template-based
- 🚧 **Core/Layer/Area mesh routing** — architecture defined, bot decision logic in progress
- 🚧 **Multi-language dictionary support** — ID format supports it; per-language grammar templates not yet built
- 📋 **Full harvest pipeline (819 topics / 24 domains)** — in progress via `harvest.py` / `validate_batch.py`
- 📋 **Self-evolution / auto ring-growth trigger** — designed, not yet implemented in code

Legend: ✅ built & verified · 🚧 in progress · 📋 designed, not yet built

---

## Why this repo exists

Public docs and the sherin.tech site don't currently explain the actual mechanism — the DNA-tagged routing, the Area/Layer addressing, or what "zero payload" precisely means in this system versus what it doesn't mean. This repo is the source of truth for that.

## License

See [`LICENSE`](LICENSE).
