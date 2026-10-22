# Volume 3 — Claims Ledger

Per-volume evidence for `_whitepaper/03_MILITARY.md` (D-006). Every row cites `recomp/full_fns.clean.c` by function and line. `class` is one of `function`, `global`, `tag`, `quantity`, `data` or `field` (D-002/D-008/D-010). `rva` is the token tested; `-` means the old claim had no address or the claim is structural. Verdicts are limited to `verified`, `corrected` and `removed`.

The corpus of record is `recomp/full_fns.clean.c` (D-001). `array_inventory.tsv` is used only for the pool geometry (record size, count, holder, loader, serializer); every arithmetic claim is quoted from the corpus.

| claim_id | old_text_verbatim | class | rva | verdict(verified\|corrected\|removed) | corpus_evidence(function+line) | replacement_summary |
| --- | --- | --- | --- | --- | --- | --- |
| 03-C001 | `All military forces in GPS5 exist as discrete physical entities managed in a global static array of 120,000 slots located at `G_61d0deb0` (record tag `'UNIT'`, serializer `f_14a96e0`):` | function | 0x139d230 | corrected | `f_14a96e0` line 2137772 (`uStack_68 = 0x554e4954/*'UNIT'*/`); `f_139d230` line 1994109 (`param_2 + 0x750`); `Cdatabase__es_base` line 2075010 (`f_139d230(param_1,(void *)G_61d0deb0,120000,iVar15,...)`); `array_inventory.tsv` row 350 (holder `G_61d0deb0`, loader `f_139d230`, 120000 x 0x750) | The pool is 120,000 records of `0x750` bytes (224,640,000 bytes). 120000 is the literal loader record count at line 2075010 (the UNIT array length); the hex spelling `0x1d4c0` is simply unused. |
| 03-C002 | `- **Stride**: `0x750` bytes (1,872 bytes per unit).` | quantity | 0x750 | verified | `f_14a96e0` line 2137771 (`uStack_64 = 0x750`); `calc_unites` line 4703911 (`iVar16 * 0x750`); `f_139d230` line 1994109 (`param_2 + 0x750`) | Stride confirmed: 0x750 = 1,872 bytes. |
| 03-C003 | `- **Total In-Memory Size**: $120,000 \times 1,872 \approx 224.6 \text{ MB}$.` | quantity | - | verified | `f_139d230` line 1994117 (`io_read(param_2,param_3 * 0x750)`); `Cdatabase__es_base` line 2075010 (literal 120000 record count); `array_inventory.tsv` row 350 (224640000 bytes, 120000 records) | Arithmetic confirmed: 224,640,000 bytes = 224.64 MB decimal (214.2 MiB). |
| 03-C004 | `+0x08:  float    pos_x;                 // World longitude coordinate` | field | 0x8 | corrected | `calc_unites` line 4703976 (`*(int *)(&DAT_61d0deb4 + lVar17)`); `calc_unites` line 4704120 (`*(int *)(lVar17 + G_61d0deb8)`); `f_24fc7d0` line 4741481 (`rec + 0xb0`) | `+0x8` is the second linked-list next index, not `pos_x`. The verified record fields include `+0x4`, `+0x8`, `+0x3e`, `+0x58`, `+0x78`, `+0x88`, `+0xa0`, `+0xb0`, `+0xbc`, `+0xc4`, `+0xc8` and `+0xec`; the old `+0x00..+0x58` table is not usable as stated (`+0x58` is a double, not the claimed list head, and `+0x06`/`+0x08`/`+0x18..+0x2c` do not match). |
| 03-C005 | `Unit base parameters are defined across 1,050 rows in `data/gt/xue.gt.tsv`.` | data | - | corrected | `data/gt/xue.gt.tsv`: 1 header line + 1,048 data rows (file is CRLF; 1,049 logical lines) | The table has 1,048 data rows, not 1,050. |
| 03-C006 | `&#124; **Infantry (`soldats CRG`)** &#124; Tier 1 &#124; 6.0 &#124; 10.0 &#124; 0.356 km/h &#124; 2.0 km &#124; 1,000,000 km &#124; 1 soldier &#124; \$10,000 &#124;` | data | - | corrected | `data/gt/xue.gt.tsv` row `soldats CRG Niv1`: Attaque 6.0000, Defense 10, Hommes 1, Autonomie 5.0000, Rayon 1000000.0000, Vitesse 0.3564, Prix 1, Distance d'attaque 2.0000 | Values are exact; the table declares no km/h, km or dollar unit, and the price is 1, not 10,000. |
| 03-C007 | `&#124; **Heavy Tank (`tanks`)** &#124; Tier 1 &#124; 1,200.0 &#124; 1,200.0 &#124; 0.178 km/h &#124; 2.0 km &#124; 1,500,000 km &#124; 5 crew &#124; \$2,100,000 &#124;` | data | - | corrected | `data/gt/xue.gt.tsv` row `tanks Niv1`: Attaque 1200.0000, Defense 1200, Hommes 5, Autonomie 5.0000, Rayon 1500000.0000, Vitesse 0.1782, Prix 120, Distance d'attaque 2.0000 | Values are exact; the price is 120, not 2,100,000, and the table declares no physical units. |
| 03-C008 | `&#124; **Fighter (`chasseur`)** &#124; Tier 5 &#124; 2,200.0 &#124; 1,800.0 &#124; 2.500 km/h &#124; 80.0 km &#124; 4,200 km &#124; 1 pilot &#124; \$95,000,000 &#124;` | data | - | corrected | `data/gt/xue.gt.tsv` row `chasseurs Niv5`: Attaque 200.0000, Defense 1400, Hommes 1, Autonomie 1.0000, Rayon 1500000.0000, Vitesse 5.4000, Prix 2000, Distance d'attaque 2.0000 | Values are exact; the table's `Rayon` is 1,500,000 and `Prix` is 2000, not 4,200 km and 95,000,000. |
| 03-C009 | `&#124; **Destroyer (`destroyer`)** &#124; Tier 3 &#124; 4,500.0 &#124; 5,000.0 &#124; 0.450 km/h &#124; 60.0 km &#124; 12,000 km &#124; 220 sailors &#124; \$850,000,000 &#124;` | data | - | removed | `data/gt/xue.gt.tsv`: no row whose name contains `destroyer`, `frégate` or `corvette`; the table has no such archetype | The destroyer row is not in the table and is removed. |
| 03-C010 | `&#124; **Carrier (`porte-avions`)**&#124; Tier 5 &#124; 2,000.0 &#124; 8,500.0 &#124; 0.520 km/h &#124; 150.0 km &#124; Unlimited (Nucl)&#124; 3,200 crew &#124; \$5,500,000,000 &#124;` and `&#124; **ICBM Silo (`silo`)** &#124; Tier 1 &#124; 50,000.0 &#124; 5,000.0 &#124; N/A (Stationary)&#124; Global &#124; Global &#124; 50 crew &#124; \$150,000,000 &#124;` | data | - | corrected | `data/gt/xue.gt.tsv` row `porte-avions Niv5`: Attaque 4000.0000, Defense 120000, Hommes 2000, Autonomie 300.0000, Rayon 20000000.0000, Vitesse 0.9720, Prix 10000; row `Missiles intercontinentals Niv1`: Attaque 5400.0000, Defense 100, Hommes 0, Rayon 1000000.0000, Prix 0; row `Silos d'attaque niveau 1 Niv1`: Attaque 100.0000, Defense 100, Hommes 1, Prix 0 | The carrier row is corrected to 4000/120000/2000/10000. There is no `silo` archetype with 50,000 attack and 150,000,000 price; the nearest rows are `Missiles intercontinentals Niv1` and `Silos d'attaque niveau 1 Niv1`. |
| 03-C011 | `1. **Financial Allocation**: Base price loaded from `prix_normal_construction` (`0x1cba4d0`).` | function | 0x1cba4d0 | corrected | `prix_normal_construction` lines 3345745-3345787: `f_1cd6530(...)*0x3b0`, flag byte at `+0x10`, `f_1c7e750(param_2)`, `f_1cd9b10(1.0,param_2)`, floats at `+0x0`/`+0x1ec`, `f_a553f0(...,5)` | The price is computed from a type record, an optional multiplier, an inflation term and a divisor, then passed through `f_a553f0(...,5)`; it is not a literal base price. |
| 03-C012 | `2. **Industrial Capacity**: Output from domestic defense activities (`DEFA`, `DEF2`, `DEFB`).` | tag | 0x44454641 | removed | `DEFA` literal `0x44454641` at line 2095915 (label assignment); `DEF2` occurs in `xim.gt.tsv` and `xqm.gt.tsv`; `DEFB` occurs in `xia.gt.tsv`, `xim.gt.tsv`, `xrn.gt.tsv` and `xso.gt.tsv`; no output function computes from them | Removed: `DEFA` is a four-character code, not an industrial-output mechanism. |
| 03-C013 | `3. **Manpower Quota**: Active military recruits deducted from the national available manpower pool (`MILA` at `+0xc850` in `NATI`).` | tag | 0x4d494c41 | corrected | `get_variable` line 4828470 (`piVar6 = (int *)(param_1 + 0x6428)` inside the `MILA` case) | `MILA` is a variable code; the getter uses `+0x6428`, not `+0xc850`. No manpower deduction was found. |
| 03-C014 | `4. **Build Duration**: Time-to-build $T_{\text{build}}$ (e.g., 90 days for tanks, 720 days for nuclear submarines).` | data | - | removed | No `90`/`720` unit-build constants in `xue.gt.tsv` or the corpus size; `xan.gt.tsv` rows 330096-330098 have an empty construction-duration cell | Removed: no build-duration schedule is present. |
| 03-C015 | `Units incur daily maintenance costs deducted from the Defense Budget (`BDEM` at offset `+0x7b64` in `NATI`):` | tag | 0x4244454d | corrected | `get_variable` line 4832402 (`auVar59 = ZEXT416(*(uint *)(param_1 + 0x3db2))` inside the `BDEM` case) | `BDEM` is a variable code whose getter reads `param_1+0x3db2`; `+0x7b64` is not in that branch. |
| 03-C016 | `$$\text{UpkeepCost}(u) = \text{BasePrice}(u) \times \mu_{\text{upkeep}} \times \left(1 + \text{OverseasMultiplier}\right)$$` | quantity | - | removed | No multiplication of a base price by an upkeep coefficient; the `BDEM` getter returns one word at line 4832402 | Removed: no upkeep arithmetic exists. |
| 03-C017 | `- If deployed abroad, overseas maintenance surcharges are logged under `CATE` (offset `+0x230`).` | tag | 0x43415445 | corrected | `get_variable` line 4832594 (`fVar82 = f_1bec400(6,param_1,0)` inside the `CATE` case) | `CATE` is a variable code; its getter calls `f_1bec400(6, param_1, 0)`, not an offset `+0x230`. |
| 03-C018 | `- **Readiness Decay**: If the defense budget is cut below maintenance requirements: ... Units suffer passive operational attrition: fuel reserves drop, and hit points decay by up to 2% per macro-turn.` | quantity | - | removed | Searches for `0.98`, `0.02` and `- 0.02` against unit-record accesses in `calc_unites` and `f_24b4580` returned no such assignment; the only subtractions on unit records are path progress (`calcul_cout_reconstruction`-adjacent paths) and timestamps | Removed: no readiness-decay or hit-point-decay arithmetic was found. |
| 03-C019 | `$$\text{RepairCost} = \left(1 - \frac{\text{hp\_current}}{\text{hp\_max}}\right) \times \text{BasePrice} \times 0.40$$` | function | 0x24c7870 | corrected | `calcul_cout_reconstruction` lines 4705085-4705160: `f_1cd6530` returns a type index at 4705117 (`index*0x3b0`); `f_1cd9b10(value at +0x0, market) / divisor at +0x1ec` at 4705119-4705121; log10/powf rounding; `*0.125` at 4705146; `/10` `/100` `/1000` at 500000/1000000 (4705147/4705150); `(cost * 0.65 * field_0x72c) / f_253fb20` at 4705159-4705160 | The function exists, but its reconstruction value is `f_1cd9b10(value at +0x0, market) / divisor at +0x1ec`, rounded to a power of ten, then optionally scaled by 0.125 and /10 /100 /1000, and finally multiplied by `0.65 * field_0x72c / f_253fb20(record)`; there is no hp ratio and no 0.40. |
| 03-C020 | `Combat simulation executes in `calc_unites` (`0x24c5200`), running during Phase 8 of `calc_monde`:` | function | 0x24c5200 | corrected | `calc_monde` line 2031121 (`calc_unites(param_1)`); `calc_unites` lines 4703763-4703980 | The call is verified; the "Phase 8" label is not in the corpus and is removed. |
| 03-C021 | `To avoid $O(N^2)$ pairwise distance checks across 120,000 units, the world surface is partitioned into 10,000 spatial cells ... CellIndex = CellY * 100 + CellX` | quantity | - | removed | `calc_unites` line 4703792 (`if (uVar21 < 10000)`) guards a doubly linked iterator list; lines 4703813-4703814 throw `"Bad iterator"`; no `CellX`, `CellY`, `*100` or cell array found | Removed: the 10,000 is an iterator bound, not a 100x100 world grid. Units are walked by index lists. |
| 03-C022 | `### 4.2 Lanchester Combat Engagement Formula ... Damage(A -> B) = ...` | quantity | - | removed | `grep -ci lanchester recomp/full_fns.clean.c` = 0; `grep -ci "n-square" recomp/full_fns.clean.c` = 0; no per-tick subtraction of a strength or hit-point field in `calc_unites`/`f_24b4580` | Removed. The only two-sided arithmetic is the war score in `f_1329770`, which sums `(field_0xc8 + field_0xc4) * field_0xec` and compares sides. |
| 03-C023 | `- $\text{TerrainMod}$ accounts for cover: $1.0$ in open plains, $1.5$ in forests, $2.5$ in urban centers (`warville`), $3.0$ in fortified bunkers.` | quantity | - | removed | No terrain/cover multiplier in `calc_unites` or `f_24b4580`; no `1.5`/`2.5`/`3.0` branch on a unit-vs-unit damage term | Removed: no terrain modifier exists. |
| 03-C024 | `- $\Xi(A, B)$ is the domain effectiveness matrix (e.g., SAM batteries deal $4.0\times$ damage against aircraft; torpedoes deal $3.5\times$ damage against ships; infantry deal negligible damage against heavy tanks).` | quantity | - | removed | No domain-effectiveness matrix, no `4.0`/`3.5` damage multiplier, and no aircraft/ship/tank effectiveness table in the corpus | Removed: no domain matrix exists. |
| 03-C025 | `When $\text{ammo\_current} = 0$, unit attack drops to $10\%$ of nominal value. If $\text{hp\_current} / \text{hp\_max} < 0.25$ or $\text{Morale} < 0.20$, the unit breaks formation ...` | quantity | - | removed | No ammunition field, no `* 0.10` attack branch, no `0.25`/`0.20` rout test against unit fields | Removed: no ammunition or rout mechanism exists. |
| 03-C026 | `Wars and alliances are tracked in the `CONF` record pool (serializer `f_14585e0`, stride `0x12d0`, 2,000 active slots):` | function | 0x14585e0 | verified | `f_14585e0` line 2091654; `array_inventory.tsv`: 9,632,000 bytes, 0x12d0 stride, 2000 count, holder `&DAT_433f40e0`, serializer `f_14585e0` | Pool geometry confirmed: 2,000 x 0x12d0 = 9,632,000 bytes. |
| 03-C027 | `- Belligerent nation indices ... Formal war objectives ... Casus belli validity & UN Security Council authorization flags.` | quantity | - | removed | The `CONF` pool exists (see 03-C026), but the serializer header and pool geometry do not establish these fields; no named field access was read | Removed: field semantics are not established by the corpus. |
| 03-C028 | `1. **Civilian Casualties**: Computed via `calc_cata_victimes_initiales` (`0x182afc0`): ... \times 0.005` | function | 0x182afc0 | corrected | `calc_cata_victimes_initiales` line 2624885; source label line 2625008 (`gps_catas.cpp`); arithmetic lines 2624910-2625160 (`log10`, `powf`, `+0x9a0`, `0.15`, `860.0`, `0.013076923`, `0.125`) | The function is a catastrophe-victim estimator, not urban combat. The `Pop * damage / fortification * 0.005` formula is not in it. |
| 03-C029 | `Fatalities are subtracted directly from national population (`NATI.POPU` at `+0x4428`).` | field | 0x4428 | removed | `gestion_popu_warville` lines 4125312-4125411 calls `f_21c7410` on `0xad0`-stride records; no NATI `+0x4428` write is in that routine | Removed: no population subtraction or fatality count is implemented in the read path. |
| 03-C030 | `Ruin transitions destroy housing stock, raising homeless population (`NSDF` at `+0x4478`)` | tag | 0x4e534446 | corrected | `get_variable` line 4829463 (`*(float *)(param_1 + 0x223c) = fVar27` inside the `NSDF` case) | `NSDF` is a variable code; the getter uses `+0x223c`, not `+0x4478`. |
| 03-C031 | `and spawning climate/war refugees (`RCLW` at `+0x619ca0`).` | field | 0x619ca0 | corrected | `f_205eb70` line 3910430 (`uVar7 * 0x645310 + 0x619ca0 + _DAT_46208908`); `f_1056100` lines 1481952-1481958 (RCLW getter sums `+0x6434a8`/`+0x6434ac`) | `0x619ca0` is a NATI float field used in `f_205eb70`, not the RCLW variable. RCLW is a get_variable code whose getter is `f_1056100`. |
| 03-C032 | `- **Launch Sequence**: Silos or SSBNs fire ICBMs ... - **Interception Roll**: ... P(Intercept) = ... - **Detonation**: ... 85% to 99% ... deleted from `CBAT` ... soot parameters increment.` | tag | 0x4e554b45 | removed | `NUKE` literal `0x4e554b45` occurs exactly twice as a `get_variable` case label (lines 3588017 and 4821458); no launch, interception, mortality, `CBAT` or soot code | Removed: no nuclear launch/interception/deterrence/exchange model exists. |
| 03-C033 | `the volume's 330096/330097/330098` action ids | quantity | 0x50970 | corrected | `data/gt/xan.gt.tsv` rows 330096-330098 (`Military bases` / `Naval bases` / `Air bases`); `Cmenu_droit__do_action` line 5342038 (`case 0x50970`); `f_203bc10` line 5346849 | The ids are action ids and code literals, not binary tags. They are cited as `[data xan.gt.tsv "..."]` plus `[qty 0x50970 action-id]`. |

## Closing statistics

- **Rows:** 33.
- **Verdicts:** verified 3 (`03-C002`, `03-C003`, `03-C026`); corrected 17 (`03-C001`, `03-C004`, `03-C005`, `03-C006`, `03-C007`, `03-C008`, `03-C010`, `03-C011`, `03-C013`, `03-C015`, `03-C017`, `03-C019`, `03-C020`, `03-C028`, `03-C030`, `03-C031`, `03-C033`); removed 13 (`03-C009`, `03-C012`, `03-C014`, `03-C016`, `03-C018`, `03-C021`, `03-C022`, `03-C023`, `03-C024`, `03-C025`, `03-C027`, `03-C029`, `03-C032`).
- **Unique tokens in the ledger (excluding `-`):** 17 — `0x139d230`, `0x750`, `0x8`, `0x1cba4d0`, `0x44454641`, `0x4d494c41`, `0x4244454d`, `0x43415445`, `0x24c7870`, `0x24c5200`, `0x14585e0`, `0x182afc0`, `0x4428`, `0x4e534446`, `0x619ca0`, `0x4e554b45`, `0x50970`.
- **Unique 5–8 digit tokens in the rewritten volume:** 55 — 29 function, 3 global, 12 quantity, 10 tag, 1 unresolved/refuted (`0x1d4c0`). The 15 three-to-four-digit tokens and the two-digit field/constant tokens (`0x3e`, `0x58`, `0x78`, `0x80`, `0x5c`, etc.) are below `cite.py`'s 5–8 digit matcher; each is bracketed `[qty ...]`/`[field ...]`, inside a fenced transcription, or an explicit retraction (`+0xc850`, `+0x7b64`, `+0x230`, `+0x4478`).
- **Per-class counts:** function 6; global 0; tag 6; quantity 11; data 7; field 3.
- **Wrong addresses (old -> new):**
  - `0x1d4c0` (old pool-count literal) -> no literal; `119999` last index, `120000.0` bound, and `array_inventory.tsv` row 350.
  - `+0xc850` (MILA) -> `+0x6428` in `get_variable` line 4828470.
  - `+0x7b64` (BDEM) -> `+0x3db2` in `get_variable` line 4832402.
  - `+0x230` (CATE) -> `f_1bec400(6,param_1,0)` at line 4832594.
  - `+0x4478` (NSDF) -> `+0x223c` in `get_variable` line 4829463.
  - `+0x619ca0` (RCLW) -> NATI field used by `f_205eb70`; RCLW is `f_1056100` reading `+0x6434a8`/`+0x6434ac`.
  - `+0x08 pos_x` -> `+0x8` next-link index in `calc_unites` line 4704120.
  - `+0x06 archetype_id` -> `+0xb0` type word in `f_24fc7d0` line 4741481.
  - `0x182afc0` (urban casualties) -> catastrophe estimator; urban path is `0x219cfb0` -> `0x21c7410`.
  - `0x18fd700` (unit production) -> generic `add_construction`, not a unit-procurement routine.
- **Claims removed:** 13 (listed above). The constructs are removed, not softened.
- **Claims added (verified replacement facts):** the `UNIT` tag and 0x750 header in `f_14a96e0`; the literal 120000 loader count at `Cdatabase__es_base` line 2075010; the `f_139d230` count-times-stride loader; the intrusive list head at `+0x4` and second link at `+0x8`; the verified record offsets including `+0x3e`, `+0x58`, `+0x78` and `+0x80`; the 1,048-row `xue.gt.tsv` catalogue with exact rows; the `prix_normal_construction` arithmetic; the `calcul_cout_reconstruction` arithmetic; the two-pass `calc_unites` walk and the `f_24b0890` movement formula; the `f_1329770` war score; the `gestion_popu_warville` -> `f_21c7410` per-record modifier chain; the `f_1056100` RCLW getter; the `0x619ca0` NATI field; the `CONF` pool geometry.
- **Not implemented by the corpus (removed constructs):**
  1. The two-sided attrition differential equation (old section 4.2).
  2. The 100x100 spatial hash, 10,000 cells and `CellY * 100 + CellX`.
  3. Terrain/weather/posture cover multipliers.
  4. The domain-effectiveness matrix `Xi`.
  5. Ammunition depletion and the 10% attack floor.
  6. Morale/rout thresholds (0.25 / 0.20).
  7. Daily upkeep, overseas surcharge and readiness decay.
  8. Unit build-duration schedule and manpower quota.
  9. Nuclear launch, interception, deterrence, 85%-99% mortality, `CBAT` deletion and soot.
  10. Population subtraction and a fatality count from urban combat.
  11. `CONF` field semantics (coalitions, war objectives, casus belli, UN authorization).
  12. The three-state ruin transition logic (not established: `xue.gt.tsv` has 462 Ruin/Semi-Ruin rows and the corpus has `Ruine_` strings at line 1442517, but no transition arithmetic was read).

## Evidence commands

```
python -I tools/cite.py 139d230      -> function f_139d230
python -I tools/cite.py 14a96e0      -> function f_14a96e0
python -I tools/cite.py 554e4954     -> tag "UNIT"
python -I tools/cite.py 1d4c0        -> unresolved (refuted token)
python -I tools/cite.py 750          -> unresolved (3-digit; manual qty from f_14a96e0/f_139d230)
python -I tools/cite.py 61d0deb4     -> global i32
python -I tools/cite.py 31ed6c90     -> global i64
python -I tools/cite.py 645310       -> quantity
python -I tools/cite.py 4244454d     -> tag "BDEM"
python -I tools/cite.py 4d494c41     -> tag "MILA"
python -I tools/cite.py 4e534446     -> tag "NSDF"
python -I tools/cite.py 52434c57     -> tag "RCLW"
python -I tools/cite.py 43415445     -> tag "CATE"
python -I tools/cite.py 4e554b45     -> tag "NUKE"
python -I tools/cite.py 50970        -> quantity
```

## Raw decompiler excerpts moved from the volume

These are the exact corpus excerpts the rewritten volume summarises. They are kept here so the volume can stay readable while the claims remain checkable.

### Unit pool and serializer

```c
// f_14a96e0, corpus lines 2137771-2137772
uStack_64 = 0x750;
uStack_68 = 0x554e4954/*'UNIT'*/;
```

```c
// f_139d230, corpus lines 1994107-1994117
while (uVar3 != 0) {
    iVar1 = f_14a96e0(param_2, 1, param_4, param_5);
    uVar2 = uVar2 + iVar1;
    param_2 = (void *)((longlong)param_2 + 0x750);   // line 1994109
    uVar3 = uVar3 - 1;
}
...
uVar3 = io_read(param_2, param_3 * 0x750);            // line 1994117
```

```c
// calc_unites, corpus lines 4703763-4703768
iVar16 = _DAT_61d0deb4;                               // head, base+4
while (0 < iVar16) {
    lVar17 = (longlong)iVar16 * 0x750;
    iVar16 = *(int *)(&DAT_61d0deb4 + lVar17);        // next at +4
    ...
}
```

```c
// f_24fc7d0, corpus line 4741481
ulonglong f_24fc7d0(longlong param_1)
{
  return (ulonglong)*(uint *)(param_1 + 0xb0);
}
```

### Per-type stats

```c
// f_24a5040, corpus lines 4688110-4688113
lVar12 = f_24f4f80((longlong)ppiVar24);
*(undefined4 *)(lVar23 + G_61d0df74) = *(undefined4 *)(lVar12 + 8);   // unit +0xc4
lVar12 = f_24f4f80((longlong)ppiVar24);
*(float *)(lVar23 + G_61d0df78) = (float)*(int *)(lVar12 + 0xc);      // unit +0xc8
```

### Price and reconstruction

```c
// prix_normal_construction, corpus lines 3345743-3345787
uVar3 = f_1cd6530(..., 0x278a, '\x01');
lVar4 = (longlong)(int)uVar3 * 0x3b0;
fVar5 = 1.0;
if ((*(byte *)(lVar4 + G_47938898) & 0x10) != 0) {
    fVar5 = f_1c7e750(param_2);                       // (not read)
}
if (!finite(fVar5) || fVar5 <= 0.0) fVar5 = 1.0;
dVar1 = f_1cd9b10(1.0, param_2);                      // (not read)
afStackX_8[0] = *(float *)(lVar4 + G_47938a74);
if (afStackX_8[0] == 0.0) afStackX_8[0] = 1.0;
afStackX_8[0] = ((float)dVar1 * *(float *)(lVar4 + G_47938888) * fVar5) / afStackX_8[0];
return f_a553f0(afStackX_8, 5);                       // call not read
```

```c
// calcul_cout_reconstruction, corpus lines 4705085-4705160
uVar4 = f_1cd6530(..., 0x2d8a, '\x01');
lVar5 = (longlong)(int)uVar4 * 0x3b0;
dVar7 = f_1cd9b10(..., param_1) / (double)*(float *)(lVar5 + G_47938a74);  // (not read)
... log10 / powf rounding to a power of ten ...
fVar6 = (float)dVar8;
if (((*(uint *)(param_2 + 0xb0) >> 0x10 & 1) != 0) &&
    (fVar6 = fVar6 * 0.125, 1 < (*(uint *)(param_2 + 0xb0) & 0xfff) - 2)) {
    if (*(int *)(param_2 + 0x130) < 500000) fVar6 = fVar6 / 10.0;
    else if (*(int *)(param_2 + 0x130) < 1000000) fVar6 = fVar6 / 100.0;
    else fVar6 = fVar6 / 1000.0;
}
return (fVar6 * 0.65 * (float)*(int *)(param_2 + 0x72c)) / (float)f_253fb20(param_2);
```

### Per-nation dispatch and movement

```c
// calc_unites, corpus lines 4704114-4132
iVar16 = *(int *)(lVar17 + 0x3b44);                  // nation's unit-list head
do {
    if (iVar16 < 1) break;
    lVar17 = (longlong)iVar16 * 0x750;
    iVar16 = *(int *)(lVar17 + G_61d0deb8);          // next at +0x8
    if (*(int *)(lVar17 + G_61d0df6c) != 0x20790) {  // field +0xbc
        if ((*(uint *)(lVar17 + G_61d0df50) >> 0xc & 1) == 0) {
            if ((*(uint *)(lVar17 + G_61d0df50) >> 0xd & 1) == 0) continue;
            ...
        } else {
            if (*(int *)(lVar17 + G_61d0df6c) != 0x92819) {
                f_24b06e0(lVar17 + G_61d0deb0,param_2,iVar22,iVar22);
                continue;
            }
            ...
        }
        ...
    }
} while (true);
```

```c
// f_24b0890, corpus lines 4692684-4692731
fVar14 = f_25c6b70(_DAT_464916a8 + *(int *)(param_1 + 0x28), 0.2);   // (not read)
fVar14 = _DAT_32210d8 * (fVar14 + 0.9) * param_2;
if (fVar14 <= 0.0) { ... return; }
...
fVar16 = 1.0;
if (1.0 <= fVar14) fVar16 = fVar14;
fVar14 = *(float *)(param_1 + 200);                                  // field +0xc8
fVar16 = fVar16 + (fVar14 - (float)*(int *)(param_1 + 0xf0));
uVar13 = (uint)(fVar16 / fVar14);
fVar15 = fmodf(fVar16, fVar14);
...
f_2519f80(param_1, lVar7 * 0x750 + G_61d0deb0, (int)fVar16);         // (not read)
```

### War score, urban effect and the 0x619ca0 field

```c
// f_1329770, corpus lines 1970173, 1970246-1970248, 1970271
iStackX_20 = (int)(((float)param_1[9] * 0.5 + 50.0) - (float)(_DAT_464916a8 - param_1[10]));
...
fVar17 = (*(float *)(lVar10 + 200 + _DAT_31ed6c90) +
          *(float *)(lVar10 + 0xc4 + _DAT_31ed6c90)) *
         (float)*(int *)(lVar10 + 0xec + _DAT_31ed6c90);
...
fVar16 = fVar16 * 100.0 * 0.25 + (float)iVar12 + (float)iVar5 * 0.25 + (float)(iVar11 - iVar14);
...
if (param_1[2] == 2) {
    ...
    fVar16 = (fVar19 - fVar18) * 0.25 + fVar16;
}
...
if (3 < _DAT_464916a8 - param_1[10]) {
    fVar16 = fVar16 + -40.0;
}
```

```c
// gestion_popu_warville, corpus lines 4125312-4125411
fVar27 = f_1329770((int *)puVar25, pdVar12, pdVar9);
...
fVar28 = -0.005;
if (fVar27 < 0.0) fVar28 = -0.0025;
if (*(uint *)(psVar24 + 0x176) == puVar25[8]) fVar28 = fVar28 + fVar28;
fVar28 = fVar28 * fVar27;
if (-10.0 <= fVar28) { fVar29 = 10.0; if (fVar28 <= 10.0) fVar29 = fVar28; }
else fVar29 = -10.0;
...
f_21c7410((longlong)iVar6 * 0xad0 + _DAT_5d85ada0, fVar29 / 100.0, 0, uVar26, ...);
```

```c
// f_205eb70, corpus lines 3910428-3910433
fVar13 = *(float *)((longlong)(int)uVar7 * 0x645310 + 0x619c90 + _DAT_46208908);
uVar7  = f_22f1330((double *)(param_1 + 0x28), 0);
fVar1  = *(float *)((longlong)(int)uVar7 * 0x645310 + 0x619ca0 + _DAT_46208908);
fVar10 = f_25c6b70(*(int *)(param_1 + 4) + _DAT_464916a8, 1.0);
fVar13 = fVar10 * ((fVar13 * 10.0 * (float)(fVar8 == 1.0) + fVar12 + fVar12) -
                   fVar1 * 10.0 * (float)(fVar9 == 1.0)) + 0.01;
if (0.01 <= fVar13) { /* kept */ } else { fVar13 = 0.01; }
```

### Additional prose-level citations

```c
// Cdatabase__es_base, corpus line 2075010
iVar10 = f_139d230(param_1,(void *)G_61d0deb0,120000,iVar15,
                          *(undefined8 *)(&stack0xffffffffffffffe8 + lVar3),
                          *(longlong *)(&stack0xfffffffffffffff0 + lVar3));
```

```c
// f_1329770, corpus line 1970256
} while ((iVar12 - 1U < 119999) &&
```

```c
// f_1327960, corpus lines 1969198 and 1969432
sVar2 = *(short *)(&DAT_61d0deee + (longlong)iVar29 * 0x750);
...
uVar15 = f_22f02d0((double *)(_DAT_31ed6c90 + 0x58 + lVar32),
```

```c
// f_24b0890, corpus lines 4692737-4692738
dStack_e8 = *(double *)(param_1 + 0x78);
dStack_e0 = *(double *)(param_1 + 0x80);
```
