# Volume 8 — Claims Ledger

Per-volume evidence for `_whitepaper/08_REGIONS_CITIES_AND_INFRASTRUCTURE.md` (D-006). Every row cites `recomp/full_fns.clean.c` by function and line, or a shipped `data/gt/*.tsv` row. `class` is one of `function`, `global`, `tag`, `quantity`, `data`, `field` or `-` (D-002/D-008/D-010/D-011/D-012). `rva` is the token tested; `-` means the old claim had no address or the claim is structural. Verdicts are limited to `verified`, `corrected` and `removed`.

The corpus of record is `recomp/full_fns.clean.c` (D-001). `array_inventory.tsv` and `nation_fields.tsv` are used only as leads; every surviving arithmetic claim is quoted from the corpus or from a named `data/gt` row.

| claim_id | old_text_verbatim | class | rva | verdict(verified\|corrected\|removed) | corpus_evidence(function+line) | replacement_summary |
| --- | --- | --- | --- | --- | --- | --- |
| 08-C001 | "Geopolitical Simulator 5 organizes planetary space and governance into a strict three-tier administrative hierarchy: **Nation (`NATI`) $\longrightarrow$ Region (`REGI`) $\longrightarrow$ City (`VILL`)**." | function | 0x85d580 | verified | `f_85d580` lines 41991 and 42005 (NATI `+0x2f0` head, REGI `+0x0` next); `f_1c32e50` lines 3291882-3291906 (REGI allocation); `f_13a07b0` lines 1996505-1996515 (VILL allocation) | The three arrays and both links exist. The hierarchy is verified. |
| 08-C002 | "    subgraph Nation_Tier [Nation Level: NATI (Stride 0x645310, 300 instances)]" | quantity | 0x645310 | corrected | `f_1399d60` line 1991897 (`param_2 + 0x645310`); `Cdatabase__es_base` lines 2074045-2074049 (`uVar58 = 300` else `_DAT_322a3f4`); `f_1cbac20` line 3345979 (`_DAT_322a3f4 = 1`) | Stride verified. 300 is one loader path; the live count is `[global 0x322a3f4 i32]`. |
| 08-C003 | "    subgraph Region_Tier [Region Level: REGI (Stride 0x910, 1,309 instances)]" | quantity | 0x910 | corrected | `f_139bb40` line 1993157 (`param_2 + 0x910`); `Cdatabase__es_base` line 2074132 (count `_DAT_463d1b48`); `f_1c32e50` line 3291882 (`count * 0x910`); `f_1cbac20` line 3345982 (`_DAT_463d1b48 = 1`) | Stride verified. 1,309 is not a code literal; the live region counter `[global 0x463d1b48 i32]` supplies the count. |
| 08-C004 | "    subgraph City_Tier [City Level: VILL (Stride 0x9cf8, ~40,184 bytes)]" | quantity | 0x9cf8 | verified | `f_13a07b0` lines 1996505-1996515 (`malloc(param_3 * 0x9cf8)`, `memcpy(..., param_3 * 0x9cf8)`); `Cdatabase__es_base` line 2076846 | Stride confirmed: 0x9cf8 = 40,184 bytes. The count is read from the save stream. |
| 08-C005 | "    subgraph Physical_Tier [Micro-Physical Assets: CBAT (Stride 0x68, 620,000 instances)]" | quantity | 0x68 | corrected | `f_96afa0` lines 221199-221214 (`param_2 + 0x68`, `param_3 * 0x68`); `f_96a840` lines 220923-220938 for CRPP `0x1cc0` | Strides verified. 620,000 and 248,000 are not code constants; the loaders use a caller-supplied count. |
| 08-C006 | "        N_HEAD[+0x2f0: first_region_id]" | field | 0x2f0 | verified | `f_85d580` line 41991 (`*(short *)(param_1 * 0x645310 + 0x2f0 + NATI_BASE)`); `f_1c01830` lines 3265819-3265822 (head replace) | `[field 0x2f0 in NATI]` is the region-list head and the activation test used by `nation_is_active`. |
| 08-C007 | "        N_PIB[+0x30ce44: National GDP NPIB]" | field | 0x30ce44 | corrected | `f_23e9640` lines 4548000-4548009 (`fVar21 = *(float*)(param_1+0x30ce44)` then `(fVar21*0.7)/5000.0`); `f_1cb2720` line 3342000 | `[field 0x30ce44 in NATI]` is a float field. The corpus never labels it GDP or NPIB; the 13 literal hits are offset accesses, not an address. |
| 08-C008 | "        R_SOV[+0x28: sovereign_nation_id]" | field | 0x28 | corrected | `f_1c32e50` lines 3291889-3291890 (16-byte tagged value); `AVTerritoire__calc_prix_territoire` lines 38187-38189 (read as integer, `* 0x645310` to index NATI) | `[field 0x28 in REGI]` is a 16-byte tagged value whose integer interpretation indexes the nation array. It is not an `int16_t` sovereign id. |
| 08-C009 | "        R_OCC[+0x30: occupying_nation_id]" | field | 0x30 | corrected | `f_22f1330` line 4378813 (`switch(*(undefined4 *)(param_1 + 1))`, i.e. +8 bytes); `f_1c32e50` line 3291890 writes the high half | `[field 0x30 in REGI]` is the tag word of the `+0x28` tagged value. It is not a second nation id. |
| 08-C010 | "        R_JURE[+0x38: dejure_nation_id]" | field | 0x38 | corrected | `f_1992f10` lines 2803962-2803963 (`*(double *)(param_1 + 0x38)` then `* 0xb88 + COMPANY_BASE`); `f_1c32e50` lines 3291898-3291901 | `[field 0x38 in REGI]` is a tagged company reference (stride 0xb88), not a de jure nation id. |
| 08-C011 | "        R_NEXT[+0x00: next_region_id in nation list]" | field | 0x0 | verified | `f_1c01830` lines 3265819-3265832 (head replace and predecessor rewrite); `f_85d580` line 42005 (`sVar5 = *(short *)(sVar5 * 0x910 + REGI_BASE)`) | `[field 0x0 in REGI]` is the next-region id. Both the store and the traversal are visible, so the list is genuinely singly linked. |
| 08-C012 | "- **Global Population**: There are exactly **1,309 administrative regions** in the simulation (`0x52d` in hex), stored in array `G_47e23350` (`0x7ff68e693350`) with record stride `0x910` (2,320 bytes)." | quantity | 0x52d | removed | `1309` decimal is `0x51d`, not `0x52d` (`0x52d` = 1325); the only `0x51d` literals are unrelated assignments at lines 3555833 and 5326376. The live count is `_DAT_463d1b48` (`f_1cbac20` 3345982 init, `f_1c32e50` 3291928 increment) | Fixed 1,309 removed, and the hex conversion corrected: 1,309 = 0x51d, not 0x52d. The shipped data file's extent / 2,320 gives 1,309 records, but that is a derived file-size quotient, not a code constant. |
| 08-C013 | "  - `+0x04`: Owning Region ID (`REGI`)." | field | 0x4 | corrected | `f_937500` lines 192372-192373 (`* 0x8808c + ... + G_26e336f4`); `f_1c32e50` line 3291885 writes the self id at `+0x4` | `[field 0x4 in REGI]` is the self region id. The city `[field 0x4 in VILL]` is a grid index, not a parent REGI id. |
| 08-C014 | "  - `+0x08`: Tactical status flag (`0` = Peace, `1` = Contested, `2` = Under Active Siege)." | field | 0x8 | corrected | `gestion_popu_warville` line 4125207 (`puVar25[2] == 2`) | `[field 0x8 in VILL]` is a uint compared to `2` on the combat path. The corpus does not name the three state values. |
| 08-C015 | "  - `+0x28`: Last bombardment / combat timestamp." | field | 0x28 | verified | `gestion_popu_warville` line 4125208 (`*(int *)(city + 0x28) == _DAT_464916a8 - 1`) | `[field 0x28 in VILL]` is a day stamp compared to the previous day. "Last bombardment" is a reasonable reading but the corpus only establishes the comparison. |
| 08-C016 | "  - `+0x34`: Mayor / Governor character pointer (`PERS`, stride `0xad0`)." | field | 0x34 | removed | `f_f92060` line 1439686 (`*(uint *)(city + 0x34) OR= 0xc0000`) | `[field 0x34 in VILL]` is a flag word, not a PERS pointer. The PERS pointer claim is removed. |
| 08-C017 | "In GPS5, Gross Domestic Product (GDP) is calculated **from the ground up** at the regional level and subsequently aggregated into national totals." | function | 0x14134a0 | removed | `_geolgame__Cgps__calc_nation` lines 2052660-2052667 walks two region chains; no function sums a region GDP field into a nation field. Searches found no accumulator over `0x910` records written to NATI. | Removed. The corpus updates region fields and nation indices; it does not implement a bottom-up regional GDP sum. |
| 08-C018 | "During Phase 24 of the macroeconomic tick (`calc_monde`), national GDP (`NPIB`, located at `NATI + 0x30ce44`) is evaluated by summing across the intrusive linked list of regions:" | function | 0x14134a0 | removed | `calc_nation` lines 2052660-2052667 and 2056596-2056622; `f_23e9640` line 4548009 | Removed. No national GDP sum and no NPIB field are established; `[field 0x30ce44 in NATI]` is an unlabeled float. |
| 08-C019 | "$$\text{Factor}_{\text{persist}} = 0.9972603, \quad \text{Factor}_{\text{delta}} = 0.0027397275$$" | quantity | - | corrected | `calc_nation` lines 2052049-2052050 and 2052083-2052084: `field * 0.9972603 + clamp(x,0,1) * 0.0027397275` | The pair is a real 365-day EMA (sum 1.0000000275, delta ≈ 1/365), but it smooths `[field 0x64346c in NATI]` and `[field 0xc868c in NATI]`, not GDP. |
| 08-C020 | "- **Autonomic Retention Ratio** $\rho_{r} \in [0.10, 0.70]$: Kept by the regional assembly to fund regional health, secondary schools, local roads, and regional civil service." | quantity | - | removed | `init_taxe` lines 3319355-3319410 builds tax relationships but has no two-sided clamp to [0.1,0.7]; search over functions with `0x910` + literals `0.1`/`0.7` found none | Removed. No regional retention ratio in that range is implemented. |
| 08-C021 | "- **Equalization Transfers**: Wealthy industrial regions remit surplus funds into a federal redistribution pool that subsidizes underdeveloped rural regions." | - | - | removed | No function takes a region surplus and adds it to another region or a national pool; `init_taxe` only assigns tax relationships | Removed. No equalisation-transfer arithmetic was found. |
| 08-C022 | `- **Action 332031**: "Frontières modifiées" (Both nations retain the territories currently under military control).`<br>`- **Action 332032**: "Annexion de certains territoires occupés" (Partial territorial annexation).`<br>`- **Action 332040**: "Pays annexé" (Total annexation: Country B is completely absorbed by Country A).` | data | 0x510ff | verified | `data/gt/xan.gt.tsv` rows 332031/332032/332040; `f_19931a0` lines 2804356 and 2804443 (`0x510ff`, `0x51100`); `exec_contrat` line 2802237 (`0x51108`); `set_action_sp2` line 3430880 | The ids are data rows and code literals. The labels are exact. |
| 08-C023 | "*(int16_t *)(region_ptr + 0x28) = annexing_nation_id;" | field | 0x28 | removed | `f_19931a0` lines 2804416-2804428 walks `[field 0x2f0 in NATI]` and calls `f_1992f10`; `f_1992f10` lines 2804025-2804028 writes `+0x38`, not `+0x28` | Removed. No action handler writes a nation id to `[field 0x28 in REGI]`; the observed write is to `[field 0x38 in REGI]`. |
| 08-C024 | "   - Conqueror electorate: Triggers `POP_AQTE` (`0x41515445` / Acquisition of Territory), generating massive patriotic approval ($+10\%$ to $+25\%$ popularity)." | data | 0x41515445 | corrected | `python -I tools/cite.py 0x41515445` -> 0 code hits, data key `POP_AQTE`, `data/gt/xim.gt.tsv` line 21637 | Replaced with `[data xim.gt.tsv "POP_AQTE"]`. The +10% to +25% approval range is not in the corpus. |
| 08-C025 | "   - Defeated electorate: Triggers `POP_PETE` (`0x50455445` / Loss of Territory), delivering severe political backlash ($-20\%$ to $-45\%$ popularity, triggering riot risks)." | data | 0x50455445 | corrected | `python -I tools/cite.py 0x50455445` -> 0 code hits, data key `POP_PETE`, `data/gt/xim.gt.tsv` line 21639 | Replaced with `[data xim.gt.tsv "POP_PETE"]`. The -20% to -45% backlash range is not in the corpus. |
| 08-C026 | "   If independence is negotiated peacefully, the new state assumes a prorated slice of the parent nation's sovereign bond obligations (`LOAN`, stride `0xec`):" | quantity | 0xec | removed | Search for `0xec`-stride loan records multiplied by a region-GDP ratio returned no such expression; no debt partition in `exec_contrat` or `f_19931a0` | Removed. No debt partition arithmetic was found. |
| 08-C027 | "   - Military units (`UNIT`) stationed in the region whose personnel originate locally defect to the new state." | - | - | removed | No `UNIT` record write tied to a seceding region in `exec_contrat` / `f_19931a0`; `calc_unites` handles combat, not secession | Removed. No garrison-defection mechanism was found. |
| 08-C028 | "        CHMN[CHMN Stride 0x98: 64,000 Road/Rail Segments]" | quantity | 0x98 | verified | `Cdatabase__es_base` line 2074177 (`f_1394770(...,64000,...)`); `f_1394770` line 1988353 (`param_2 + 0x98`) | CHMN stride 0x98 and count 64,000 verified by the loader call. |
| 08-C029 | "        CHNO[CHNO Stride 0x260: 64,000 Network Nodes]" | quantity | 0x260 | verified | `Cdatabase__es_base` line 2074187 (`f_1394930(...,64000,...)`); `f_1394930` line 1988428 (`param_2 + 0x260`) | CHNO stride 0x260 and count 64,000 verified by the loader call. |
| 08-C030 | "        PONT[PONT Stride 0x30: 3,000 Bridges]" | quantity | 0x30 | verified | `Cdatabase__es_base` lines 2074938-2074941 (loop 3000, `+0x30`, `f_1499950`); `f_1499950` line 2128493 (`auStackX_8[0] = 0x30`), line 2128495 (`[tag 0x504f4e54 "PONT"]`) | 3,000 bridge records of 0x30 bytes, serialized with the `PONT` tag. |
| 08-C031 | "        TUNN[TUNN Stride 0x30: 2,000 Tunnels]" | quantity | 0x30 | verified | `Cdatabase__es_base` lines 2074904-2074907 (loop 2000, `+0x30`, `f_14a8a60`); `f_14a8a60` line 2137169 (`auStackX_8[0] = 0x30`), line 2137171 (`[tag 0x54554e4e "TUNN"]`) | 2,000 tunnel records of 0x30 bytes, serialized with the `TUNN` tag. |
| 08-C032 | "  $$\text{Cost} = \text{Length}_{\text{km}} \times \text{BaseCostPerKm} \times M_{\text{terrain}} \times M_{\text{labor}}$$" | - | - | removed | `data/gt/xan.gt.tsv` rows 300157/300158/300132 give a single `MISE` and duration; no length × base × terrain × labour expression in the corpus | Removed. The action rows do not implement the old cost formula. |
| 08-C033 | "  Where $M_{\text{terrain}} = 1.0$ (Flat plains), $2.5$ (Mountainous/Tunnels), $3.2$ (Water crossings/Bridges)." | - | - | removed | No 1.0/2.5/3.2 terrain multiplier triple in the corpus or the action rows | Removed. |
| 08-C034 | "  $$\text{FrictionFactor} = \frac{1.0}{1.0 + 0.35 \times C_{r}}$$" | - | - | removed | No `0.35` friction term or connectivity index in the corpus | Removed. |
| 08-C035 | "&#124; **Nuclear Power Plant** &#124; `504809` (`VICN`) &#124; Uranium (`DFAC` 42) &#124; 1,000 – 4,000 MW &#124; 0.00 &#124; Nuclear Meltdown (`POP_ACCN`) &#124;" | data | 504809 | corrected | `data/gt/xan.gt.tsv` row 504809 = `Centrale nucléaire`, reaction `VICN`, parameter 256; row 310001 = `Nucléaire` with `ENEE*8;ENER*8;ENVI*-10`, 2555 days | 504809 exists as a data row but carries no MW, DFAC fuel or cost cell. The plant rows are 310000/310001/310004/310008. |
| 08-C036 | "&#124; **Coal / Thermal Plant**&#124; `300110` (`CENT`) &#124; Coal / Gas (`DFAC` 38/39) &#124; 500 – 2,000 MW &#124; High (`CO2P`) &#124; Acid Rain & Smog Unrest &#124;" | data | 300110 | corrected | `data/gt/xan.gt.tsv` row 300110 = `Tremblement de terre` (earthquake), not a plant | Removed the coal-plant identification. 310000 (`Energie fossile`) is the thermal row. |
| 08-C037 | "&#124; **Hydroelectric Dam**   &#124; `300112` (`BARR`) &#124; Zero (River topography) &#124; 200 – 10,000 MW&#124; 0.00 &#124; Dam Breach / Flooding (`CATA`) &#124;" | data | 300112 | corrected | `data/gt/xan.gt.tsv` row 300112 = `Éruptions volcaniques` (volcanic eruptions), not a dam | Removed the hydro-dam identification. No hydro plant row was found in the inspected energy block. |
| 08-C038 | "&#124; **Solar / Wind Farm**   &#124; `300118` (`AREN`) &#124; Zero (Sun/Wind climate) &#124; 50 – 500 MW &#124; 0.00 &#124; Intermittent weather variance &#124;" | data | 300118 | corrected | `data/gt/xan.gt.tsv` row 300118 = `Pollution`, not a solar/wind farm | Removed. The wind row is 310004 and the solar row is 310008. |
| 08-C039 | "   $$\text{GridBalance} = \sum_{p \in \text{PowerPlants}} \text{GenMW}_{p} - \left(\text{Demand}_{\text{industrial}} + \text{Demand}_{\text{municipal}} + \text{Demand}_{\text{civilian}}\right)$$" | - | - | removed | No generation-minus-demand aggregate in `taux_critere`, `f_a45270`, `f_1c12160` or the energy data rows | Removed. No grid-balance arithmetic was found. |
| 08-C040 | "2. **Power Shortage Crisis (`POP_PNEE`)**:" | tag | 0x504e4545 | corrected | `taux_critere` line 4374801 (`0x504e4545` -> factor `1.2` or `6.0`); `f_a45270` line 356626 calls `taux_critere` with the tag; `data/gt/xim.gt.tsv` line 21605 | `PNEE` is a real tag and a data key. The corpus does not implement a 20%-50% shutdown; that mechanism claim is removed. |
| 08-C041 | "3. **Nuclear Catastrophes (`POP_ACCN`)**:" | tag | 0x4143434e | corrected | `f_1c12160` line 3275564 (`0x4143434e` passed to `f_21c7410`); `data/gt/xim.gt.tsv` line 21643 | `ACCN` is a real tag and a data key. No reactor-failure check or health/agriculture liability formula was found; those mechanism claims are removed. |
| 08-C042 | "- **Land Army Bases (`330096`)**:" | data | 0x50970 | verified | `data/gt/xan.gt.tsv` row 330096; `Cmenu_droit__do_action` line 5342038 (`case 0x50970`); `f_203bc10` line 5346849 | The id is a data row and a code literal, matching accepted Volume 3. The branch toggles a menu bit. |
| 08-C043 | "  - Garrison capacity: 5,000 to 50,000 personnel." | quantity | - | removed | `Cmenu_droit__do_action` line 5342038 toggles `_DAT_464de228 ^ 0x10000000000`; no capacity field or check | Removed. No garrison-capacity mechanism was found. |
| 08-C044 | "  - Essential for constructing capital ships (aircraft carriers, destroyers, submarines); without active naval shipyards, foreign procurement is mandatory." | - | - | removed | `Cmenu_droit__do_action` line 5342043 toggles `_DAT_464de228 ^ 0x20000000000`; no shipyard-construction branch | Removed. No naval-shipyard construction mechanism was found. |
| 08-C045 | "  - Projects radar surveillance spheres and surface-to-air missile (SAM) defense umbrellas over adjacent regions." | - | - | removed | `Cmenu_droit__do_action` line 5342048 toggles the next menu bit; no radar/SAM range or coverage arithmetic | Removed. |
| 08-C046 | "  Every military base incurs continuous monthly maintenance amortization billed directly to the Defense Ministry budget (`BDEM`/`CATE`). Abandoned or unfunded bases suffer structural decay and reduced combat readiness." | - | - | removed | No base-upkeep or readiness-decay expression in `Cmenu_droit__do_action` or `f_203bc10`; `BDEM`/`CATE` are variable codes, not base-upkeep arithmetic | Removed. |

## Closing statistics

- **Unique RVAs / offsets tested (non-`-`):** 0x0, 0x4, 0x8, 0x28, 0x30, 0x34, 0x38, 0x68, 0x98, 0x260, 0x2f0, 0x52d, 0x910, 0x9cf8, 0x30ce44, 0x645310, 0x14134a0, 0x4143434e, 0x41515445, 0x50455445, 0x504e4545, 0x50970, 0x510ff, 0x85d580, 0xec, 504809, 300110, 300112, 300118 (29 unique tokens). Of these, 10 are 5-8 digit hexadecimal tokens that `tools/cite.py` resolves; the rest are 1-4 hex-digit field offsets, strides, or decimal data ids.
- **Per-class counts:** function 3; quantity 13; field 11; data 8; tag 2; `-` 9. Total 46 rows.
- **Wrong addresses / classes (old -> new):**
  - `0x41515445` (presented as a binary constant) -> `[data xim.gt.tsv "POP_AQTE"]`.
  - `0x50455445` (presented as a binary constant) -> `[data xim.gt.tsv "POP_PETE"]`.
  - `0x30ce44` (presented as `NATI + NPIB`) -> `[field 0x30ce44 in NATI]`, an unlabeled float.
  - `0x52d` (presented as the region count) -> removed; the count is `[global 0x463d1b48 i32]`.
  - `0x645310`, `0x910`, `0x9cf8` -> `[qty ... stride]` with the multiplying expression.
  - `0x2f0`, `0x28`, `0x38`, `0x0` -> `[field ... in NATI/REGI]`.
  - `0xec` (presented as a LOAN stride used in a debt formula) -> removed; no debt formula exists.
- **Claims removed:** 17 (08-C012, C016, C017, C018, C020, C021, C023, C026, C027, C032, C033, C034, C039, C043, C044, C045, C046).
- **Claims corrected:** 18 (08-C002, C003, C005, C007, C008, C009, C010, C013, C014, C019, C024, C025, C035, C036, C037, C038, C040, C041).
- **Claims verified:** 11 (08-C001, C004, C006, C011, C015, C022, C028, C029, C030, C031, C042).
- **Claims added:**
  1. The live region counter `[global 0x463d1b48 i32]`, initialised to 1 and incremented per allocation.
  2. Nation activation is `[field 0x2f0 in NATI] != 0`, not a separate flag.
  3. `[field 0x14 in REGI]` is the integer nation id.
  4. `[field 0x38 in REGI]` is a company reference at stride `0xb88`.
  5. PONT/TUNN are tagged 0x30-byte serializer records, 3,000 and 2,000 respectively.
  6. `506900` is a data row only; the code literal is `0x7bc15` (506901).
  7. The 365-day EMA targets `[field 0x64346c in NATI]` and `[field 0xc868c in NATI]`, not GDP.
  8. Energy rows 310000/310001/310004/310008 carry the real plant formulas.
  9. 300110/300112/300118 are earthquake / volcanic-eruption / pollution rows.
  10. 330096-330098 toggle menu bits in `_DAT_464de228`.

## Constructs the corpus does not implement

1. A bottom-up regional GDP sum or a national raw-GDP total over region chains.
2. A GDP-targeted 365-day EMA (the EMA exists on two other fields).
3. A regional retention ratio in [0.10, 0.70] and equalisation transfers.
4. A write of an annexing nation id to `[field 0x28 in REGI]`.
5. GDP or asset transfer between defeated and annexing nations.
6. Sovereign-debt partitioning by GDP ratio and a credit-rating downgrade to `D`.
7. Garrison defection or fixed-infrastructure seizure on secession.
8. A city id at `[field 0x0 in VILL]`, a parent-region id at `[field 0x4 in VILL]`, or a mayor `PERS` pointer.
9. A regional connectivity index, a trade-cost multiplier, or `1/(1 + 0.35*C)`.
10. A terrain/labour highway cost formula with multipliers 1.0/2.5/3.2.
11. Megawatt capacities, CO2 coefficients, or the `DFAC` 38/39/42 fuel mapping.
12. A national grid balance, 20%-50% shutdown, rolling blackouts, or a reactor-failure check.
13. Base garrison capacity 5,000-50,000, shipyard construction, SAM/radar umbrellas, or base upkeep.
14. A fixed 1,309 region count; the count is the runtime counter `[global 0x463d1b48 i32]`.

## Evidence commands

```
python -I tools/cite.py 0x30ce44   -> quantity (13 hits)
python -I tools/cite.py 0x645310   -> quantity (10,171 hits)
python -I tools/cite.py 0x41515445 -> data (POP_AQTE, xim.gt.tsv line 21637, 0 code hits)
python -I tools/cite.py 0x50455445 -> data (POP_PETE, xim.gt.tsv line 21639, 0 code hits)
python -I tools/cite.py 0x9cf8     -> unresolved (4-digit; manual qty from f_13a07b0)
python -I tools/cite.py 0x910      -> unresolved (3-digit; manual qty from f_139bb40/f_85d580)
python -I tools/cite.py 0x504f4e54 -> tag "PONT"
python -I tools/cite.py 0x54554e4e -> tag "TUNN"
python -I tools/cite.py 0x504e4545 -> tag "PNEE"
python -I tools/cite.py 0x4143434e -> tag "ACCN"
python -I tools/cite.py 0x463d1b48 -> global i32
python -I tools/cite.py 0x322a3f4  -> global i32
```
