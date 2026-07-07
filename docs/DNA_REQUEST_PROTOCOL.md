# DNA Request Protocol

## 1. Purpose

When a user submits a task, SHERIN doesn't process it as a single opaque blob. It's decomposed into sub-tasks, and each candidate reasoning path through the mesh is tagged with a compact, structured identifier — informally called a "DNA" tag, because it sequentially encodes exactly which request, task, bot, layer, and area produced it.

This tagging is what makes the "zero payload" property possible: instead of moving generated content between mesh stages, the system moves these small fixed-format ID strings, and only resolves them to real content once, at the very end.

## 2. Tag format

A single outcome path is tagged as:

```
req_<N>:task_<N>:bot_<N>:layer_<N>:area_<N>
```

A full task response is a **set** of these tags — one per candidate outcome — joined together:

```
{
  req_1:task_1:bot_1:layer_1:area_1   ||
  req_1:task_1:bot_12:layer_12:area_12 ||
  req_1:task_1:bot_32:layer_3:area_32  ||
  req_1:task_1:bot_222v:layer_12:area_222 ||
  req_1:task_1:bot_356:layer_10:area_356
}
```

Note that `req_1:task_1` is shared across every outcome in the set — they're all candidate answers to the *same* task. What differs is which bot/layer/area produced each one, meaning each tag represents a distinct reasoning path through the mesh.

## 3. Lifecycle of a request

1. **Decomposition** — the incoming task is split into sub-tasks.
2. **Safety gate** — each sub-task is routed through the Layer-1 Safety bot, which organizes it against the model and determines domain routing (e.g. a physics-flavored task routes toward the physics Layer and its linked domains — chemistry, mathematics, etc.).
3. **Mesh traversal** — the DNA-tagged request travels Layer 1 → Layer 2 → Layer 3. At each stage, Planning and Execution bots evaluate the tag against their local Area/KU contents and contribute to the set of possible outcomes.
4. **Outcome generation** — depending on task complexity, this produces **6 to 720 candidate outcome tags** in parallel.
5. **KU-ID attachment** — once a candidate outcome is selected/finalized at the model side, its corresponding **KU-ID chunk reference** is attached to the tag. This is still just an ID — not the underlying content.
6. **Transfer to user** — the DNA+KU-ID tag set is what actually crosses from model to user device. This is the entire "payload": structured IDs, not content.
7. **Local decode** — on the user's device, the KuLexicon / "Lexicon Snake" engine (see [`LEXICON_ENGINE.md`](LEXICON_ENGINE.md)) resolves the KU-IDs into human-readable language and assembles the final output.

## 4. What "zero payload" means here, precisely

This term gets used loosely, so it's worth being exact:

- **True of this design:** the *routing and reasoning-comparison pipeline* (steps 1–6 above) moves only IDs — never raw content, never full candidate text. This is a real, verifiable property of the tag format.
- **Not automatically true:** "zero payload" does not mean the *computation* of those 6–720 candidate outcomes is free or instantaneous. Generating a candidate outcome — even represented as a tiny ID afterward — still requires the Planning/Execution bots to actually evaluate the task against the Layer 2/3 mesh. That evaluation cost scales with task complexity and outcome count, and is the real driver of end-to-end latency, not the size of the tag itself.

In short: the DNA tag format guarantees a **lightweight transport layer**. It does not, by itself, guarantee lightweight *compute* — those are separate properties and should be benchmarked separately (see [`BENCHMARKS.md`](BENCHMARKS.md)).

## 5. Open questions / not yet finalized

- Exact criteria the Safety bot uses to route a task to specific Layers/domains
- How the Planning vs. Execution bots' outcome sets are reconciled into a single final answer (voting? scoring? first-valid?)
- Whether outcome count (6–720) is chosen up front based on task classification, or emerges dynamically during mesh traversal
