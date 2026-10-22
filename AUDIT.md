# Whitepaper Evidence Audit Contract

This file is the human-facing contract for every citation in `_whitepaper/`.
It is normative: a volume is not releasable until every address-like token in
its prose either matches the format below or is deleted. Machine enforcement
lives in `tools/wp_verify.py`; the resolver lives in `tools/cite.py`.

The corpus of record is `recomp/full_fns.clean.c` (125,881 functions). The
indexes `.team/corpus_index.tsv` and `.team/globals_index.tsv` are read-only
inputs. No volume body, README, corpus file or `.team/` file was modified to
produce this contract. Tokens in this document appear only inside fenced code
blocks, which rule R1 exempts exactly like mermaid and code samples.

---

## 1. Citation format (D-002)

Every prose token must be a bracketed citation carrying its class and the
evidence that resolves it. A bare token with no class is a defect (rule R2).

### function

```text
[fn 0x215be60 importance_action]

python -I tools/cite.py 215be60
  class: function
  name: importance_action
  size: 2899 bytes
  corpus line: 4083652
```

A function citation is valid only when the RVA is a `// ==== ` header in the
corpus of record.

### global

```text
[global 0x50ef4788 f64]

python -I tools/cite.py 50ef4788
  class: global
  type: f64 (double)
  size: 8 bytes
  owning functions: 31
  first owners: 0x984f00 f_984f00 ... 0x13f3290 _geolgame__Cgps__calc_monde
```

### tag

```text
[tag 0x534f4c56 "SOLV"]

python -I tools/cite.py 534f4c56
  class: tag
  big-endian string order:    "SOLV"
  little-endian string order:  "VLOS"
  literal 0x534f4c56 occurs 9 times in the corpus
  comparing functions: 0x1cd4600 f_1cd4600 (first corpus_line 3359730) and 6 more
```

D-009 governs the byte order: the literal as written occurs 9 times and the
byte-swapped form occurs 0 times, so big-endian string order is the empirical
reading. A bit-31 form is reported as a masked tag with an explicit
`masked_to=` pointer; this is not a guess, because the corpus compares the
masked value to the SOLV literal (D-013):

```text
f_1cd8ba0, recomp/full_fns.clean.c:3362306
  (param_1 & 0x7fffffff) == 0x534f4c56
  so 0xd34f4c56 is the bit-31-set SOLV tag
```

A printable FourCC with zero code-immediate hits is never `tag` (D-011).

### quantity

```text
[qty 0x645310 stride]

python -I tools/cite.py 645310
  class: quantity
  exact literal hits in corpus: 10171
  first occurrence: 0x856c70 f_856c70 corpus_line 37217
```

The bracketed word after the RVA must say what the number is (stride, bound,
scalar, count). A quantity is not a memory address.

### data-table key (D-008/D-011/D-012)

```text
[data xim.gt.tsv "POP_AQTE"]
[data xim.gt.tsv "POP_PETE"]

python -I tools/cite.py 41515445
  class: data
  code-immediate hits in corpus: 0
  data-table key: POP_AQTE
  file: data/gt/xim.gt.tsv  line: 21637
  row: POP_AQTE | Acquisition territoriale | Acquisition of Territory
```

`POP_AQTE` and `POP_PETE` are full text keys in `data/gt/xim.gt.tsv`
(lines 21637 and 21639), not binary immediates. A zero-hit printable FourCC is
`data` only when the full key exists; otherwise it is `unresolved`, never
`tag` (D-011). The exact key and exact file must be named (D-012); a bare
`AQTE` also occurs inside unrelated rows and is not a valid key.

### field offset (D-010)

```text
[field 0x620ee0 in NATI]

recomp/full_fns.clean.c:3361889
  piVar1 = (int *)(param_1 + 0x620ee0);
```

A struct field offset is neither an address nor a global. The verifier accepts
the bracketed `[field ...]` form (D-010). The resolver reports the same number
as `class: function` because a real function header `f_620ee0` shares the
number; that collision is why the citation must also quote the indexing
expression, as above.

---

## 2. RVA, stride and tag are three different things

```text
python -I tools/cite.py 215be60   -> class: function, name: importance_action, corpus line: 4083652
python -I tools/cite.py 645310    -> class: quantity, exact literal hits: 10171
python -I tools/cite.py 534f4c56  -> class: tag, big-endian "SOLV", 9 literal hits, 7 comparing functions
```

The paper's uniform numeric prefix hides three different objects: an RVA is a
code address reproduced by the first command as a function at corpus line
4083652, a stride is a record size reproduced by the second as a quantity with
10171 literal hits, and a tag is four ASCII bytes used as an immediate
reproduced by the third as big-endian SOLV with 9 hits and 7 comparing
functions. Because the prefix is identical, a reader cannot tell which object
is meant without running the resolver. Conflating them is how the old draft
manufactured addresses: the first token is a header, the second is not an
address, and the third is not a function entry point.

---

## 3. Per-volume status (pending)

Rows stay `pending` until each volume owner lands
`_whitepaper/evidence/NN_CLAIMS.md` and the integrator assembles
`_whitepaper/EVIDENCE_INDEX.md`.

| Volume | Unique cited RVAs | Resolved | Wrong-RVA corrections | Claims removed | Claims added from corpus | Status |
| --- | --- | --- | --- | --- | --- | --- |
| 00_OVERVIEW.md | pending | pending | pending | pending | pending | pending |
| 01_SIMULATION_CORE.md | pending | pending | pending | pending | pending | pending |
| 02_ECONOMY.md | pending | pending | pending | pending | pending | pending |
| 03_MILITARY.md | pending | pending | pending | pending | pending | pending |
| 04_POLITICS_AND_POPULARITY.md | pending | pending | pending | pending | pending | pending |
| 05_PERSONALITY_AND_CHARACTERS.md | pending | pending | pending | pending | pending | pending |
| 06_LAWS_AND_LEGISLATION.md | pending | pending | pending | pending | pending | pending |
| 07_DEMOGRAPHY_AND_ENVIRONMENT.md | pending | pending | pending | pending | pending | pending |
| 08_REGIONS_CITIES_AND_INFRASTRUCTURE.md | pending | pending | pending | pending | pending | pending |

The generated resolver table `_whitepaper/evidence/RVA_CLASSIFICATION.tsv`
contains one row per unique token per file over the ten content files
(`00`–`08` plus `README.md`), including tokens inside fenced code blocks. Rule
R1 deliberately ignores fenced blocks (mermaid and code samples are not prose
citations), so the TSV row count is larger than the R1 token count.

### Known attribution conflict (recorded, not hidden)

```text
names.tsv:           parlement_vote_lawpack_chambre1_estimation -> 0x2160650
corpus header:       0x2160650 parlement_vote_lawpack_chambre1_estimation
assert_names.tsv:    string 0x2a36bf0 attributed to function 0x2160540
corpus header there:0x2160540 f_2160540 (different name)
```

Both attributions are printed by the resolver; neither is silently preferred.
The other corrected anchors are listed below with their true corpus RVAs.

```text
calc_effet_loi    cited 0x1d58a10 -> 0x1cfb560
gestion_epidemies cited 0x16b8a40 -> 0xd53a40
traitement_loi    0x1d60ce0 (already correct)
importance_action 0x215be60 (already correct)
```

### Current gate snapshot

At the time this contract was written the gate fails for real content reasons.

R1 fails on 11 unique prose tokens. Nine resolve to no function header, global
entry, numeric literal or data key; two are data keys cited as binary
immediates:

```text
0x16b8a40  0x182b0c0  0x1d4a920  0x1d58a10  0x21ce890
0x29a09f0  0x2a04f10  0x47938840  0x621768
0x41515445 (POP_AQTE -> data/gt/xim.gt.tsv:21637, must use [data ...])
0x50455445 (POP_PETE -> data/gt/xim.gt.tsv:21639, must use [data ...])
```

None is a stride, and no token that earns `tag` (one or more comparing
functions) is flagged; the tag and stride tokens in the volumes all resolve.
R2 still fails because a substantial number of prose occurrences are
bare, unconverted citations; the gate prints the exact current count. R5
passes (no `UNVERIFIED`, `FABRICATED?`, `TODO`, `TBD` or `??` markers). R3, R4
and R6 fail but are non-gating until the integrator lands the per-volume
ledgers, the missing `pseudocode` blocks and `EVIDENCE_INDEX.md`. Counts in
this section are a snapshot; the volumes are being edited in parallel, so run
the gate for the current numbers.

The gate only counts complete 5–8 digit hex numbers as RVA tokens. A longer
runtime address is a module-base pointer and is not truncated into a bogus RVA
prefix.

---

## 4. How to reproduce any verdict

Run from the repository root with an isolated interpreter (`-I`). Windows paths
work unchanged; `cite.py` locates the corpus relative to its own file.

```bat
python -I tools/cite.py 215be60
python -I tools/cite.py 215be60 --body --max-lines 80
python -I tools/cite.py 50ef4788
python -I tools/cite.py 534f4c56
python -I tools/cite.py 645310
python -I tools/cite.py 1d58a10
python -I tools/cite.py 2160650 2160540
python -I tools/cite.py 215be60 --json
```

Regenerate the classification table over the ten content files:

```bat
python -I tools/cite.py --classify-all _whitepaper/00_OVERVIEW.md _whitepaper/01_SIMULATION_CORE.md _whitepaper/02_ECONOMY.md _whitepaper/03_MILITARY.md _whitepaper/04_POLITICS_AND_POPULARITY.md _whitepaper/05_PERSONALITY_AND_CHARACTERS.md _whitepaper/06_LAWS_AND_LEGISLATION.md _whitepaper/07_DEMOGRAPHY_AND_ENVIRONMENT.md _whitepaper/08_REGIONS_CITIES_AND_INFRASTRUCTURE.md _whitepaper/README.md > _whitepaper/evidence/RVA_CLASSIFICATION.tsv
```

Run the release gate:

```bat
python -I tools/wp_verify.py
python -I tools/wp_verify.py --strict
python -I tools/wp_verify.py --rule R1
python -I tools/wp_verify.py --rule R1 --report-json C:\Users\Riley\AppData\Local\Temp\opencode\r1.json
```

`--rule` re-runs one rule in isolation. Without `--strict`, R3, R4 and R6 are
printed but non-gating; with `--strict` every rule gates. `--report-json` with
no path writes JSON to stdout (human text goes to stderr); with a path it
writes the JSON file and prints the human summary to stdout. The exit status is
0 only when every enabled rule passes. The JSON contains `generated_at`, so
two otherwise identical runs differ in that field only.

To check one citation by hand, run the resolver with its RVA and compare the
printed class, name/type, corpus line and literal count with the bracketed
citation in the volume. If the class is `unresolved`, the citation is a defect
and must be corrected to the real RVA or deleted (D-003/D-004).
