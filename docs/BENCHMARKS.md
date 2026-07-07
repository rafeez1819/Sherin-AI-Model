# Benchmarks

## 1. Why this doc exists

Speed is central to SHERIN's value proposition, so speed claims need to survive scrutiny. This doc separates **what has been measured** from **what is a design goal**, and flags where a benchmark's framing needs to be more precise.

## 2. What's actually measured and verified

### KuLexicon lookup (`ku_bench`)
- avg 83 ns, p99 295 ns per lookup
- ~14.4M lookups/sec
- 1,000 IDs → generated text: avg 0.05 ms
- Dictionary size on disk: 5.3 MB

This is real, reproducible, and consistent with what a well-built in-memory open-addressing hash table should achieve. No concerns here.

### `sherin_wire_test.py` pipeline benchmark (10,000 runs each)
Measured stages: Snake sentence assembly, diagram engine, intent whisper, auto KU generator, full pipeline `process()`. All results reported in **microseconds**, including "full research deep" at ~32–54 µs.

**Flag:** this is faster than a single real LLM token generation, even on capable local hardware — actual local model inference typically runs in the low-to-high **milliseconds** per token, not microseconds, regardless of GPU. The most likely explanation is that this benchmark is measuring the **orchestration/routing/template-fill layer only** (DNA tag construction, KU lookup dispatch, Snake template assembly) — which genuinely can run in microseconds, per the lookup/template mechanics documented in `LEXICON_ENGINE.md` — and does **not** yet include an actual generative/model inference step (e.g., an Ollama call) in the timed path.

This is not a claim that the numbers are fabricated — it's a claim that **the benchmark is currently measuring a narrower thing than "full research deep" implies**, and the label should be corrected or the missing stage should be added to the timed path.

## 3. Recommended instrumentation (not yet implemented)

To make performance claims defensible, `process()` should log timestamps at each of these boundaries separately, and report them as **separate numbers**, not one collapsed total:

1. DNA tag creation
2. Safety-bot routing decision
3. Mesh traversal / outcome generation (Planning + Execution comparison against Layer 2/3)
4. KU-ID fetch from storage
5. Any actual model/Ollama inference call, if one occurs in this path
6. Local Lexicon decode (lookup + template fill)

Reporting per-stage numbers will very likely show: routing and decode are extremely fast (microseconds, matching the lookup benchmark), while outcome generation (stage 3) — if it involves any real inference — is the actual bottleneck and will be orders of magnitude slower. That's not a bad result; it's the honest result, and it tells you where to actually spend optimization effort.

## 4. On "tasks/sec" as a unit

"60,000 tasks/sec" is not a standard, comparable unit the way tokens/sec is for language models, because task complexity varies enormously — a KU lookup and a 720-outcome generative comparison are not the same unit of work. Any published performance claim should specify:

- A **fixed, defined task** (exact input, exact expected output complexity)
- The **hardware** it was run on
- Whether the timed path includes actual content generation or only routing/lookup

Without these, the number can't be meaningfully compared to anything else — including to itself across different runs.

## 5. Status

| Component | Benchmark status |
|---|---|
| KuLexicon lookup | ✅ Measured, credible, reproducible |
| KuLexicon template generation | ✅ Measured, credible (matches expected cost of template-fill) |
| Full `process()` pipeline, staged | 📋 Needs re-instrumentation per §3 before publishing as a headline number |
| Mesh outcome generation cost (6–720 outcomes) | 📋 Not yet isolated/measured separately |
