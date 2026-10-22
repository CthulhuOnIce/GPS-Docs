# Volume 1 — Claims Ledger

One row per pre-existing claim touched while rewriting `_whitepaper/01_SIMULATION_CORE.md`. The corpus of record is `recomp/full_fns.clean.c` (D-001). `old_text_verbatim` is quoted from the pre-rewrite volume. `rva` is the primary address the old text used; `—` means the old claim had no address. Verdicts are exactly `verified`, `corrected`, or `removed`.

| claim_id | old_text_verbatim | class | rva | verdict(verified\|corrected\|removed) | corpus_evidence(function+line) | replacement_summary |
| --- | --- | --- | --- | --- | --- | --- |
| V1-001 | `The GPS5 simulation operates across three decoupled time scales to balance interactive 60 FPS rendering with compute-heavy macroeconomic modeling:` | function | 0x1c37d70 | corrected | `f_1c37d70` 0x1c37d70 lines 3294578-3294693; `f_25b6690` 0x25b6690 lines 4847348-4847355; `calc_monde_fin_tour` 0x140e5d0 lines 2050105-2050107 | Three time representations exist (tick seconds, game-hour accumulator, macro day counter), but they are coupled through code; "decoupled clocks" is replaced by the actual data flow. |
| V1-002 | `The engine timer (`f_25b6690`) measures wall-clock elapsed seconds using high-resolution hardware counters:` | function | 0x25b6690 | corrected | `f_25b6690` 0x25b6690 lines 4847341-4847356; globals 0x70ee6528, 0x70ee6524 | f_25b6690 reads a tick counter at 0x70ee6528 and a rate at 0x70ee6524 and returns a float. The corpus does not establish a hardware counter or wall-clock time; "hardware" and "wall-clock" are removed. |
| V1-003 | `$$\text{Elapsed Seconds} = \frac{\text{Ticks}_{f64} \text{ (at } \mathtt{0x70ee6528}\text{)}}{\text{Rate}_{i32} \text{ (at } \mathtt{0x70ee6524}\text{)}}$$` | function | 0x25b6690 | corrected | `f_25b6690` 0x25b6690 lines 4847348-4847355 | The simple ticks/rate form is only the fallback branch; the non-zero smoothing branch uses 0x70ee6548, 0x70ee6540, 0x70ee651c and 0x70ee6530 and clamps the advance. |
| V1-004 | `Function `f_1c37d70` (invoked at the head of every `calc_monde` call) translates wall-clock duration into game hours:` | function | 0x1c37d70 | corrected | `f_1c37d70` 0x1c37d70 line 3294602; `calc_monde` 0x13f3290 line 2030564 | The normal branch divides an elapsed-time difference by the ratio of 0x464ddc88 to 0x322a4f4. The corpus does not establish wall-clock duration or game-hour units; those mechanism words are removed. |
| V1-005 | `The accumulated float value at `0x464ddc98` increments the master date structure (`COleDateTime` at `0x4813c0a8`), tracking:` | global | 0x464ddc98 | corrected | `f_1c37d70` 0x1c37d70 lines 3294625-3294641, 3294679; `f_25a7630` 0x25a7630; `f_25a7940` 0x25a7940; `f_25cafe0` 0x25cafe0 | 0x464ddc98 is a float whole-hour accumulator; 0x464ddc9c holds the fraction. 0x4813c0a8 is an 8-byte millisecond timestamp built by f_25a7630, not a COleDateTime. |
| V1-006 | `- `today` (`0x464916a8`): Julian day counter integer.` | global | 0x464916a8 | corrected | `f_1c37d70` 0x1c37d70 line 3294689; `f_25cafe0` 0x25cafe0 | 0x464916a8 is `(int)floor(ms / 86400000.0)`, a day serial, not a Julian day counter. |
| V1-007 | `- `year` (`0x46494828`), `month` (`0x46494824`), `day_of_year` (`0x46494810`).` | global | 0x46494828 | verified | `f_1c37d70` 0x1c37d70 lines 3294683-3294693; `f_25e9b60` 0x25e9b60; `f_25df0d0` 0x25df0d0 | Confirmed: year, month and day-of-year are written from the date object. |
| V1-008 | `- `game_mode` (`0x4649165c`): Simulation state flag ($< -6$ represents menus/paused state, $0$ represents active simulation).` | global | 0x4649165c | removed | `f_1c37d70` 0x1c37d70 lines 3294600-3294640; corpus-wide comparisons to 0 and to -1, -2, -3, -4, -6, -7, -8, -9, -10 | The value is a mode gate and `0` is the normal path, but the corpus never defines `-6` as menus/paused or `0` as active simulation. The semantic claim is removed; only the gate fact survives. |
| V1-009 | `Clock variable: `0x50ef4788` (double precision floating-point days).` | global | 0x50ef4788 | verified | `calc_monde_fin_tour` 0x140e5d0 lines 2050105-2050107; `f_239f3b0` 0x239f3b0 line 4494244 | Confirmed: a double used as a day serial and advanced by one assignment. |
| V1-010 | `$$\Delta t_{\text{turn}} = \frac{1}{26} \text{ year} \approx 14.03846 \text{ days}$$` | function | 0x140e5d0 | corrected | `calc_monde_fin_tour` 0x140e5d0 lines 2050098-2050107 | The code increments by `((days_in_year / 365.0) * 0.038461538461538464 * 365.25)`; 0.038461538461538464 is 1/26, so the 365-day value is 14.048076923… days, not 14.03846. |
| V1-011 | `A macro catch-up loop in `f_140e000` (`bank_step`) triggers whenever:` | function | 0x140e000 | corrected | `bank_step` 0x1378460 line 1980293; `calc_monde_fin_tour` 0x140e5d0 lines 2050100-2050107 | `f_140e000` is an 8-byte global, not a function header. The real bank_step is 0x1378460 and only rounds a double. The day comparisons are in calc_monde_fin_tour. |
| V1-012 | `$$\text{today} - \text{day}(\text{clock} + \Delta t_{\text{turn}} + 5) > 0$$` | function | 0x140e5d0 | corrected | `calc_monde_fin_tour` 0x140e5d0 lines 2050100-2050107, 2048929-2048935 | No such single inequality exists. The code compares 0x464916a8 with `bank_step(dVar55) + 1`, `+2`, `+3` or `+4` in separate branches. |
| V1-013 | `runs under the synchronization of the master permanent mutex `IDME_cmt_permanent`.` | function | 0x13f3290 | verified | `calc_monde` 0x13f3290 lines 2030785-2030787 | Confirmed as a log string at the lock site; the lock call is `mutex_scope(3,1)`. |
| V1-014 | `It processes 36 sequential phases.` | function | 0x13f3290 | removed | `calc_monde` 0x13f3290 line 2030705 (1x prof_begin), 56x prof_end; `prof_end` 0x12d04b0 is 3 bytes | The count 36 is not in the corpus. Removed. Counted facts: 1 prof_begin, 56 prof_end, no phase index and no label string. |
| V1-015 | `Profiler instrumentation calls `f_12d04b0(profiler, label)` mark the boundaries:` | function | 0x12d04b0 | corrected | `prof_end` 0x12d04b0 line 1922599; `prof_begin` 0x12bb2b0 line 1922580; `calc_monde` 0x13f3290 line 2030705 | 0x12d04b0 is `prof_end`, a 3-byte `return;` with no arguments. `prof_begin` takes one opaque argument. No label string is in the corpus. |
| V1-016 | "Serializes game state into profiles/<name>.sav" | function | 0x1409f40 | corrected | `calc_monde_debut_tour_etapes` 0x1409f40 Etape18 label line 2048108; `f_8732b0` 0x8732b0; literals `.sav`/`profiles/` in the block | Etape18 builds the profile/save path and metadata from the `.sav`/`profiles/` literals and `f_8732b0`; the block does not itself call io_write, so "Disk I/O autosave" is narrowed to path/metadata construction. |
| V1-017 | `The driver function `calc_monde_debut_tour` (`0x1409f40`) shards the computation across subsequent visual frames via the global state index `_DAT_324da9c`:` | function | 0x1409f40 | corrected | `calc_monde` 0x13f3290 lines 2031956-2031957; `calc_monde_debut_tour_etapes` 0x1409f40 line 2045281; global 0x324da9c | The driver takes `param_1` and calc_monde passes 0x324da9c into it and stores the return value. The 19 labeled blocks are counted below. |
| V1-018 | `Executing a full macroeconomic turn across 300 nations, 364 sectors, 124,000 enterprises, and 620,000 industrial facilities takes tens of billions of CPU instructions.` | quantity | — | corrected | `f_8c5310` 0x8c5310 lines 114698-114711 (inside `if (param_1 == 0x166)`; `lVar2 = 299`, 299 iterations); `f_8c5500` 0x8c5500 lines 114783, 114819 (`while (iVar6 < 300)`, iVar6 starts at 1, also 299 iterations); 124,000 bound in `f_8dc870` 0x8dc870 line 133281; no 364 or 620,000 count at these sites | The 299-iteration nation loops are real. No assignment of the literal 300 to 0x322a3f4 was found. The 364-sector and 620,000-facility counts are not established; the 124,000 bound is used against 0xd65da58, not 0xe1e9910. |
| V1-019 | `**Heaviest step in engine**: iterates all companies $\times$ buildings to compute national sector outputs` | function | 0x1409f40 | removed | `calc_monde_debut_tour_etapes` 0x1409f40 Etape1 line 2045796; calls f_1a4e8d0, f_1a33cf0 | The qualitative "heaviest" claim is not in the corpus. The verified Etape1 calls are listed instead. |
| V1-020 | `Traverses 12,000 records of stride `0x41b8`` | quantity | 0x41b8 | corrected | `calc_monde_debut_tour_etapes` 0x1409f40 lines 2046292, 2046501; base 0x1e660190 | The 0x41b8 stride is real and used with 0x1e660190; the 12,000 count is not present at that site and is removed. |
| V1-021 | `Computes input availability bottlenecks across Leontief input-output matrix` | function | 0x18e77a0 | removed | `calc_monde_debut_tour_etapes` 0x1409f40 Etape5 line 2046658; calls f_18e77a0, f_155cc10 | No matrix arithmetic is named in the block. The description is removed; the calls remain. |
| V1-022 | `124,000 companies evaluate investment, hiring, expansion, or bankruptcy` | function | 0x9df3b0 | removed | `calc_monde_debut_tour_etapes` 0x1409f40 Etape12 line 2047413; calls f_9df3b0, f_15445f0 | The calls are real, but the 124,000 bound is not used at that site and the described decision logic is not in the block. Removed. |
| V1-023 | `Increments turn counter; resets `_DAT_324da9c` to 0` | function | 0x1409f40 | corrected | `calc_monde_debut_tour_etapes` 0x1409f40 corpus line 2048638 (body line 3358); `calc_monde` 0x13f3290 line 2032028 | The driver returns the unsigned value -1; calc_monde tests `(int)_DAT_324da9c < 0` and handles it. No turn counter is incremented in the driver. |
| V1-024 | `The master loader/saver `Cdatabase::es_base` (`0x143acd0`) manages the 8.48 GB in-memory state:` | function | 0x143acd0 | removed | `es_base` 0x143acd0 line 2072351; no 8.48 GB literal in the corpus | The 8.48 GB figure is a runtime observation, not a corpus value. Removed. |
| V1-025 | `- **Signature**: `void es_base(int mode, const wchar_t* path, int version = -1)`` | function | 0x143acd0 | corrected | `es_base` 0x143acd0 line 2072351 | Decompiled signature: `ulonglong es_base(uint param_1, ushort *param_2, ulonglong param_3, undefined8 param_4)`. The return type and arity are corrected; "version = -1" is not established. |
| V1-026 | `- `Mode 5`: Fast binary image restore from disk (sequential block streaming via `0x25b58e0`).` | function | 0x25b58e0 | verified | `es_base` 0x143acd0 lines 2073817-2073818; `io_read` 0x25b58e0 | Confirmed: mode 5 calls io_read. |
| V1-027 | `- `Mode 6`: Fast binary snapshot dump (sequential block writing via `0x25b5ea0`).` | function | 0x25b5ea0 | verified | `es_base` 0x143acd0 lines 2073823-2073824; `io_write` 0x25b5ea0 | Confirmed: mode 6 calls io_write. |
| V1-028 | `- `Mode 1`: Field-by-field serialized walker (used for savegame files `.sav` and text export). Supports **delta compression** against a base reference file (`_DAT_70eeaa50`), storing only modified fields.` | global | 0x70eeaa50 | removed | `es_base` 0x143acd0 lines 2073820-2073821; `f_146c790` 0x146c790; `io_read` 0x25b58e0 line 4846251 | Mode 1 uses f_146c790 selector 1. 0x70eeaa50 is a file handle used by io_read/io_write; no delta base or modified-field stream is visible. Delta compression removed. Modes 2,3,4,7 also exist. |
| V1-029 | `- Nation array pointer: `*0x46208908` (stride `0x645310`, count at `0x322a3f4` = 300).` | global | 0x46208908 | corrected | `f_8c5310` 0x8c5310 lines 114698-114711 (inside `if (param_1 == 0x166)`); `f_1c37d70` 0x1c37d70 line 3294698 | Base and stride are confirmed. `f_8c5310` runs 299 iterations (`lVar2 = 299` down to 0); the literal 300 is in `f_8c5500` (`while (iVar6 < 300)`, iVar6 from 1, also 299 iterations). No assignment of the literal 300 to 0x322a3f4 was found. |
| V1-030 | `- Companies array pointer: `*0xe1e9910` (stride `0x1f60`, count = 124,000).` | global | 0xe1e9910 | corrected | `f_914a20` 0x914a20 line 177185; 124,000 bound against 0xd65da58 in `f_8dc870` 0x8dc870 line 133281 | Base and stride are confirmed. 0x1f60 is the company stride; the 124,000 bound is not established for 0xe1e9910 (it is used with 0xd65da58). |
| V1-031 | `- Variable store pointer: `*0x32d1920` (4,000,000 entries of 16 bytes each).` | global | 0x32d1920 | corrected | `variable_store_get` 0x8b9490 line 105580 | The allocation is `f_27933fc(&DAT_32d1920, 0x10, 4000000, f_1cc89c0)`: 4,000,000 records of 16 bytes. The base is `&DAT_32d1920`, not a dereferenced pointer. |
| V1-032 | `- Persons array pointer: `*0x5d85ada0` (stride `0xad0`, count = 51,000).` | global | 0x5d85ada0 | corrected | `f_869870` 0x869870 line 46506; `f_99fe50` 0x99fe50 line 253812 | Base and stride are confirmed (`index * 0xad0 + _DAT_5d85ada0`). The 51,000 count is a loop bound in a different function and is not established for this base at that site. |
| V1-033 | `When building native savegames, each record is preceded by an 8-byte chunk header:` | tag | 0x4e415449 | verified | `f_147d030` 0x147d030 lines 2111594-2111600; `f_8604f0` 0x8604f0 | Confirmed for the NATI record: 4-byte tag `0x4e415449` followed by 4-byte size `0x645310` through f_8604f0. No named struct is defined. |
| V1-034 | `Within `NATI`, sub-arrays for economic sectors are stored contiguously starting at byte offset `+0xcbc8`, with each sector allocated exactly `0x4068` bytes (364 sectors $\times$ `0x4068` = 6,108,864 bytes).` | quantity | 0x4068 | corrected | `f_8c5310` 0x8c5310 line 114703; `f_86f240` 0x86f240 line 48961 | The `+0xcbc8` base and 0x4068 stride are real. The old product is arithmetically wrong (364 × 16,488 = 6,001,632) and the 364-sector count is not established at the site. |
| V1-035 | `Corporate redundancies, plant closures, severance` | function | 0x1053e40 | removed | `f_1053e40` 0x1053e40 line 1480951 (38 bytes; returns `(int)((_DAT_1c168090 / f_104d590()) * 212.0)`) | The cited function is a 38-byte arithmetic helper, not a redundancy system. The description is removed. |
| V1-036 | `Player leaderboard & historical score evaluation` | function | 0x959710 | removed | `f_959710` 0x959710 line 214919 (3 bytes; `return;`) | The cited function is a three-byte no-op. The leaderboard description is removed. |
| V1-037 | `Scripted scenario conditions & event reactions` | function | 0x22c3590 | corrected | `reactions_process` 0x22c3590 line 4349710 | The address is the function `reactions_process`, not `trigger`; the description is kept only as the runtime label. |
| V1-038 | `Macro-turn debt & financial flow settlements` | function | 0x1378460 | corrected | `bank_step` 0x1378460 line 1980293 (60 bytes) | The address is the rounding helper `bank_step`, not the bank system. The description is corrected to "rounding helper". |

## Closing statistics

- **Unique address tokens cited in the old volume:** 91.
  - 89 unique `0x…` tokens (65 function headers, 18 global headers, 6 numeric literals used as strides/offsets).
  - 2 additional bare `f_…` tokens not written with `0x`: `f_12d04b0` (function `prof_end`) and `f_140e000` (global, not a function).
- **How many resolved as which class:** 66 functions, 19 globals, 6 quantities, 0 tags. The six quantity tokens are `0x1f60`, `0x4068`, `0x41b8`, `0x645310`, `0xad0`, `0xcbc8`.
- **Wrong addresses:** 1 function address corrected: `f_140e000` (old text: `bank_step`) → `0x1378460` (`bank_step`). Two further targets were wrong for the named behaviour and had no replacement address: `0x1053e40` (not `plansociaux`) and `0x959710` (a no-op, not `classer_joueur`). Two names were corrected without changing the address: `0x22c3590` is `reactions_process` (not `trigger`), and `0x1378460` is `bank_step` (not `bank`).
- **Claims removed:** 9. They are V1-008 (menu/pause semantics), V1-014 (36 phases), V1-019 (heaviest step), V1-021 (Leontief matrix), V1-022 (124,000 decision logic), V1-024 (8.48 GB), V1-028 (delta compression), V1-035 (plansociaux description), V1-036 (classer_joueur description).
- **New corpus-derived claims added:** 16. They are the exact f_25b6690 smoothing branch; the f_1c37d70 hour accumulator and date construction; the millisecond-timestamp interpretation of 0x4813c0a8 (f_25cafe0/f_25dd540/f_25a7630); the day-serial interpretation of 0x464916a8; the exact 0x50ef4788 increment expression; the f_239f3b0 initialisation; bank_step's rounding rule; the 1×prof_begin/56×prof_end count; mutex_scope(3,1) plus the IDME log string; the 19 Etape labels and the missing Etape10; the unsigned -1 return of the stage driver; es_base modes 2/3/4/7; the NATI 4-byte tag + 4-byte size header; the 299-iteration nation loop in f_8c5310 (and f_8c5500's `while (iVar6 < 300)`, also 299 iterations); the 4,000,000 × 16-byte variable-store allocation; and the 0x41b8/0x1f60/0xad0 strides tied to their base globals.

## T-007 structural corrections (D-016 restyle)

Two structural facts in the stage driver were corrected in the volume during the D-016 restyle. Both are verified against the corpus of record. The 38 rows above keep their original verdicts; nothing else in them changed.

| claim_id | what changed | corpus_evidence | verdict |
| --- | --- | --- | --- |
| V1-039 | The `< 7` early-work gate is a two-conjunct condition: `((int)param_1 < 7) && (bVar42 == 0)`. `bVar42` is set to `_DAT_4649165c != 0` at 2045459, stored at 2045460 and reloaded at 2046210. The volume pseudocode now reads `stage < 7 and mode_value == 0` with `mode_value = global 4649165c`, and mermaid G7 reads `stage < 7 and 4649165c == 0`. | `calc_monde_debut_tour_etapes` 0x1409f40 corpus 2045459-2045460, 2046210, 2046396 | corrected |
| V1-040 | The `< 3` block has a real early return: corpus 2045876-2045879 sets the return value to 2 and jumps to 2048636, which runs `f_12bd280(); return (ulonglong)uVar41;`. Etapes 2-19 are skipped on that path. The section 3 mermaid now carries the edge `D3 -> RET["early return 2, skips Etapes 2-19"]`. | `calc_monde_debut_tour_etapes` 0x1409f40 corpus 2045876-2045879, 2048636-2048638 | corrected |

## Raw decompiler excerpts moved out of the volume body (D-016)

These are the decompiler excerpts that the four volume pseudocode blocks contained before the T-007 restyle. They are kept here, with the corpus ranges, so the structural claims stay checkable while the volume body stays readable. The `> Evidence:` footer under each restyled block cites the same ranges.

### Temporal core

`f_25b6690` corpus 4847341-4847356; `f_1c37d70` corpus 3294553-3294761; `calc_monde_fin_tour` excerpt corpus 2050082-2050107.

```c
// f_25b6690 @ 0x25b6690 (corpus 4847341) -- seconds from tick counter
float f_25b6690(void) {
    if (_DAT_70ee6548 != 0.0) {
        d = ((double)_DAT_70ee651c + _DAT_70ee6540) * _DAT_70ee6548
            + (1.0 - _DAT_70ee6548) * _DAT_70ee6528;
        if ((double)_DAT_70ee6524 * _DAT_70ee6530 < d - _DAT_70ee6528)
            d = _DAT_70ee6528 - (double)_DAT_70ee6524 * _DAT_70ee6530;
        return (float)(d / (double)_DAT_70ee6524);
    }
    return (float)(_DAT_70ee6528 / (double)_DAT_70ee6524);
}

// f_1c37d70 @ 0x1c37d70 (corpus 3294553) -- game-hour accumulator and date build
undefined4 f_1c37d70(void) {
    mutex_scope(0xf, 1);
    uVar12 = 0;
    auStackX_10[0] = 0;
    if (_DAT_46209e1c == 0) {
        if (_DAT_464ddc84 != 0) {
            fVar14 = (float)f_25b6690();
            _DAT_464ddc8c = (fVar14 - _DAT_46054ecc) - _DAT_464ddc88 * 24.0;
            uVar12 = 4294967295U;
            auStackX_10[0] = 4294967295U;
        }
        if (_DAT_46495ec4 < 1) {
            if ((_DAT_322a4f4 == 0.0) || ((_DAT_464de20c + 2U & 4294967291U) == 0)) {
                fVar14 = (float)f_25b6690();
                fVar14 = fVar14 - _DAT_46054ecc;
                _DAT_464ddc8c = fVar14;
            } else {
                if (_DAT_46495ec4 == -1) {
                    _DAT_46495ec4 = 0;
                    _DAT_464ddc8c = (float)f_25b6690();
                    _DAT_464ddc8c = _DAT_464ddc8c - _DAT_46054ecc;
                }
                if (_DAT_4649165c == 0 && _DAT_4649166c == 0) {
                    fVar14 = (float)f_25b6690();
                    fVar14 = ((fVar14 - _DAT_464ddc8c) - _DAT_46054ecc)
                             / (_DAT_464ddc88 / _DAT_322a4f4);
                }
                else
                    fVar14 = (-_DAT_464ddc8c - _DAT_46054ecc)
                             / (_DAT_464ddc88 / _DAT_322a4f4);
                fVar16 = fVar14;
                if (_DAT_4649165c == 0) {
                    if (1.0 < _DAT_464ddc8c) _DAT_464ddc8c = 1.0;
                    fVar16 = 1.0;
                    if (fVar14 <= 1.0) fVar16 = fVar14;
                }
                fVar15 = 0.0;
                if (0.0 <= fVar16) fVar15 = fVar16;
                fVar15 = _DAT_464ddc9c + fVar15;
                _DAT_464ddc9c = fVar15;
                fVar16 = floorf(fVar15);
                if (fVar16 != 0.0) {
                    _DAT_464ddc9c = fVar15 - fVar16;
                    fVar15 = floorf((_DAT_464ddc98 + fVar16) / 24.0);
                    if (((float)_DAT_4649481c < fVar15) &&
                        ((((_DAT_324da98 != 0 || ((float)(_DAT_46494820 + 1) < fVar15))
                           || (cVar1 = f_91e5c0(), fVar14 = _DAT_464ddc8c, cVar1 == 0))
                          && (_DAT_4649165c == 0))) {
                        fVar14 = (float)f_25b6690();
                        fVar14 = (fVar14 - _DAT_46054ecc)
                                 - (_DAT_464ddc88 / _DAT_322a4f4) * _DAT_464ddc9c;
                        fVar16 = 0.0;
                        _DAT_464ddc9c = 1.0;
                        _DAT_464ddc8c = fVar14;
                    }
                    _DAT_464ddc98 = _DAT_464ddc98 + fVar16;
                }
            }
        } else {
            fVar14 = (float)f_25b6690();
            _DAT_464ddc8c = (fVar14 - _DAT_46054ecc) - (float)_DAT_46495ec4;
            fVar15 = (float)f_25b6690();
            fVar14 = _DAT_464ddc8c;
            fVar16 = (float)_DAT_46495ec4;
            fVar15 = floorf((fVar15 - _DAT_464ddc8c) - _DAT_46054ecc);
            if (fVar16 <= fVar15) fVar16 = fVar15;
            _DAT_464ddc98 = _DAT_464ddc98 + fVar16;
            _DAT_46495ec4 = -1;
            auStackX_10[0] = 1;
            uVar12 = 1;
        }
        if (_DAT_4649165c == 0) {
            _DAT_46054ecc = (float)f_25b6690();
            _DAT_46054ecc = _DAT_46054ecc - _DAT_464ddc8c;
        } else {
            _DAT_46054ecc = -fVar14;
        }
        if (_DAT_46209e20 != 0) {
            _DAT_1844bd08 = _DAT_464ddc98;
            _DAT_1844bd0c = _DAT_464ddc9c;
        }
    } else {
        _DAT_464ddc98 = _DAT_1844bd08;
        _DAT_464ddc9c = _DAT_1844bd0c;
    }
    iVar10 = (int)_DAT_464ddc98;
    puVar4 = f_25a7630(&tmp, 0, 0, 0, iVar10, 0, 0);       // ms from hours
    puVar4 = f_25a7b90(&tmp2, _DAT_47938838, *puVar4);
    f_25a7940(&DAT_4813c0a8, &lStackX_8, *puVar4);          // write ms timestamp
    f_25a7880(); f_25a7880(); f_25a7880();
    _DAT_46494828 = f_25e9b60(&DAT_4813c0a8);               // year
    _DAT_46494824 = f_25df0d0(&DAT_4813c0a8);               // month
    _DAT_46494814 = f_25d2b10(&DAT_4813c0a8);
    lVar5 = f_25dd540(&DAT_4813c0a8);
    iVar8 = _DAT_464916a8;
    _DAT_46494808 = (undefined4)lVar5;
    _DAT_464916a8 = f_25cafe0(_DAT_4813c0a8);               // day serial
    _DAT_46494818 = f_25da840(&DAT_4813c0a8);
    plVar6 = f_25a7610(&lStackX_8, _DAT_46494828, 1, 1);
    iVar3 = f_25cafe0(*plVar6);
    _DAT_46494810 = (_DAT_464916a8 - iVar3) + 1;            // day-of-year
    f_25a7880();
    plVar6 = (longlong *)f_25a7630(&lStackX_8, 0, 0, 0, iVar10, 0, 0);
    _DAT_4649481c = f_25cafe0(*plVar6);
    f_25a7880();
    if (((_DAT_464916a8 != iVar8) && (_DAT_46494824 == 2)) &&
        ((_DAT_46494814 == 1 && (lStackX_8._0_2_ = 1, 1 < _DAT_322a3f4)))) {
        do {
            // nation pair loop body 3294700-3294731:
            // f_1a414d0 / f_1a216c0 with masks 0xfffeffffffffffff,
            // 0xffff7fffffffffff, 0x1000000000000
            // (not read: full loop body)
        } while ((short)lStackX_8 < _DAT_322a3f4);
    }
    plVar6 = f_25a7610(&lStackX_8, _DAT_46494828 + 1, 1, 1);
    plVar7 = f_25a7610(&lStackX_8, _DAT_46494828, 1, 1);
    iVar8 = f_25cafe0(*plVar6);
    iVar3 = f_25cafe0(*plVar7);
    _DAT_322a4e0 = (float)(iVar8 - iVar3);
    f_25a7880(); f_25a7880();
    plVar6 = f_25a7610(&lStackX_8, _DAT_46494828, 1, 1);
    plVar7 = f_25a7610(&lStackX_8, _DAT_46494828 - 1, 1, 1);
    iVar8 = f_25cafe0(*plVar6);
    iVar3 = f_25cafe0(*plVar7);
    _DAT_322a4e4 = (float)(iVar8 - iVar3);
    f_25a7880(); f_25a7880();
    _DAT_46495e7c = (int)(_DAT_46494828 - _DAT_4649482c) % 0x1d + 1;
    _DAT_464ddc90 = _DAT_464ddcb0;
    if (_DAT_46209e1c == 0) {
        _DAT_464ddc90 = (double)(_DAT_464ddc9c + _DAT_464ddc98);
    }
    _DAT_4649480c = (int)((_DAT_464ddc90 - (double)_DAT_464ddc98) * 60.0);
    if (_DAT_4649480c < 0) {
        _DAT_4649480c = 0;
    } else if (0x3b < _DAT_4649480c) {
        _DAT_4649480c = 0x3b;
    }
    mutex_scope(0xf, 0);
    return uVar12;
}

// calc_monde_fin_tour @ 0x140e5d0 (corpus 2048643) -- only advancing write to 50ef4788
// (not read: the other 552 corpus lines of the function, 2048643-2051194)
if (_DAT_4649165c == 0 && _DAT_463d1a7c == 0) {
    plVar17 = f_25a7610(&tmp1, 0, 0, (int)_DAT_50ef4788);
    uVar13 = f_25e9b60(plVar17);
    plVar17 = f_25a7610(&tmp2, uVar13 + 1, 1, 1);          // 1 Jan next year
    plVar18 = f_25a7610(&tmp3, 0, 0, (int)_DAT_50ef4788);
    uVar13 = f_25e9b60(plVar18);
    plVar18 = f_25a7610(&tmp4, uVar13, 1, 1);              // 1 Jan this year
    iVar40 = f_25cafe0(*plVar17);
    iVar14 = f_25cafe0(*plVar18);
    f_25a7880(); f_25a7880(); f_25a7880(); f_25a7880();
    dVar55 = ((double)(iVar40 - iVar14) / 365.0) * 0.038461538461538464 * 365.25
             + _DAT_50ef4788;
    iVar40 = bank_step(dVar55);
    if (_DAT_464916a8 == iVar40 + 3) DAT_4649484b = 1;
    iVar40 = bank_step(dVar55);
    if (_DAT_464916a8 == iVar40 + 4) {
        DAT_4649484b = 0;
        _DAT_50ef4788 = dVar55;
    }
}
```

### Daily driver

`calc_monde` corpus 2029428 head; ordered call sites 2030531-2033073; body 15,853 lines (2029428-2045280).

```c
// calc_monde @ 0x13f3290 (corpus 2029428, 15,853 lines)
// (not read: the full 93,356-byte body; only the head and the ordered call sites were read)
void calc_monde(uint param_1, ...) {
    mutex_scope(3, 1);                       // corpus 2030531
    // (not read: setup before the first temporal call)
    f_1c37d70();                             // corpus 2030564, game-hour/date update
    // (not read: early scenario setup)
    prof_begin(pppdVar20);                   // one call, corpus 2030705
    // observed order (calc_monde_steps.txt, observed not decompiled):
    // (not read: selected call-site names in first-occurrence order; arguments and per-call guards omitted)
    maj_sat();
    maj_chemin_nation();
    maj_agenda();
    gere_requetes();
    gere_rdv();
    gere_histo();
    calc_actions();
    calc_unites();
    org_adh_loop();
    reactions_process();
    f_1053e40();
    calc_monde_debut_tour_etapes(_DAT_324da9c);   // corpus 2031956
    // (not read: the branching around the call, corpus 2031956-2032040)
    gereAccordMainOeuvreLissee();            // only when (int)_DAT_324da9c < 0
    // (not read: the remaining daily subsystems; order taken from the call list)
    _DAT_324daa0 = calc_monde_fin_tour(_DAT_324daa0);   // corpus 2033073 call site (2048643 is the calc_monde_fin_tour header)
    // ... 56 x prof_end() in total
}
```

### Stage driver

`calc_monde_debut_tour_etapes` corpus 2045281-2048638. The two corrected facts are at 2046396 (two-conjunct `< 7` gate) and 2045876-2045879 -> 2048636 (early return 2).

```c
// calc_monde_debut_tour_etapes @ 0x1409f40 (corpus 2045281) -- early gates + 19 Etape labels
ulonglong calc_monde_debut_tour_etapes(uint param_1) {
    f_2793510();
    // label "calc_monde_debut_tour"
    if (param_1 == 0) {
        DAT_326caa01 = '\x01';
        DAT_326caa02 = '\x01';
    }
    /* label "Debut" */
    // (not read: setup/logging block before the <1 gate)
    // early-work gate, NOT Etape1
    if ((int)param_1 < 1) {
        if (_DAT_4649165c == 0) {
            // (not read: full logging block 2045629-2045663; calls include
            // f_85fdf0, f_25e9800, f_25e97b0, f_25e9760, f_25e9670,
            // f_1ecb9e0, f_959710)
        }
        f_140e000();
        if (DAT_4649484b == '\0' && (int)param_1 < 1) f_1a252a0((undefined1 *)G_50ef2200);
        if (_DAT_4649165c < -1) {
            // nation loop: f_9ebf20, f_9eac60, f_9defe0, f_9de5d0,
            // f_1a38620, f_1a36d10, f_9b4df0, f_977360, f_9e8fb0
            // (not read: full loop)
        }
        // (not read: clock/timing guard 2045767-2045772)
    }
    // early-work gate, NOT Etape2
    if ((int)param_1 < 2) {
        f_1cc9370();
        f_1cc8d90(_DAT_464916a8, ...);
        // (not read: clock/timing guard 2045781-2045794)
    }
    /* label "Etape1" (corpus 2045796) */
    if ((int)param_1 < 3) {
        if (DAT_326caa01 != '\0') {
            puVar12 = (undefined4 *)f_1a4e8d0();
            f_1a50500(puVar12);
        }
        piVar13 = (int *)f_1a4e8d0();
        uVar39 = 0;
        piVar36 = (int *)(ulonglong)(bVar42 ^ 1);
        f_1a4d9f0(piVar13, _DAT_4649168c, bVar42 ^ 1, 0);
        if ((extraout_AL == '\0') && (DAT_463d1b41 == '\0')) {
            uVar41 = 2;
            DAT_326caa01 = '\0';
            goto LAB_7ff647c7df98;   // early return path
        }
        DAT_326caa01 = '\x01';
        // (not read: clock/timing guard 2045882-2045924)
    }
    // (not read: the rest of the Etape1 block up to the Etape2 label at 2046212)
    /* label "Etape2" (corpus 2046212) */
    if ((int)param_1 < 4) {
        f_159bfe0();
        for (iVar31 = *(int *)(_DAT_5d85ada0 + 4); 0 < iVar31;
             iVar31 = *(int *)((longlong)iVar31 * 0xad0 + 4 + _DAT_5d85ada0)) {
            // f_21bf540 / f_10f2060 on the 0x41b8 profile record
            // (not read: full loop)
        }
        // (not read: clock/timing guard 2046310-2046327)
    }
    if ((int)param_1 < 5) {
        f_1c386d0();
        // (not read: clock/timing guard 2046339-2046367)
    }
    if ((int)param_1 < 6) {
        f_1c408f0();
        // (not read: clock/timing guard 2046379-2046395)
    }
    if ((int)param_1 < 7) { /* clock/timing only */ }   // T-007: corpus 2046396 also requires (bVar42 == 0)
    // (not read: the rest of the Etape2 block up to the Etape3 label at 2046427)
    /* label "Etape3" (corpus 2046427) */
    // f_193af90 on 0x41b8 * index + _DAT_1e660190 (corpus 2046501)
    // labels Etape4..Etape20 (19 labels total; no Etape10)
    // (not read: the remaining stage bodies)
    uVar41 = 4294967295U;   // (uint)-1; corpus writes all bits set
    f_12bd280();
    return (ulonglong)uVar41;
}
```

### Serializer and layout

`Cdatabase__es_base` corpus 2072351-2073839; `f_147d030` corpus 2111594-2111600; `f_8c5310` corpus 114686-114716; `variable_store_get` corpus 105569-105588.

```c
// Cdatabase__es_base @ 0x143acd0 (corpus 2072351)
ulonglong Cdatabase__es_base(uint param_1, ushort *param_2, ulonglong param_3, undefined8 param_4) {
    // (not read: setup and header path handling before the mode dispatch)
    puVar30 = variable_store_get();
    f_1376270(...);
    if (param_1 == 5) {
        uVar58 = io_read(puVar30, 0x40d9908);
    } else if (param_1 == 1) {
        uVar58 = f_146c790(puVar30, 1, p2, ...);
    } else if (param_1 == 6) {
        uVar58 = io_write(puVar30, 0x40d9908);
    } else if (param_1 == 2) {
        uVar58 = f_146c790(puVar30, 2, p2, ...);
    } else if (param_1 == 3) {
        uVar58 = f_146c790(puVar30, 3, p2, ...);
    } else if (param_1 == 4) {
        inlined_stub_45(puVar30, 0, 0x40d9908);
        uVar58 = f_146c790(puVar30, 3, p2, ...);
    } else if (param_1 == 7) {
        uVar58 = f_146c790(puVar30, 3, p2, ...);
    }
    // (not read: the rest of the 39,622-byte function)
}

// f_147d030 @ 0x147d030 (corpus 2111365) -- NATI record header
uStack_12088 = 0x645310;          // record size
uStack_1208c = 0x4e415449;        // "NATI"
f_8604f0(fd, &uStack_1208c, param_3);
f_8604f0(fd, &uStack_12088, param_3);

// f_8c5310 @ 0x8c5310 (corpus 114686) -- nation scan, only when param_1 == 0x166
float f_8c5310(short param_1, int param_2) {
    if (param_1 == 0x166) {
        piVar1 = (int *)G_5c841f1c;
        lVar2 = 299;
        lVar3 = 0x645310;
        fVar5 = 0.0;
        do {
            if (0 < *piVar1) {
                dVar4 = f_1a37c40((longlong)param_2 * 0x4068 + lVar3
                                  + _DAT_46208908 + 0xcbc8);
                if (fVar5 <= (float)dVar4) fVar5 = (float)dVar4;
            }
            lVar3 = lVar3 + 0x645310;
            piVar1 = piVar1 + 1;
            lVar2 = lVar2 + -1;
        } while (lVar2 != 0);
        return fVar5;
    }
    dVar4 = f_1a37c40((longlong)param_1 * 0x645310 + _DAT_46208908
                      + (longlong)param_2 * 0x4068 + 0xcbc8);
    return (float)dVar4;
}

// variable_store_get @ 0x8b9490 (corpus 105569)
// The guard indexes a nested pointer chain at [field 0x514e64 in variable_store_get];
// the displacement is a field offset, not an address (D-010).
undefined *variable_store_get(void) {
    if (*(int *)(*(longlong *)(*(longlong *)(unaff_GS_OFFSET + 0x58)
                + (ulonglong)_DAT_73be39b0 * 8) + 0x514e64) < _DAT_73ab228) {
        operator_new(&DAT_73ab228);
        if (_DAT_73ab228 == -1) {
            f_27933fc(&DAT_32d1920, 0x10, 4000000, f_1cc89c0);
            _DAT_73ab220 = 1;
            f_1cca000((int *)&DAT_32d1920);
            f_27933e4();
            operator_delete(&DAT_73ab228);
            return &DAT_32d1920;
        }
    }
    return &DAT_32d1920;
}
```
