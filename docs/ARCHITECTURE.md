# SHERIN Architecture

## 1. Overview

SHERIN's structure is a **spherical mesh**, built from concentric rings ("Layers") stacked from a central Core outward. Each Layer is a disc divided into 12 equal 30° wedges ("Areas"), and each Layer connects vertically to the Layer above (`+1`) and below (`-1`).

This is not a neural network in the transformer sense. It's a **deterministic hierarchical router and retrieval structure** over pre-harvested knowledge units (KUs). Reasoning happens by routing a task through this structure and comparing it against what's stored at each node, not by computing activations over learned weights.

```
                    Core (Level 1)
                        |
        ┌───────────────┼───────────────┐
        |               |               |
    Safety Bot      Planning Bot   Execution Bot   ← Layer 1 (3 bots)
        |               |               |
   [3 sub-bots]    [3 sub-bots]    [3 sub-bots]    ← Layer 2 (9 bots total)
        |               |               |
  [sub-divisions] [sub-divisions] [sub-divisions]  ← Layer 3
```

## 2. Core

The Core is the Level-1 entry/exit point. Every user task arrives here first and every final response passes back through here.

## 3. Layer 1 — the three governing bots

| Bot | Role |
|---|---|
| **Safety** | Organizes and gates all user tasks against the model. Acts as the routing/oversight port — every task passes through Safety before being dispatched into the mesh. |
| **Planning** | Compares the incoming task against the full Layer 2 + Layer 3 mesh to construct candidate outcome paths. |
| **Execution** | Same cross-referencing role as Planning, from the execution/action side — turns candidate paths into concrete outcome sets. |

Layer 1 has 9 bots in Layer 2 (3 per Layer-1 bot) and further sub-division in Layer 3.

## 4. Layers as domain stack

Vertical position in the mesh (`-1` / `+1`) carries a second meaning beyond "distance from Core": each Layer is also associated with a **knowledge domain** (e.g. physics, chemistry, mathematics, history, biology), and adjacent Layers hold adjacent/related domains. Moving vertically through Layers is effectively moving through a domain graph, not just a depth counter.

**Open question:** whether Area growth (below) happens per-Layer/per-domain independently, or across the whole sphere simultaneously, is still being finalized.

## 5. Areas — the 30°/12-port ring structure

Each Layer-disc is divided into **12 Areas**, each a 30° wedge, separated by 12 I/O ports around the ring (12 × 30° = 360°). This division exists specifically to **avoid data-path collisions**:

- Each Area carries **two one-directional data lines**: one carrying tokens outward (toward the ring's outer edge), one carrying tokens inward (toward Core). Because both directions exist in every Area, any Area can reach any token/task data without needing true bidirectional traffic on a single lane.
- **Adjacent Areas alternate rotation direction** — Area 1 clockwise, Area 2 anticlockwise, Area 3 clockwise, and so on. At every Area-to-Area boundary, this guarantees the two flows are always **crossing** (a natural to-and-fro handoff, outer-in / inner-out) rather than **merging** (which would risk collision if both flowed the same direction into the same junction).
- Each Area has its own local **16-vertex** addressing scheme for locating data within it (see the vertex-numbering notes for the exact sequential order).

## 6. Self-evolution / ring growth

The mesh is not pre-provisioned. It grows **inside-out**, in fixed increments, purely in response to data volume:

1. Define **Area 1** in the innermost ring → fill it with harvested data → attach its I/O path.
2. Once Area 1's I/O path is set, define **Area 2** → fill → attach I/O path. Repeat through **Area 12**.
3. Once all 12 Areas in that ring are created and filled, the same process **starts again one ring further out** — a fresh set of 12 Areas.
4. The whole structure grows outward ring by ring as capacity is actually needed — never before.

This is the "self-evolution" property: the architecture designs its own next increment of capacity based on fill-state, rather than being manually re-architected.

**Design note (open, not yet resolved):** ring growth guarantees the *transport/routing layer* never provisions unused capacity. It does not, by itself, reduce the *compute cost* of generating candidate outcomes at each Layer — that cost scales with how many outcome paths (6–720, depending on task complexity) are actually being evaluated. See [`BENCHMARKS.md`](BENCHMARKS.md) for why this distinction matters for any performance claim.

## 7. What this architecture is good at, and what it isn't

**Strong fit:**
- Fully offline, single-user knowledge retrieval over a domain you've actually harvested
- Deterministic, auditable routing — you can trace exactly which Layer/Area/KU produced an output
- Low-latency response once a task maps onto existing harvested structure

**Structural limitation:**
- Genuinely novel tasks outside the harvested curriculum have nothing to route to. This is not a tuning problem — it's inherent to a retrieval-over-harvested-KUs design, as opposed to a model that generalizes from learned weights. Scoping SHERIN's claims to "fast, private, offline retrieval over my own curated knowledge" is accurate; scoping it as a competitor to frontier generalist models on open-ended reasoning is not.
