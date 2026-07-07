# Contributing

SHERIN is currently a single-developer research project. This file exists mainly so the intent is documented even while the project is solo.

## Reporting issues
Open a GitHub issue describing:
- Which component (Core/Layer/Area mesh, DNA protocol, KuLexicon, harvest pipeline)
- Expected vs. actual behavior
- Hardware/environment if performance-related

## Benchmark contributions
If you're submitting a performance number, please include:
1. The exact task/input used
2. Hardware specification
3. Which pipeline stage was timed (see `docs/BENCHMARKS.md` §3 for the stage breakdown)

Numbers without this context won't be accepted into `docs/BENCHMARKS.md`, since the whole point of that document is to keep claims falsifiable and comparable.

## Code style
- Rust components: standard `rustfmt`
- Python components: `black` formatting preferred
