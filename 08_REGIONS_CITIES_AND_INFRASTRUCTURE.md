# Volume 8: Regions, Cities, Territory & Infrastructure

---

## 1. Administrative & Spatial Hierarchy

The corpus implements a three-level record hierarchy: a nation array of records with stride `[qty 0x645310 stride]`, a region array with stride `[qty 0x910 stride]`, and a city array with stride `[qty 0x9cf8 stride]`. The nation-to-region link and the region-to-region link are two different mechanisms: the nation record stores a first-region id at `[field 0x2f0 in NATI]`, and each region stores the next region id at `[field 0x0 in REGI]`.

```mermaid
flowchart TD
    N["NATI record, stride 0x645310"]
    R["REGI record, stride 0x910"]
    V["VILL record, stride 0x9cf8"]
    N -->|"field 0x2f0: first region id"| R
    R -->|"field 0x00: next region id"| R
    R -->|"field 0x14: int nation id"| N
    R -->|"field 0x28: tagged value read as NATI index"| N
    R -->|"field 0x38: tagged value read as company index"| C["0xb88-stride company array"]
    V -->|"field 0x04: grid index"| G["0x8808c-stride grid"]
```

```pseudocode
For each nation record:
    first_region = nation.first_region_id          // nation field +0x2f0
    while first_region > 0:
        visit region first_region
        first_region = region.next_region_id       // region field +0x00

To unlink one target region from a nation's list:
    if nation.first_region_id == target:
        nation.first_region_id = target.next_region_id
    else:
        current = nation.first_region_id
        while current > 0:
            if current.next_region_id == target:
                current.next_region_id = target.next_region_id
                stop
            current = current.next_region_id
```
> Evidence: f_85d580 [fn 0x85d580], corpus 41991 and 42005; f_1c01830 [fn 0x1c01830], corpus 3265819-3265832. Strides: `[qty 0x645310 stride]` from f_1399d60:1991897, `[qty 0x910 stride]` from f_139bb40:1993157.

### 1.1 Nation Array and Activation

The nation array base is `[global 0x46208908 i64]`. The loader `[fn 0x1399d60 f_1399d60]` advances the destination pointer by `[qty 0x645310 stride]` for every record (corpus line 1991897), and `[fn 0x85d580 f_85d580]` indexes a record with the same stride (corpus line 41991; the multiplying expression is in the pseudocode block). `Cdatabase__es_base` calls the loader with a count (corpus line 2074049). That count is set to `[qty 300 nation-slots]` for the normal path and to the live counter `[global 0x322a3f4 i32]` for the other path (corpus lines 2074045-2074047). The live counter is initialised to 1 in `[fn 0x1cbac20 f_1cbac20]` (corpus line 3345979).

`nation_is_active` is not a flag test: it returns true when `[field 0x2f0 in NATI]` is non-zero (`[fn 0x1a47850 nation_is_active]`, corpus line 2942160). So a nation is active when its first-region field is non-zero. `[fn 0x1c30ef0 f_1c30ef0]` allocates a region and stores the new id into that same field when the nation has none (corpus lines 3290386-3290390).

### 1.2 Region Record, Identity and the Linked List

The region array is loaded by `[fn 0x139bb40 f_139bb40]`, which adds `[qty 0x910 stride]` per record (corpus line 1993157). Its count is the live region counter `[global 0x463d1b48 i32]`, passed at corpus line 2074132. The counter is initialised to 1 at corpus line 3345982 and incremented after each allocation at corpus line 3291928. The shipped data file's extent divided by the record size gives 1,309 records, but the corpus does not contain that number as a literal; the loader uses the counter. The old `0x52d` is 1,325, not 1,309: `0x52d` occurs 202 times in the corpus as an unrelated bound, while 1,309 = `0x51d` and its only two literals are unrelated assignments.

The region allocator `[fn 0x1c32e50 f_1c32e50]` writes the record at index `[global 0x463d1b48 i32]` (corpus lines 3291881-3291906):

| Field | Evidence | Meaning established by the code |
| :--- | :--- | :--- |
| `[field 0x0 in REGI]` | `[fn 0x1c32e50 f_1c32e50]` line 3291885 | next region id / previous list head |
| `[field 0x4 in REGI]` | `[fn 0x1c32e50 f_1c32e50]` line 3291884 | self region id |
| `[field 0x14 in REGI]` | `[fn 0x1c32e50 f_1c32e50]` line 3291886 | integer nation id |
| `[field 0x28 in REGI]` | `[fn 0x1c32e50 f_1c32e50]` lines 3291889-3291890 | 16-byte tagged value (double payload, tag at `[field 0x30 in REGI]`) |
| `[field 0x38 in REGI]` | `[fn 0x1c32e50 f_1c32e50]` lines 3291898-3291901 | 16-byte tagged value |
| `[field 0x48 in REGI]` | `[fn 0x1c32e50 f_1c32e50]` line 3291897 | integer payload |

The `[field 0x28 in REGI]` value is read as a nation reference: `[fn 0x857ff0 AVTerritoire__calc_prix_territoire]` reads it, converts it to an integer and multiplies by `[qty 0x645310 stride]` to index the nation array (corpus lines 38187-38189). The `[field 0x38 in REGI]` value is not a nation reference: `[fn 0x1992f10 f_1992f10]` reads it as an integer and multiplies by `[qty 0xb88 stride]` to index the company array `[global 0x5d859dc8 i64]` (corpus line 2803960).

The list is a genuine singly-linked list because both the store and the traversal are visible. The store is `[fn 0x1c01830 f_1c01830]` (corpus lines 3265819-3265832): it replaces the nation's `[field 0x2f0 in NATI]` head or rewrites the predecessor's `[field 0x0 in REGI]`. The traversal is `[fn 0x85d580 f_85d580]` (corpus lines 41991 and 42005).

### 1.3 City Record

The city array base is `[global 0x26e03518 i64]`. `[fn 0x13a07b0 f_13a07b0]` allocates count × `[qty 0x9cf8 stride]` bytes and copies the records (corpus lines 1996505-1996515); the count comes from the save stream at corpus line 2076846.

The corpus exposes these city fields:

| Field | Evidence | Meaning established by the code |
| :--- | :--- | :--- |
| `[field 0x0 in VILL]` | `[fn 0x12f2010 f_12f2010]` line 1937098; `[fn 0x12f3de0 f_12f3de0]` line 1938482; `[fn 0x13000f0 f_13000f0]` line 1942298 | integer handle/id: the value is OR-ed with `[qty 0x2000 flag-bit]` at 1937098, compared for identity at 1938482, and used as an index into a 16-bit lookup array at 1942298 |
| `[field 0x4 in VILL]` | `f_937500` lines 192372-192373 | integer multiplied by `[qty 0x8808c grid-stride]` to index a per-city grid |
| `[field 0x8 in VILL]` | `gestion_popu_warville` line 4125207 | uint compared to `2` on the combat path |
| `[field 0x20 in VILL]` | `f_92a0b0` line 184420 | integer multiplied by `[qty 0x750 stride]` to index a `[qty 0x750 stride]` array |
| `[field 0x28 in VILL]` | `gestion_popu_warville` line 4125208 | day stamp compared to `[global 0x464916a8 i32] - 1` |
| `[field 0x2c in VILL]` | `gestion_popu_warville` line 4125201 | integer tested for `< 1` |
| `[field 0x34 in VILL]` | `f_f92060` line 1439686 | flag word OR-ed with `[qty 0xc0000 flag-bits]` |

`[field 0x0 in VILL]` is an integer handle/id, read at the three sites above; the corpus does not label it "unique city id", but the accesses are consistent with an id used as a lookup key. The old `+0x04` parent `REGI` id, `+0x08` peace/contested/siege enumeration, `+0x28` "last bombardment", and `+0x34` mayor pointer are not supported. The code has a combat branch that tests `[field 0x8 in VILL] == 2` and a day stamp at `[field 0x28 in VILL]`, but it does not label the state numbers or the stamp as siege/bombardment.

### What the code does not establish

- A "unique city id" label for `[field 0x0 in VILL]`; the field is used as a handle and lookup index, not labelled further.
- A parent region id at `[field 0x4 in VILL]`; the field is used as a grid index.
- The meanings "peace / contested / siege" for the three values of `[field 0x8 in VILL]`.
- A mayor or governor `PERS` pointer in the city record.
- Municipal hospitals, schools or police as city fields.
- A fixed 1,309 region count; the count is the runtime counter `[global 0x463d1b48 i32]`.

---

## 2. Regional GDP Calculation & National Factoring

The section title is retained for continuity, but the corpus does not implement a bottom-up regional GDP sum. `calc_nation` walks the `[field 0x2f0 in NATI]` region chain and calls `[fn 0x1c3ef40 f_1c3ef40]` on each region (the decompiler prints `param_1 + 0x5e` because `param_1` is `double *`, so 0x5e * 8 = 0x2f0); `f_1c3ef40` itself walks the same `[field 0x2f0 in NATI]` chain. The 365-day smoother is applied to two nation fields. The old production formula, the Leontief subtraction and the national raw-GDP sum are removed.

```mermaid
flowchart LR
    CN["calc_nation: NATI record"] -->|"field 0x2f0: region id (printed as param_1 + 0x5e)"| R1["REGI chain"]
    CN -->|"calls f_1c3ef40"| RF["f_1c3ef40 region update"]
    RF -->|"field 0x2f0: region id"| R2["REGI chain"]
    RF -->|"writes field 0x10c"| F1["REGI field 0x10c"]
    CN -->|"EMA 0.9972603 / 0.0027397275"| F3["NATI fields 0x64346c and 0xc868c"]
    F14["f_141b2d0 (separate function)"] -->|"field 0x2f0: region id"| R3["REGI chain"]
    F14 -->|"compares field 0x38"| F2["tagged value compare"]
```

```pseudocode
Function update_nation(nation):
    if the nation is not active: return
    region = nation.first_region_id
    while region != 0:
        update region from nation        // f_1c3ef40
        region = region.next_region_id   // region field +0x00

    if the budget/score path is enabled:
        score = lookup_score(nation)

        // first smoothed index: EXCI (repo label; no corpus name)
        target = nation.exci - ((score*10.0 - 2.0)*4.0)/100.0
        target = clamp(target, 0.0, 1.0)
        nation.exci = nation.exci*0.9972603 + target*0.0027397275   // constants 0x14134a0:2052049-2052050

        // second smoothed index: no corpus name
        candidate = ((score*10.0 - 5.0)*4.0)/100.0 + nation.control
        if a condition is true and nation.control > candidate:
            target = nation.control
        else:
            target = clamp(candidate, 0.0, 1.0)
        nation.control = nation.control*0.9972603 + target*0.0027397275   // constants 0x14134a0:2052083-2052084

Function update_regions_from_nation():   // f_1c3ef40
    for each nation:
        region = nation.first_region_id
        while region != 0:
            if the region's linked record is non-zero:
                write the nation-derived value into the region
            region = region.next_region_id

Function refresh_tagged_region_values(value):   // f_141b2d0
    for each nation:
        region = nation.first_region_id
        while region != 0:
            if region's tagged +0x38 value matches value:
                replace it with value
            region = region.next_region_id
```
> Evidence: calc_nation [fn 0x14134a0], corpus 2051819-2056518 (region walk 2052660-2069; EMA 2052036-2052050 and 2052065-2084); f_1c3ef40 [fn 0x1c3ef40], corpus 3298856-3298872 (the +0x2f0 walk); f_141b2d0 [fn 0x141b2d0], corpus 2056596-2056622 (the +0x2f0 walk with the +0x38 compare).

### 2.1 What `calc_nation` Walks

`[fn 0x14134a0 _geolgame__Cgps__calc_nation]` first checks whether the nation is active. In the `else` branch of the guard at corpus lines 2052585/2052659 (the guard compares `[global 0x4649165c i32]` with -3) it reads a region id from `[field 0x2f0 in NATI]` and walks that region chain (corpus lines 2052660-2069, including the loop-back at 2069), calling `[fn 0x1c3ef40 f_1c3ef40]` on each region. The decompiler prints `param_1 + 0x5e` because `param_1` is `double *`; 0x5e * 8 = 0x2f0.

`f_1c3ef40` computes `[field 0x10c in REGI] = [field 0xc4a8 in NATI] * [field 0x118 in REGI]` (corpus lines 3298434-3298436), then dispatches an event tagged `[tag 0x434c5545 "CLUE"]` (corpus line 3298465). It is region-level value propagation, not a sum of regional GDP into a national total. The raw statements are quoted in `08_CLAIMS.md` (raw region-update excerpt; see also 08-C018).

`f_1c3ef40` also walks the `[field 0x2f0 in NATI]` region chain at corpus line 3298864; this is the `+0x2f0` walk reachable from `calc_nation`. A different function, `[fn 0x141b2d0 f_141b2d0]`, walks every active nation's `[field 0x2f0 in NATI]` chain and compares the `[field 0x38 in REGI]` tagged value with `f_22f02a0` (corpus lines 2056596-2056622). Neither walk accumulates a GDP field.

The old identification of `[field 0x30ce44 in NATI]` as NPIB is not supported. `[field 0x30ce44 in NATI]` is a float field (13 corpus hits), and `[fn 0x23e9640 f_23e9640]` uses it inside a `(value * 0.7) / 5000.0` formula (corpus line 4548009), but the corpus never labels it GDP, NPIB or PIB. The tag `[tag 0x43414c4a "CALJ"]` reads and writes it in `[fn 0x2593bd0 get_variable]` (corpus lines 4832529-4832534), which still does not establish GDP semantics. The repo-derived `nation_fields.tsv` names `PIB1/PIB2/PIB3` at `[field 0x5410 in NATI]`, `[field 0x5414 in NATI]`, `[field 0x5418 in NATI]` and `EPIB` at `[field 0xc4a8 in NATI]`; the field `[field 0x30ce44 in NATI]` is not among them. That file is a derived map, not a shipped table, so it is only a lead, not proof.

### 2.2 The 365-Day Smoother Is Real, but It Is Not GDP

The exact constants exist twice in `[fn 0x14134a0 _geolgame__Cgps__calc_nation]`:

- first smoothed index (corpus lines 2052036-2052050): `target = clamp(old - ((score*10.0 - 2.0)*4.0)/100.0, 0, 1)`; then `new = old*0.9972603 + target*0.0027397275`.
- second smoothed index (corpus lines 2052065-2052084): `candidate = ((score*10.0 - 5.0)*4.0)/100.0 + old`; `target = clamp(candidate, 0, 1)` unless a condition holds and `old > candidate`, in which case `target = old`; then `new = old*0.9972603 + target*0.0027397275`.

The two coefficients sum to 1.0000000275, and `0.0027397275` is the source's decimal approximation of `1/365` (1/365 = 0.0027397260...); so the form is a 365-day exponential smoother. But the smoothed object is `[field 0x64346c in NATI]` / `[field 0xc868c in NATI]`, each clamped to `[0,1]` before the blend. `[fn 0xa5f510 f_a5f510]` reads `[field 0x64346c in NATI]` only to choose between a fallback constant and an `LCPE` value (corpus line 366826), and `[fn 0x1d9cbd0 f_1d9cbd0]` aggregates it across active nations as `* 100.0` for a UI/variable read (corpus line 3498469). Neither function treats it as GDP. The old "GDP EMA" claim is therefore replaced by: a 365-day EMA exists, but its target is a unit-interval index with no corpus name (the repo-derived `nation_fields.tsv` labels it EXCI), not national GDP.

### 2.3 Tax Retention and Equalisation

No retention ratio in `[0.10, 0.70]` was found. `[fn 0x1c904b0 init_taxe]` iterates nations and regions (corpus lines 3319355-3319410) and builds tax relationships, but it contains no clamp of a regional tax ratio to that range. The searches were: every function containing both a `[qty 0x910 stride]` region index and the literals `0.1` and `0.7`; no function contains a two-sided clamp on a tax share. Equalisation transfers likewise have no implementing function: no function takes a region surplus and adds it to another region or to a national pool.

### What the code does not establish

- A regional value-added formula with capital, workers, modernity or health terms.
- A Leontief intermediate-input subtraction over 364 sectors.
- A national raw-GDP sum over `[field 0x2f0 in NATI]` chains.
- The identification of `[field 0x30ce44 in NATI]` as GDP / NPIB.
- A GDP-targeted 365-day EMA (the EMA exists, but on two other fields).
- A regional retention ratio in `[0.10, 0.70]`.
- Equalisation transfers between regions.
- The `CBAT` / `CRPP` counts 620,000 and 248,000 as code constants; only the strides `[qty 0x68 stride]` and `[qty 0x1cc0 stride]` are verified by the loaders `[fn 0x96afa0 f_96afa0]` and `[fn 0x96a840 f_96a840]`.

---

## 3. Territorial Annexation & Secession Dynamics

The action ids in this area are data-table rows, and four of them are also code literals. The corpus does not show the pointer rewrite the old text described at `[field 0x28 in REGI]`. It does show list-traversal code, a list-unlink function, and an action handler that writes the `[field 0x38 in REGI]` tagged value.

```mermaid
flowchart TD
    A["332031 Frontières modifiées"] -->|"0x510ff"| H["f_19931a0"]
    B["332032 Annexion de certains territoires occupés"] -->|"0x51100"| H
    C["332040 Pays annexé"] -->|"0x51108"| E["exec_contrat / set_action_sp2"]
    D["332046 Indépendance"] -->|"0x5110e"| E
    H -->|"NATI field 0x2f0"| R["REGI chain"]
    H -->|"f_1992f10"| T["REGI field 0x38 tagged value"]
```

```pseudocode
When the action id is 332031 (Frontières modifiées):
    for each first nation A (index 1..nation_count):
        if A fails f_1a439e0 and its flag bit 2 is not set: continue
        for each second nation B (index 1..nation_count):
            if B fails f_1a43aa0 and its flag bit 2 is not set: continue
            if the ordered-pair gate f_1cd5100(A, B) is false: continue

            if A.code_value == 0x5541 and B.code_value == 0x5255:
                for each region in A's +0x2f0 list (source adds 0x2dc to the +0x14 pointer):
                    if the region's +0x38 tagged value passes f_18f06d0:
                        apply f_1992f10 to the region

            if B.code_value == 0x5541 and A.code_value == 0x5255:
                for each region in B's +0x2f0 list:
                    if the region's +0x38 tagged value passes f_18f06d0:
                        apply f_1992f10 to the region

When the action id is 332032 (Annexion de certains territoires occupés):
    conflict = the conflict record selected by the acting nation
    apply f_1992f10 to each selected region

When f_1992f10(region) runs:
    company_index = integer part of the region's +0x38 tagged value
    for each company record in that slot:
        update the company fields
    if the region's +0x4 counter is positive:
        the region's +0x38 tagged value = the supplied value
```
> Evidence: f_19931a0 [fn 0x19931a0], corpus 2804356-2804432 (ordered pair loop 2804399-2804428; f_1cd5100 gate 2804396; A's +0x2f0 list via 0x2dc at 2804400; B's direct +0x2f0 load at 2804416; code values 0x5541/0x5255); f_1992f10 [fn 0x1992f10], corpus 2803962-2803963 and 2804025-2804028.

### 3.1 Action IDs: Data Rows and Code Literals

| Action id | Data row (French label) | Code literal | Handler / evidence |
| :--- | :--- | :--- | :--- |
| `[data xan.gt.tsv "332031"]` | Frontières modifiées | `[qty 0x510ff action-id]` | `[fn 0x19931a0 f_19931a0]` line 2804356 |
| `[data xan.gt.tsv "332032"]` | Annexion de certains territoires occupés | `[qty 0x51100 action-id]` | `[fn 0x19931a0 f_19931a0]` line 2804443 |
| `[data xan.gt.tsv "332040"]` | Pays annexé | `[qty 0x51108 action-id]` | `[fn 0x198cc20 exec_contrat]` line 2802237; `[fn 0x1d36a90 set_action_sp2]` line 3430880 |
| `[data xan.gt.tsv "332046"]` | Indépendance | `[qty 0x5110e action-id]` | `[fn 0x198cc20 exec_contrat]` line 2803456 |
| `[data xan.gt.tsv "102104"]` | Donner l'indépendance à une région | `[qty 0x18ed8 action-id]` | branch conditions, e.g. corpus line 2319450 |
| `[data xan.gt.tsv "506900"]` | Demander l'organisation d'un référendum d'autodétermination | none | only `[qty 0x7bc15 action-id]` (506901) occurs as a code literal, 35 times; 506900 itself has zero 32-bit immediate hits |

The `506900` row is not a code constant. Treating the id as a `*506900` base plus offset is not supported by the corpus; the code uses `[qty 0x7bc15 action-id]`, which is the next integer.

### 3.2 What the Annexation Handler Actually Does

`[fn 0x19931a0 f_19931a0]` handles `[qty 0x510ff action-id]` with an ordered nation-pair loop: outer nation A, inner nation B, gated by `[fn 0x1cd5100 f_1cd5100]` and by the code values `[qty 0x5541 code-value]` / `[qty 0x5255 code-value]` (corpus lines 2804399-2804428). When A carries `[qty 0x5541 code-value]` and B carries `[qty 0x5255 code-value]` it walks A's `[field 0x2f0 in NATI]` region list (A's load at 2804400 is from the +0x14 pointer plus 0x2dc); when B carries `[qty 0x5541 code-value]` and A carries `[qty 0x5255 code-value]` it walks B's `[field 0x2f0 in NATI]` region list (B's direct +0x2f0 load is at 2804416). It calls `[fn 0x1992f10 f_1992f10]` for regions whose `[field 0x38 in REGI]` passes `f_18f06d0`. The `[qty 0x51100 action-id]` branch reads a conflict record at `[field 0x428 in NATI]` (stride `[qty 0x6cb8 conflict-stride]`) and calls the same function (corpus lines 2804443-2804462). The `[qty 0x51108 action-id]` branch begins with a formatted notification, `" est intégré par "` (corpus lines 2802237-2250), then performs nine state updates: two `f_18e9ed0` calls, two `f_1a3a8d0` checks, `f_1d12bb0`, two `f_22c1dc0` flag ORs, `f_22c28f0` and `f_22c3110` (corpus lines 2802237-2475).

`f_1992f10` writes the `[field 0x38 in REGI]` tagged value at corpus lines 2804025-2804028 and operates on `[global 0x5d859dc8 i64]` records of stride `[qty 0xb88 stride]`. No write to `[field 0x28 in REGI]` or `[field 0x14 in REGI]` occurs in the cited ranges of either action handler. The list-unlink code `[fn 0x1c01830 f_1c01830]` exists and rewrites `[field 0x2f0 in NATI]` / `[field 0x0 in REGI]` (corpus lines 3265819-3265832), but its only call sites are corpus lines 3259833 (inside `[fn 0x1bf6220 f_1bf6220]`) and 3262957 (inside `[fn 0x1bfbe30 f_1bfbe30]`), not `exec_contrat` or `set_action_sp2`.

The old line `*(int16_t *)(region_ptr + 0x28) = annexing_nation_id;` is therefore removed. The `[field 0x28 in REGI]` value is a 16-byte tagged value, and the observed action-handler write is to `[field 0x38 in REGI]`, not `[field 0x28 in REGI]`.

### 3.3 Secession

The secession rows exist: `[data xan.gt.tsv "102104"]`, `[data xan.gt.tsv "506900"]`, `[data xan.gt.tsv "332046"]`. The code literals `[qty 0x18ed8 action-id]` and `[qty 0x5110e action-id]` appear in branch conditions (corpus line 2319450 and line 2803456). The list-unlink function `[fn 0x1c01830 f_1c01830]` exists, but no call from the secession paths is visible. A dormant nation slot becomes active when `[fn 0x1c30ef0 f_1c30ef0]` stores a region id into `[field 0x2f0 in NATI]` (corpus lines 3290386-3290390).

The corpus does not implement the debt partition or the garrison defection described in the old text. Searches for a loan record (`[qty 0xec stride]`) multiplied by a region-GDP ratio returned no such expression, and no unit-record write was found tied to a seceding region. Those claims are removed.

### 3.4 The Two R1-Flagged Lines

`POP_AQTE` and `POP_PETE` are not binary constants. Both tokens have zero code-immediate hits. They are full data keys in `data/gt/xim.gt.tsv`:

- `[data xim.gt.tsv "POP_AQTE"]` — row `POP_AQTE | Acquisition territoriale | Acquisition of Territory`, line 21637.
- `[data xim.gt.tsv "POP_PETE"]` — row `POP_PETE | Perte territoriale | Loss of Territory`, line 21639.

The old AQTE/PETE binary-constant parentheticals are deleted. Both tokens have zero code-immediate hits; the surviving citations are the two `xim.gt.tsv` keys above. Volume 4 (accepted) already established the same classification; this volume is consistent with it.

### What the code does not establish

- A pointer write `[field 0x28 in REGI] = annexing nation id`.
- An unlink/relink call inside the `332031` / `332032` / `332040` handlers.
- GDP or asset transfer between the defeated and annexing nations.
- A `POP_AQTE` approval surge of +10% to +25%, or a `POP_PETE` backlash of -20% to -45%.
- Cultural/religious integration friction against `1DIC` / `LCUL`, or guerrilla spawn.
- A debt partition by GDP ratio, a credit-rating downgrade to `D`, or garrison defection.
- A code constant `506900`; only the data row exists.

---

## 4. Infrastructure Systems & Public Works

The corpus contains four transport pools with verified strides and loader counts, plus data-table rows for power plants and military bases. It does not contain a connectivity index, a friction multiplier, megawatt capacities, or base-capacity mechanics.

```mermaid
flowchart TD
    CHMN["CHMN: 64,000 records x 0x98"]
    CHNO["CHNO: 64,000 records x 0x260"]
    PONT["PONT: 3,000 records x 0x30"]
    TUNN["TUNN: 2,000 records x 0x30"]
    ACT["xan.gt.tsv rows 310000 / 310001 / 310004 / 310008 / 504809"]
    MIL["xan.gt.tsv rows 330096 / 330097 / 330098"]
    NET["transport network records"]
    GEN["energy action records"]
    DEF["base action records"]
    CHMN --> NET
    CHNO --> NET
    PONT --> NET
    TUNN --> NET
    ACT --> GEN
    MIL --> DEF
```

```pseudocode
Load the transport network:
    read 64,000 CHMN records, each 0x98 bytes
    read 64,000 CHNO records, each 0x260 bytes
    read 2,000 TUNN records, each 0x30 bytes, tagged "TUNN"
    read 3,000 PONT records, each 0x30 bytes, tagged "PONT"

For each PONT or TUNN record:
    read its tag
    read a 0x30 size word
    read the 0x30-byte payload
```
> Evidence: Cdatabase__es_base [fn 0x143acd0], corpus 2074177 and 2074187 (CHMN/CHNO counts) and 2074904-2074941 (TUNN/PONT loops); f_1394770 [fn 0x1394770] line 1988353 (CHMN stride 0x98); f_1394930 [fn 0x1394930] line 1988428 (CHNO stride 0x260); f_1499950 [fn 0x1499950] lines 2128493-2128497 (PONT tag/size); f_14a8a60 [fn 0x14a8a60] lines 2137169-2137173 (TUNN tag/size).

### 4.1 Transportation Pools

| Pool | Stride | Count and loader evidence | Record tag |
| :--- | :--- | :--- | :--- |
| CHMN | `[qty 0x98 stride]` | `[qty 64000 chmn-records]` at corpus line 2074177; `[fn 0x1394770 f_1394770]` advances by `[qty 0x98 stride]` (line 1988353) | none observed |
| CHNO | `[qty 0x260 stride]` | `[qty 64000 chno-records]` at corpus line 2074187; `[fn 0x1394930 f_1394930]` advances by `[qty 0x260 stride]` (line 1988428) | none observed |
| PONT | `[qty 0x30 record-size]` | `[qty 3000 bridge-records]` at corpus lines 2074938-2074941 | `[tag 0x504f4e54 "PONT"]` in `[fn 0x1499950 f_1499950]` line 2128495 |
| TUNN | `[qty 0x30 record-size]` | `[qty 2000 tunnel-records]` at corpus lines 2074904-2074907 | `[tag 0x54554e4e "TUNN"]` in `[fn 0x14a8a60 f_14a8a60]` line 2137171 |

PONT and TUNN are serialized as a tag, a size word `[qty 0x30 record-size]` and a `[qty 0x30 record-size]`-byte payload (corpus lines 2128493-2128497 and 2137169-2137173). The corpus does not expose lane capacity, electrification, terrain resistance or a pathfinding graph over these records.

### 4.2 Energy Action Rows

The old table assigned coal, hydro and solar to `300110`, `300112` and `300118`. Those rows are natural-disaster actions, not power plants:

- `[data xan.gt.tsv "300110"]` — `Tremblement de terre` / earthquake.
- `[data xan.gt.tsv "300112"]` — `Éruptions volcaniques` / volcanic eruptions.
- `[data xan.gt.tsv "300118"]` — `Pollution`.

The plant rows that do exist are:

| Action row | Label | Energy / environment formula | Build days | Cost cell |
| :--- | :--- | :--- | :--- | :--- |
| `[data xan.gt.tsv "310000"]` | Energie fossile | `ENEE*5,2;ENER*5,2;ENVI*-1000` | 1095 | `MISE=274000000` |
| `[data xan.gt.tsv "310001"]` | Nucléaire | `ENEE*8;ENER*8;ENVI*-10` | 2555 | `MISE=10000000000` |
| `[data xan.gt.tsv "310004"]` | Eolien Terrestre | `ENEE*40;ENER*40;ENVI*80` | 270 | `MISE=822000000` |
| `[data xan.gt.tsv "310008"]` | Solaire | `ENEE*40;ENER*120;ENVI*200` | 244 | `MISE=676500000` |
| `[data xan.gt.tsv "504809"]` | Centrale nucléaire | reaction `VICN`, parameter 256 | empty | empty |

The `504809` row is a separate action row with no cost or build-duration cell. It is not a code immediate: its 32-bit decimal value has zero hits in the corpus.

Two event themes are real tags as well as data keys:

- `[tag 0x504e4545 "PNEE"]` — `[fn 0x22e4ad0 taux_critere]` line 4374801 returns a factor of `1.2` or `6.0` in the `PNEE` branch; `[fn 0xa45270 f_a45270]` calls `taux_critere` with that tag at line 356626.
- `[tag 0x4143434e "ACCN"]` — passed to `f_21c7410` at corpus line 3275564 in `[fn 0x1c12160 f_1c12160]`.

The matching data keys are `[data xim.gt.tsv "POP_PNEE"]` (line 21605) and `[data xim.gt.tsv "POP_ACCN"]` (line 21643). They name event themes; the corpus does not contain a generation-minus-demand balance or a reactor-failure probability.

### 4.3 Military Bases

The three base rows match Volume 3 and are action ids as well as data rows:

- `[data xan.gt.tsv "330096"]` — `Bases militaires`; code literal `[qty 0x50970 action-id]` in `[fn 0x2021e10 Cmenu_droit__do_action]` (corpus line 5342038) and `[fn 0x203bc10 f_203bc10]` (line 5346849).
- `[data xan.gt.tsv "330097"]` — `Bases navales`; code literal `[qty 0x50971 action-id]` (lines 5342043 and 5346962).
- `[data xan.gt.tsv "330098"]` — `Bases aériennes`; code literal `[qty 0x50972 action-id]` (lines 5342048 and 5347075).

Those branches toggle bits in `[global 0x464de228 u64]` (XOR with 0x10000000000, 0x20000000000, and the next bit). They are menu/selection state, not base-capacity or construction mechanics. Volume 3 already records that the `xan.gt.tsv` rows have an empty construction-duration cell.

### What the code does not establish

- A regional connectivity index, a trade-cost multiplier, or `1.0 / (1.0 + 0.35 * C)`.
- Highway/rail construction cost as length times base cost times terrain multipliers 1.0/2.5/3.2; the action rows give a single `MISE` and duration.
- CHMN/CHNO record field semantics (length, lanes, electrification, terrain resistance).
- Megawatt capacities, CO2 coefficients, or the `DFAC` 38/39/42 fuel mapping.
- A national grid balance, 20%-50% manufacturing shutdown, or rolling-blackout arithmetic.
- A nuclear meltdown check, reactor aging, or health/agriculture liability formula.
- Base garrison capacity 5,000-50,000, shipyard construction, SAM/radar umbrellas, or base upkeep.
- Any runtime count for `CBAT`/`CRPP`; only their strides are verified.

---

*Evidence base: `recomp/full_fns.clean.c` (D-001) and the shipped `data/gt/xan.gt.tsv` / `xim.gt.tsv` rows named above. Fields are cited by displacement (D-010); strides and counts by quantity (D-002); data keys by full key and file (D-008/D-012).*
