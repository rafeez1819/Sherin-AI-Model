# Roadmap / Open Questions

This is a living list of what's architecturally decided vs. what's still unresolved. Kept separate from README so it can be updated without touching the project's front door.

## Architecture — open questions
- [ ] Does a newly-grown outer ring inherit the same domain as the ring inside it, or can it shift to a related domain?
- [ ] Does ring growth happen per-Layer/domain independently, or across the whole sphere at once?
- [ ] Exact reconciliation logic: how do Planning and Execution bots' separate outcome sets get merged into one final answer (voting / scoring / first-valid-match)?
- [ ] Is outcome count (6–720) determined before mesh traversal (based on task classification) or does it emerge dynamically?

## Lexicon / multilingual
- [ ] Build canonical concept-index ordering so KU-IDs mean the same concept across all language dictionaries
- [ ] Build per-language grammar/template sets (starting with one validated language beyond English)
- [ ] Decide long-term plan for the 16-language ceiling (4-bit language_id field) if more languages are needed later

## Benchmarking
- [ ] Re-instrument `process()` per stage (see `BENCHMARKS.md` §3) before publishing headline numbers
- [ ] Isolate and measure the actual cost of mesh outcome generation (Planning/Execution comparison), separate from routing and decode
- [ ] Define a fixed benchmark task suite so "tasks/sec" numbers are comparable across hardware and across versions

## Harvest pipeline
- [ ] Complete full 819-topic / 24-domain harvest via `harvest.py` + `validate_batch.py`
- [ ] Implement the self-evolution trigger (automatic new-Area/new-Layer creation on fill-state) in code, not just architecture

## Positioning
- [ ] Decide and document, clearly and consistently, what SHERIN is being compared against — "fast offline retrieval over a personally-curated knowledge base" is a defensible, real claim; "faster than frontier generalist models" is not, since they solve different problems (generalization vs. retrieval over harvested data)
