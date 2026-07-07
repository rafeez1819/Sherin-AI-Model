# KuLexicon Engine v2.0

## 1. Purpose

KuLexicon is the component that runs on the **user's device** and turns KU-ID chunks (received from the model side via the DNA request protocol) into human-readable language. It is deliberately split into two independent pieces with very different performance characteristics:

1. **Lookup** — resolving a KU-ID to its word/metadata. This is a solved, fast, well-engineered problem.
2. **Generation** — assembling resolved words into readable sentences/paragraphs. This is currently **template-based**, not generative in the language-model sense.

Keeping these separate in your own mental model (and in any benchmark) matters — they have fundamentally different cost profiles.

## 2. ID format

```
[31..28]  language_id   (4 bits  → up to 16 languages)
[27..0]   word_index    (28 bits → up to 268M words per language)
```

A `KuId` is a single packed `u32`. `language_id` selects which loaded dictionary to query; `word_index` is the key within it.

## 3. Dictionary file format (`.kud`)

LZ4-compressed binary file:

```
[0..7]    magic "KUDICT\0\0"
[8..11]   version u32
[12..15]  language_id u32
[16..19]  word_count u32
[20..23]  bucket_count u32   (power of 2, load factor ≤0.5)
[24..27]  strings_offset u32
[28..31]  arrays_offset u32
--- bucket table: bucket_count × 32-byte BucketEntry ---
--- string pool: null-terminated UTF-8 strings ---
--- arrays pool: length-prefixed u32 arrays (synonyms/antonyms/variants) ---
```

## 4. Lookup mechanism (verified, real O(1))

- Open-addressing hash table, **Fibonacci multiplicative hashing** for distribution, linear probing on collision
- Each bucket is 32 bytes (two fit per 64-byte cache line)
- Lookup returns `WordInfo` with pointers **directly into the memory-mapped string pool** — zero-copy, no allocation on the hot path
- Measured performance (per repo benchmark suite): avg 83 ns / p99 295 ns per lookup, ~14.4M lookups/sec on the reference hardware

This part of the system is legitimately fast for the reason the benchmark suggests: it's a well-built in-memory hash table doing exactly what hash tables are good at.

## 5. Generation mechanism (template-based — important distinction)

The five generators (Email / Poem / Script / Story / Essay) all follow the same pattern: resolve a batch of KU-IDs to words, then **insert those words into fixed, hardcoded sentence templates** specific to each mode/tone/phase.

Example (from the Story generator's "Exposition" phase, illustrative structure):

```
"{character} {verb} in {article} {word0} world, where {word1} {verb} all things.
 The {word2} of {character} {verb} quiet, yet filled with {word3}."
```

This is why generation is extremely fast (sub-millisecond for 1,000 IDs) — filling blanks in a fixed string is computationally trivial. It is **not** dynamic sentence construction, grammar resolution, or reasoning about phrasing — it is template-filling ("Mad Libs"-style), which means:

- Two different tasks that route to the same generator/mode/phase with the same KU count will produce **structurally identical output**, differing only in the words plugged in.
- Expressiveness is bounded by however many templates exist per mode/tone/phase — there is no mechanism for the system to invent new sentence structures.

This is a legitimate, useful component for what it does. It should not be described as "language generation" in the sense of a generative language model — it's closer to a fast, deterministic mail-merge over a hash table.

## 6. Multilingual plan

The ID format already supports up to 16 independently-loaded language dictionaries — this part requires no core engine changes. Two things are **not** solved by adding a dictionary alone:

1. **Concept alignment** — for the same KU-ID to mean "the same concept" across languages, every language's dictionary must be built against one shared, canonical concept-index ordering. Building each language's dictionary independently (its own word ordering) will make the same ID resolve to unrelated words across languages.
2. **Per-language grammar/templates** — the current templates are hardcoded English grammar (word order, verb form, article rules). Adding a new language's dictionary supplies vocabulary only; each language needs its **own template set** reflecting its own grammar, or output will read as broken (e.g., correct vocabulary, wrong word order).

**Recommended approach:** treat multilingual support as *N dictionaries + N template sets*, not just *N dictionaries*. Validate with one additional language end-to-end (dictionary + templates + real sentence review) before scaling to more.

## 7. C API surface

```c
#include "include/kulexicon.h"
ku_load_dictionary(0, "en.kud");
WordInfo w = ku_lookup(0x00000001);
void* gen = ku_create_poem(0, "Cosmos");
ku_gen_feed(gen, ids, 30, my_callback, NULL);
ku_gen_flush(gen, my_callback, NULL);
ku_gen_free(gen);
```

Python and C bindings are provided under `examples/`.
