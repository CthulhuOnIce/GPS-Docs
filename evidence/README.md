# Per-volume evidence ledgers

Each volume `NN_*.md` must ship `_whitepaper/evidence/NN_CLAIMS.md`
(`00_CLAIMS.md` through `08_CLAIMS.md`). These files are written by the volume
owner, never by the evidence or integrator tasks. Rule R4 of
`tools/wp_verify.py` checks that all nine exist.

Each file is a Markdown table with these exact columns, in this order:

```
claim_id | old_text_verbatim | class | rva | verdict(verified|corrected|removed) | corpus_evidence(function+line) | replacement_summary
```

Column contract:

| Column | Meaning |
| --- | --- |
| `claim_id` | stable id, e.g. `04-C017` |
| `old_text_verbatim` | the exact sentence/row from the old volume, copied verbatim |
| `class` | `function`, `global`, `tag`, `quantity`, `data` or `field` (D-002/D-008/D-010) |
| `rva` | the cited token, normalized to lowercase `0x...`; `-` for removed claims |
| `verdict` | `verified`, `corrected` or `removed` |
| `corpus_evidence(function+line)` | resolving name plus `recomp/full_fns.clean.c` line, e.g. `importance_action+4083652`; `-` when removed |
| `replacement_summary` | one sentence describing only arithmetic and branches that exist in the C, or `deleted` |

`removed` means deleted, not softened. A replacement may not invent a
"conceptually similar" mechanism from domain knowledge (D-004). Corrections use
the real RVA (D-003), never a footnote pointing at the old wrong address.

## Generated inputs

`RVA_CLASSIFICATION.tsv` in this directory is generated over the ten content
files only (the ten-file command is deliberate; `_whitepaper/*.md` would also
match `AUDIT.md` and produce eleven files):

```bat
python -I tools/cite.py --classify-all _whitepaper/00_OVERVIEW.md _whitepaper/01_SIMULATION_CORE.md _whitepaper/02_ECONOMY.md _whitepaper/03_MILITARY.md _whitepaper/04_POLITICS_AND_POPULARITY.md _whitepaper/05_PERSONALITY_AND_CHARACTERS.md _whitepaper/06_LAWS_AND_LEGISLATION.md _whitepaper/07_DEMOGRAPHY_AND_ENVIRONMENT.md _whitepaper/08_REGIONS_CITIES_AND_INFRASTRUCTURE.md _whitepaper/README.md > _whitepaper/evidence/RVA_CLASSIFICATION.tsv
```

Columns: `file`, `rva`, `class`, `resolved_name_or_type`, `evidence_pointer`.
It contains one row per unique token per file, including fenced code blocks.
It is an input to the integrator's global `_whitepaper/EVIDENCE_INDEX.md`; the
integrator owns that index and the final regeneration after every volume edit
settles. This checked-in snapshot is not authoritative while volumes are still
being edited; do not re-run it mid-flight and assume it is final. Do not
hand-edit the generated TSV.
