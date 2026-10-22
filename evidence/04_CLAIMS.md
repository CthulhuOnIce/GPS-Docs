# Volume 4 Claims Ledger

Per-volume evidence for `_whitepaper/04_POLITICS_AND_POPULARITY.md` (D-006). Every row cites `recomp/full_fns.clean.c` by function and line. `class` is one of function, global, tag, quantity, field, data, formula, structural, source-file. `rva` is the value tested; `-` means the claim is structural/formula-only. Verdicts are limited to verified, corrected and removed.

## Claim ledger

| claim_id | old_text_verbatim | class | rva | verdict | corpus_evidence(function+line) | replacement_summary |
|---|---|---|---|---|---|---|
| C001 | Every sovereign nation record (`NATI`) embeds a **120-dimensional political opinion ledger** | structural | - | verified | f_1cd7ef0 line 3361899 (`0x77 < iVar2`); f_1cd2170 line 3358584 (`< 0x78`) | Capacity is 120 entries (index 0..119), terminated by a zero tag. The number of populated tags with evidence is 102; the remaining 18 positions are not separately enumerated. |
| C002 | Located within each nation record `*0x46208908 + (nation_idx * 0x645310)` | global | 0x46208908 | verified | f_215d460 line 4084448; f_215cd40 line 4084275; f_215ba00 line 4083549 | The base global is `[global 0x46208908 ptr]`; the nation stride is `[qty 0x645310 stride]`. |
| C003 | `+0x620ee0`: `int32_t theme_ids[120]` | field | 0x620ee0 | verified | f_1cd7ef0 line 3361889 (`piVar1 = (int *)(param_1 + 0x620ee0)`) | Confirmed as a 120-entry int array scanned by tag value. |
| C004 | `+0x6210c0`: `float theme_importance[120]` | field | 0x6210c0 | verified | f_1cd7ef0 line 3361903 (`*(float *)(param_1 + 0x6210c0 + i*4)`) | Confirmed as a float array read by the same index as the tag. |
| C005 | `+0x6212a0`: `float theme_position[120]` | field | 0x6212a0 | corrected | f_1cd7ef0 line 3361904 (`*(undefined4 *)(param_1 + 0x6212a0 + i*4)`) | The only corpus read is a 32-bit word, not a float; labelled neutral 'target word'. The [-1,+1] float label is not established. |
| C006 | `+0x621480`: `float theme_value[120]` | field | 0x621480 | verified | f_2143660 line 4069159 (read-modify-write of `+ 0x621480 + i*4`); f_1cd7ef0-adjacent reads at line 1975730 | Confirmed float array, read and written. |
| C007 | `+0x621660`: `float ledger_scale` — National opinion scaling constant. | field | 0x621660 | corrected | f_1cd7ef0 line 3361906 (`param_5 = *(float *)(param_1 + 0x621660)`); f_215d330 line 4084435 sets element 0 to 1.0f | It is the first float of a fifth 480-byte block. Element 0 is used as a multiplier; the remaining 119 floats are not shown to be used. |
| C008 | **The 120-Iteration Simulation Bound (`0x78`)** | quantity | 0x78 | verified | f_1cd7ef0 line 3361899; f_2149fc0 line 4073125; f_214a2e0 line 4073229; f_1cd2170 line 3358584 | 120 iterations confirmed. |
| C009 | **The 128-Element Memory Block Alignment (`0x80`)** | quantity | 0x80 | removed | no 0x80 bound or 128-element alignment found in the theme-array functions | Removed: no arithmetic or loop uses 128 for this structure. |
| C010 | Below is the complete, exhaustive index of every slot in the national opinion array | structural | - | corrected | three searches over 102 non-zero tags; 9 data-key-only, 93 code immediate | 102 non-zero tags survive (102 verified, 0 corrected). The 18 zero rows are removed from the tag classification and remain only as reserved placeholders; the table reports search counts and evidence class per tag. |
| C011 | &#124; **100** &#124; `SOLV` &#124; `0xd34f4c56` &#124; ... | tag | 0x534f4c56 | verified | f_1cd8ba0 line 3362306 (`(param_1 & 0x7fffffff) != SOLB && != SOLV`); f_1cd6d20 lines 3361376-3361377; f_1cd73d0 lines 3361561-3361562; f_1cd78d0 lines 3361716-3361717 | 0xd34f4c56 is the bit-31-set SOLV tag: 0x534f4c56 &#124; 0x80000000. The mask recovers 0x534f4c56. Not a byte-order error. |
| C012 | Bit 31 is utilized by the engine as an active state flag indicating whether the sovereign solvency crisis condition is actively firing. | formula | 0x7fffffff | removed | f_1cd8ba0 3362306, f_1cd6d20 3361376-3361377, f_1cd73d0 3361561-3361562, f_1cd78d0 3361716-3361717 only clear the bit before tag comparison; no branch reads it as a crisis state | Removed: the 'active solvency-crisis state' semantics is not established. The mask and the 0xd34f4c56 value are verified in C011/C046. |
| C013 | In addition to the 102 persistent national ledger themes, `data/gt/xim.gt.tsv` defines specialized dynamic event tokens (e.g., pandemic themes `POP_VACC`, `POP_CONF`, `POP_MASQ`, `POP_REST`, `POP_TELE`; trade pact themes `POP_CODH`, `POP_LIEC`; moral/marital events `POP_MARI`, `POP_DIVO`). These dynamic tokens are injected into temporary event calculation contexts (`actions.bin`) and project deltas into the persistent ledger slots via the sensitivity kernel before decaying over time. | data | - | corrected | 0x41515445=0 hits, 0x50455445=0 hits; POP_AQTE and POP_PETE are keys in data/gt/xim.gt.tsv only (D-012). Bare AQTE also occurs in xrn.gt.tsv; bare PETE occurs in xrn.gt.tsv and in xvs.gt.tsv only inside RU_SAINT_PETERSBURG. | Corrected: AQTE/PETE are data-table keys (POP_AQTE/POP_PETE, xim.gt.tsv only), not binary constants. The `actions.bin` injection claim is not verified and is dropped. |
| C014 | **Engine Alias Redirection Table** ... 18 pairs | structural | 0x215b3a0 | verified | f_215b3a0 lines 4083296-4083359 (17 return statements + 1 assignment for TFO5) | 18 alias pairs survive; 0 removed. |
| C015 | **Education**: `SCO2` $\to$ `SCOL` (Slot 17) | tag | 0x215b3a0 | verified | f_215b3a0 lines 4083296-4083297 | Pair confirmed. Same form for HOS2/VIE2/FAM2/FEM2/LOG2/HUM2/REC2/CUL2/SPO2/TRA2/JUS2/TFO1-5. |
| C016 | **Militarism**: `ARM2` $\to$ `ARME` (Slot 51) | tag | 0x215b3a0 | verified | f_215b3a0 lines 4083303-4083304 | ARM2 -> ARME confirmed. The old catalog slot row was ARM3; both tags exist, but the pair is ARM2. |
| C017 | ### 2.1 The Action Importance Kernel (`importance_action` / `0x215be60`) | function | 0x215be60 | verified | corpus line 4083652, size 2899 | Function and RVA verified. |
| C018 | ### 2.3 The Theme Delta Resonance Kernel (`theme_delta` / `0x215d4a0`) | function | 0x215d4a0 | corrected | corpus line 4084451, size 456; arithmetic lines 4084462-4084500 | Arithmetic is a piecewise resonance function; label is supported but the theme_importance*ledger_scale factor is applied by the caller at line 4075505, not inside f_215d4a0. |
| C019 | ### 2.2 Sensitivity Damping (`f_214a610`) | function | 0x214a610 | corrected | corpus line 4073288, size 1381; interpolation lines 4073464-4073484 | f_214a610 is a multi-point interpolation and [-1,+1] clamp, not a sens/10 damping curve. |
| C020 | Importance_raw x (1 - sens/10) / (sens/10 + 1) | formula | 0x214a610 | removed | the sens/10 expression does not occur; the actual +0x74 multiplier is in importance_action line 4084019 | Removed. Replaced by (sens/100 + 1) applied only to positive values, then clamp to [-1,+1]. |
| C021 | steps x 1.33 if bit 16 ... steps x 2.00 if bit 17 ... bit 18 ... bit 19 | quantity | 0x215be60 | verified | importance_action lines 4083924-4083936 | 1.33 and 2.0 confirmed; bit order and positive/negative conditions confirmed. |
| C022 | Parameters A, B are loaded from offset `+0x224` of the action record (`DEFA`) | field | 0x224 | corrected | importance_action lines 4083969-4083970: A at +0x224, B at +0x45 pointer units = byte 0x228 | Wrong field B -> 0x228. A remains at 0x224. |
| C023 | Importance_raw = clamp(v, -4.0, +4.0) | quantity | 0x215be60 | verified | importance_action lines 4084004-4084006 | Confirmed literal -4.0 and +4.0 clamp. |
| C024 | f(x) = (1 - &#124;x&#124;*0.75)/0.20 ... / ((1-&#124;x&#124;)*0.75)/0.80 - 1.0 | quantity | 0x215d4a0 | verified | f_215d4a0 lines 4084483-4084488 | Piecewise constants 0.20, 0.75, 0.80 confirmed. |
| C025 | p = (Importance x w_j) x (theme_importance_j x ledger_scale) | formula | 0x215d4a0 | corrected | f_215d4a0 line 4084489 (`param_5 = param_1 * param_2`); caller line 4075505 multiplies by +0x621660 | The kernel multiplies two arguments only; the ledger_scale factor is applied by the caller. |
| C026 | Pop = 0.75 x (...) + 0.25 x (...) + sum(Offsets) | formula | 0x219cfb0 | removed | hex 0.75 double 2 hits (393798, 5302988); hex 0.25 double 2 hits (282433, 282623, f_9d0ba0); decimal 0.75: 543 token-boundary / 547 raw; decimal 0.25: 1,123 token-boundary / 1,140 raw. Read of f_215d4a0, f_214a610, stat_popularite_add (0x1cc4440) and f_215ba00 finds no three-term popularity expression. | Removed. The only established aggregate is the population-weighted mean in f_1412580 lines 2051611-2051807. |
| C027 | divisor `2 * N_peers` | formula | - | removed | no division by a doubled peer count in f_215d4a0, f_214a610, stat_popularite_add (0x1cc4440) or f_215ba00 | Removed. The aggregate divides by the sum of nation weights, not by a peer count. |
| C028 | Exogenous Offsets: scandal -0.05..-0.25; war +0.10..+0.30; famine -0.15..-0.40 | formula | - | removed | no additive offset term in f_215d4a0, f_214a610, stat_popularite_add (0x1cc4440) or f_215ba00 | Removed. No evidence for an additive offset sum. |
| C029 | inside `calc_perso` (`0x219cfb0`) | function | 0x219cfb0 | corrected | corpus line 4125085: `// ==== 219cfb0 gestion_popu_warville size=4172` | Wrong identity 0x219cfb0 calc_perso -> gestion_popu_warville. The popularity equation is not at this RVA. |
| C030 | `stat_popularite_add` (`0x1cc4440`) | function | 0x1cc4440 | verified | corpus line 3349581, size 156 | Verified: copies one float into a 365-slot ring. |
| C031 | (added) popularity ring base and length | field | 0x3bd0 | verified | stat_popularite_add line 3349592; modulo 0x16d at line 3349589 | Added: ring is 365 slots at +0x3bd0; value read from actor+0x49c. |
| C032 | DifficultyFactor = 0.38 Easy / 1.00 Normal / 1.45 Hard | quantity | 0x1cd2170 | verified | f_1cd2170 lines 3358518-3358535 | 0.38, 1.0, 1.45 and 0.25 literals confirmed; Easy/Hard labels are not in the code, only _DAT_32210d0 branch values. |
| C033 | additional stabilizing divisor of 3.0 | quantity | 0x1cd2170 | verified | f_1cd2170 line 3358586 (`fVar20 = fVar20 / 3.0`) | Confirmed, with the code's player/controller condition. |
| C034 | Society is stratified into 15,000 active group instances | quantity | 0x5d859dc8 | verified | array_inventory.tsv row holder _DAT_5d859dc8, stride 0xb88, count 15000; f_146e300 line 2102948 | Confirmed: 15,000 GROU records, byte size 0xb88*15000 = 44,280,000. |
| C035 | `GROU` ... stride `0xb88` / 2,952 bytes | quantity | 0xb88 | verified | line 133305 (`index * 0xb88 + _DAT_5d859dc8`); f_146e300 line 2102948 | Confirmed. |
| C036 | `+0x00` group id, `+0x04` nation, `+0x10` influence, `+0x24` satisfaction, `+0x89` alignment, `+0x9f` seats, `+0x2a0` radicalization, `+0x2ba` reserves | field | - | removed | no GROU base expression uses these displacements in the corpus | Removed: none of the eight offsets was tied to the GROU array. |
| C037 | `TROU` array (stride `0x348`, 6,000 slots) | quantity | 0x348 | verified | array_inventory.tsv row holder G_60b65f30, stride 0x348, count 6000; add_trouble line 4666905 | Confirmed: 6,000 TROU records. |
| C038 | `add_trouble` (`0x247eb60`) | function | 0x247eb60 | verified | corpus line 4664527, size 5779 | Verified. |
| C039 | **Gravity Progression (`trouble_update_gravite` / `0x248bc10`)** | function | 0x248bc10 | verified | corpus line 4672015, size 1297 | Verified function and RVA. |
| C040 | Gravity(t) = Gravity(t-1) x 0.95 + AngerFactor x Mobilization | formula | 0x248bc10 | removed | no 0.95 literal; no AngerFactor*Mobilization product; actual code lines 4672038-4672128 | Removed. Replaced by the observed target pursuit and flag scales 1.2/1.4/0.7, -30/+10. |
| C041 | Trigger Condition: GROU.satisfaction < -0.35 and GROU.influence > 0.05 | formula | 0x247eb60 | removed | neither -0.35 nor 0.05 occurs in add_trouble or trouble_update_gravite | Removed. Callers pass a trouble type to add_trouble; the trigger predicate is not established. |
| C042 | `TROU` / `gps_troubles.cpp` | source-file | 0x248bc10 | verified | corpus line 4672246 assertion string `... gps_troubles.cpp ...`; add_trouble is 0x247eb60 | The path string exists; the TROU association is supported by the same file. |
| C043 | dynamic event tokens ... injected into temporary event calculation contexts (`actions.bin`) | data | - | corrected | POP_VACC, POP_CONF, POP_MASQ, POP_REST, POP_TELE, POP_CODH, POP_LIEC, POP_MARI, POP_DIVO are xim.gt.tsv keys | Reclassified as data-table keys. The `actions.bin` injection claim is not verified and is dropped. |
| C044 | (added) `0x215ba00` wrapper | function | 0x215ba00 | verified | corpus line 4083536, size 218; calls f_215bae0 at line 4083559 (header 4083563, 890 bytes) | Added: real function; it wraps the approval average in f_215bae0. |
| C045 | `FUN_7ff648547ef0` (`d_01c.c:147786`) and `calc_opinion` (`d_021.c:58624`) | function | - | removed | `FUN_7ff648547ef0` occurs 0 times in the corpus; no `calc_opinion` header | Removed: these decompiler labels are not RVAs in the corpus of record. |
| C046 | (added) SOLV/SOLB bit-31 tag mask | tag | 0x7fffffff | verified | f_1cd8ba0 line 3362306; f_1cd6d20 lines 3361376-3361377; f_1cd73d0 lines 3361561-3361562; f_1cd78d0 lines 3361716-3361717 | Restored mechanism: 0x534f4c56 &#124; 0x80000000 = 0xd34f4c56; the four functions compare the masked word against SOLV/SOLB. |

## Per-tag slot classification (102 surviving tags)

The three searches were: code immediate (big-endian u32 per D-009), quoted code text, and data-table key. 92 tags are code-immediate rows, 1 code-immediate row also carries a flagged SOLV value (slot 100), and 9 are data-key-only rows (PRIV plus ABMO, COLT, UEUR, NUME, ROBO, TRAD, LCUL, LAIC).

| Tag | Class | Evidence |
|---|---|---|
| `1DIC` | code-immediate | [tag 0x31444943 "1DIC"] [fn 0x15ee940 Cdlg_action_law___event] |
| `2LIB` | code-immediate | [tag 0x324c4942 "2LIB"] [fn 0x15ee940 Cdlg_action_law___event] |
| `3CAP` | code-immediate | [tag 0x33434150 "3CAP"] [fn 0x15ee940 Cdlg_action_law___event] |
| `4SEC` | code-immediate | [tag 0x34534543 "4SEC"] [fn 0x15ee940 Cdlg_action_law___event] |
| `5IND` | code-immediate | [tag 0x35494e44 "5IND"] [fn 0x15ee940 Cdlg_action_law___event] |
| `HOSP` | code-immediate | [tag 0x484f5350 "HOSP"] [fn 0x15ee940 Cdlg_action_law___event] |
| `HOS2` | code-immediate | [tag 0x484f5332 "HOS2"] [fn 0x2342a00 reaction_yesno_sp8] |
| `VIEU` | code-immediate | [tag 0x56494555 "VIEU"] [fn 0x15ee940 Cdlg_action_law___event] |
| `VIE2` | code-immediate | [tag 0x56494532 "VIE2"] [fn 0x215b3a0 f_215b3a0] |
| `JEUN` | code-immediate | [tag 0x4a45554e "JEUN"] [fn 0x22e4ad0 taux_critere] |
| `FAMI` | code-immediate | [tag 0x46414d49 "FAMI"] [fn 0x13f3290 _geolgame__Cgps__calc_monde] |
| `FAM2` | code-immediate | [tag 0x46414d32 "FAM2"] [fn 0x215b3a0 f_215b3a0] |
| `FEMM` | code-immediate | [tag 0x46454d4d "FEMM"] [fn 0x21554b0 f_21554b0] |
| `FEM2` | code-immediate | [tag 0x46454d32 "FEM2"] [fn 0x215b3a0 f_215b3a0] |
| `AVOR` | code-immediate | [tag 0x41564f52 "AVOR"] [fn 0x2593bd0 get_variable] |
| `LOGE` | code-immediate | [tag 0x4c4f4745 "LOGE"] [fn 0x15ee940 Cdlg_action_law___event] |
| `LOG2` | code-immediate | [tag 0x4c4f4732 "LOG2"] [fn 0x215b3a0 f_215b3a0] |
| `SCOL` | code-immediate | [tag 0x53434f4c "SCOL"] [fn 0x15ee940 Cdlg_action_law___event] |
| `SCO2` | code-immediate | [tag 0x53434f32 "SCO2"] [fn 0x232ea90 reaction_yesno_sp15] |
| `INTE` | code-immediate | [tag 0x494e5445 "INTE"] [fn 0x15ee940 Cdlg_action_law___event] |
| `HUMA` | code-immediate | [tag 0x48554d41 "HUMA"] [fn 0x22b9d00 cmpcode_reaction_sp4] |
| `HUM2` | code-immediate | [tag 0x48554d32 "HUM2"] [fn 0x215b3a0 f_215b3a0] |
| `ABMO` | data-key | [data xim.gt.tsv "POP_ABMO"] |
| `PROS` | code-immediate | [tag 0x50524f53 "PROS"] [fn 0x1cfb560 calc_effet_loi] |
| `CHAS` | code-immediate | [tag 0x43484153 "CHAS"] [fn 0x1cd4110 f_1cd4110] |
| `HAND` | code-immediate | [tag 0x48414e44 "HAND"] [fn 0x232ea90 reaction_yesno_sp15] |
| `CONS` | code-immediate | [tag 0x434f4e53 "CONS"] [fn 0x232ea90 reaction_yesno_sp15] |
| `ANIM` | code-immediate | [tag 0x414e494d "ANIM"] [fn 0x1cd4110 f_1cd4110] |
| `JEUX` | code-immediate | [tag 0x4a455558 "JEUX"] [fn 0x232ea90 reaction_yesno_sp15] |
| `RECH` | code-immediate | [tag 0x52454348 "RECH"] [fn 0x22e4ad0 taux_critere] |
| `REC2` | code-immediate | [tag 0x52454332 "REC2"] [fn 0x215b3a0 f_215b3a0] |
| `PHYS` | code-immediate | [tag 0x50485953 "PHYS"] [fn 0x232ea90 reaction_yesno_sp15] |
| `ASTR` | code-immediate | [tag 0x41535452 "ASTR"] [fn 0x232ea90 reaction_yesno_sp15] |
| `SPAC` | code-immediate | [tag 0x53504143 "SPAC"] [fn 0x22f1a70 reaction_idyesno] |
| `AERO` | code-immediate | [tag 0x4145524f "AERO"] [fn 0x2593bd0 get_variable] |
| `GENE` | code-immediate | [tag 0x47454e45 "GENE"] [fn 0x232ea90 reaction_yesno_sp15] |
| `SIDA` | code-immediate | [tag 0x53494441 "SIDA"] [fn 0x15ee940 Cdlg_action_law___event] |
| `CANC` | code-immediate | [tag 0x43414e43 "CANC"] [fn 0x22e4ad0 taux_critere] |
| `CULT` | code-immediate | [tag 0x43554c54 "CULT"] [fn 0x1e2eef0 gere_histo] |
| `CUL2` | code-immediate | [tag 0x43554c32 "CUL2"] [fn 0x232ea90 reaction_yesno_sp15] |
| `MUSV` | code-immediate | [tag 0x4d555356 "MUSV"] [fn 0x22e4ad0 taux_critere] |
| `MUSI` | code-immediate | [tag 0x4d555349 "MUSI"] [fn 0x1e18460 getval_rdv_rdv_test_cmp] |
| `MUSR` | code-immediate | [tag 0x4d555352 "MUSR"] [fn 0x1e38e40 f_1e38e40] |
| `PEIN` | code-immediate | [tag 0x5045494e "PEIN"] [fn 0x22ad5a0 cmpcode_reaction] |
| `LITT` | code-immediate | [tag 0x4c495454 "LITT"] [fn 0x232ea90 reaction_yesno_sp15] |
| `CINE` | code-immediate | [tag 0x43494e45 "CINE"] [fn 0x1e38e40 f_1e38e40] |
| `THEA` | code-immediate | [tag 0x54484541 "THEA"] [fn 0x1e38e40 f_1e38e40] |
| `DANS` | code-immediate | [tag 0x44414e53 "DANS"] [fn 0x1e38e40 f_1e38e40] |
| `MODE` | code-immediate | [tag 0x4d4f4445 "MODE"] [fn 0x142f4d0 f_142f4d0] |
| `ARCH` | code-immediate | [tag 0x41524348 "ARCH"] [fn 0x1e18460 getval_rdv_rdv_test_cmp] |
| `MEDI` | code-immediate | [tag 0x4d454449 "MEDI"] [fn 0x219e5e0 f_219e5e0] |
| `ARME` | code-immediate | [tag 0x41524d45 "ARME"] [fn 0x22b9d00 cmpcode_reaction_sp4] |
| `SENA` | code-immediate | [tag 0x53454e41 "SENA"] [fn 0x1be3950 f_1be3950] |
| `ARMR` | code-immediate | [tag 0x41524d52 "ARMR"] [fn 0x232ea90 reaction_yesno_sp15] |
| `ARM3` | code-immediate | [tag 0x41524d33 "ARM3"] [fn 0x153fbe0 f_153fbe0] |
| `SPOR` | code-immediate | [tag 0x53504f52 "SPOR"] [fn 0x1e2eef0 gere_histo] |
| `SPO2` | code-immediate | [tag 0x53504f32 "SPO2"] [fn 0x215b3a0 f_215b3a0] |
| `FOOT` | code-immediate | [tag 0x464f4f54 "FOOT"] [fn 0x1e18460 getval_rdv_rdv_test_cmp] |
| `TENN` | code-immediate | [tag 0x54454e4e "TENN"] [fn 0x1e18460 getval_rdv_rdv_test_cmp] |
| `AUTO` | code-immediate | [tag 0x4155544f "AUTO"] [fn 0x1e18460 getval_rdv_rdv_test_cmp] |
| `ENVI` | code-immediate | [tag 0x454e5649 "ENVI"] [fn 0x15ee940 Cdlg_action_law___event] |
| `TRAN` | code-immediate | [tag 0x5452414e "TRAN"] [fn 0x232ea90 reaction_yesno_sp15] |
| `TRA2` | code-immediate | [tag 0x54524132 "TRA2"] [fn 0x215b3a0 f_215b3a0] |
| `TRAI` | code-immediate | [tag 0x54524149 "TRAI"] [fn 0x1a19240 f_1a19240] |
| `AVIO` | code-immediate | [tag 0x4156494f "AVIO"] [fn 0x232ea90 reaction_yesno_sp15] |
| `ROUT` | code-immediate | [tag 0x524f5554 "ROUT"] [fn 0x22ad5a0 cmpcode_reaction] |
| `INDU` | code-immediate | [tag 0x494e4455 "INDU"] [fn 0x22b9d00 cmpcode_reaction_sp4] |
| `ENER` | code-immediate | [tag 0x454e4552 "ENER"] [fn 0x22b9d00 cmpcode_reaction_sp4] |
| `ENEE` | code-immediate | [tag 0x454e4545 "ENEE"] [fn 0x22b9d00 cmpcode_reaction_sp4] |
| `AGRI` | code-immediate | [tag 0x41475249 "AGRI"] [fn 0x232ea90 reaction_yesno_sp15] |
| `PECH` | code-immediate | [tag 0x50454348 "PECH"] [fn 0x22e4ad0 taux_critere] |
| `JUST` | code-immediate | [tag 0x4a555354 "JUST"] [fn 0x2593bd0 get_variable] |
| `JUS2` | code-immediate | [tag 0x4a555332 "JUS2"] [fn 0x215b3a0 f_215b3a0] |
| `TERR` | code-immediate | [tag 0x54455252 "TERR"] [fn 0x2593bd0 get_variable] |
| `SECT` | code-immediate | [tag 0x53454354 "SECT"] [fn 0x2593bd0 get_variable] |
| `ARPE` | code-immediate | [tag 0x41525045 "ARPE"] [fn 0x15ee940 Cdlg_action_law___event] |
| `TFO1` | code-immediate | [tag 0x54464f31 "TFO1"] [fn 0x215b3a0 f_215b3a0] |
| `TFO2` | code-immediate | [tag 0x54464f32 "TFO2"] [fn 0x215b3a0 f_215b3a0] |
| `TFO3` | code-immediate | [tag 0x54464f33 "TFO3"] [fn 0x215b3a0 f_215b3a0] |
| `TFO4` | code-immediate | [tag 0x54464f34 "TFO4"] [fn 0x215b3a0 f_215b3a0] |
| `TFO5` | code-immediate | [tag 0x54464f35 "TFO5"] [fn 0x215b3a0 f_215b3a0] |
| `COLT` | data-key | [data xim.gt.tsv "POP_COLT"] |
| `HOMO` | code-immediate | [tag 0x484f4d4f "HOMO"] [fn 0x1cfb560 calc_effet_loi] |
| `EUTH` | code-immediate | [tag 0x45555448 "EUTH"] [fn 0x1cfb560 calc_effet_loi] |
| `CANN` | code-immediate | [tag 0x43414e4e "CANN"] [fn 0x1cfb560 calc_effet_loi] |
| `ADHO` | code-immediate | [tag 0x4144484f "ADHO"] [fn 0x2242ba0 f_2242ba0] |
| `BPRI` | code-immediate | [tag 0x42505249 "BPRI"] [fn 0xe49b40 f_e49b40] |
| `EMPO` | code-immediate | [tag 0x454d504f "EMPO"] [fn 0x15ee940 Cdlg_action_law___event] |
| `IMMG` | code-immediate | [tag 0x494d4d47 "IMMG"] [fn 0x15ee940 Cdlg_action_law___event] |
| `UEUR` | data-key | [data xim.gt.tsv "POP_UEUR"] |
| `NUME` | data-key | [data xim.gt.tsv "POP_NUME"] |
| `ALCO` | code-immediate | [tag 0x414c434f "ALCO"] [fn 0x22e4ad0 taux_critere] |
| `ROBO` | data-key | [data xim.gt.tsv "POP_ROBO"] |
| `TRAD` | data-key | [data xim.gt.tsv "POP_TRAD"] |
| `PRIV` | data-key | [data xim.gt.tsv "POP_PRIV"] |
| `ENFA` | code-immediate | [tag 0x454e4641 "ENFA"] [fn 0x22f8c10 f_22f8c10] |
| `LCUL` | data-key | [data xim.gt.tsv "POP_LCUL"] |
| `LAIC` | data-key | [data xim.gt.tsv "POP_LAIC"] |
| `AREN` | code-immediate | [tag 0x4152454e "AREN"] [fn 0x232ea90 reaction_yesno_sp15] |
| `SERV` | code-immediate | [tag 0x53455256 "SERV"] [fn 0x153cc10 f_153cc10] |
| `SOLV` | code-immediate | [tag 0x534f4c56 "SOLV"] [fn 0x22e4ad0 taux_critere] [tag 0xd34f4c56 "bit31-SOLV"] |
| `MREL` | code-immediate | [tag 0x4d52454c "MREL"] [fn 0x172f350 f_172f350] |

## Closing statistics

- Claim rows: 46; verdicts: corrected=10, removed=10, verified=26.
- Per-class counts: data=2, field=8, formula=8, function=9, global=1, quantity=10, source-file=1, structural=3, tag=4.
- Unique function/global RVAs in the claim table: 9 (0x1cc4440, 0x214a610, 0x215ba00, 0x215be60, 0x215d4a0, 0x219cfb0, 0x247eb60, 0x248bc10, 0x46208908).
- Unique function RVAs across the claim and per-tag tables: 35.
- Distinct values in the rva column (functions, globals, tags, quantities, fields): 25 (0x1cc4440, 0x1cd2170, 0x214a610, 0x215b3a0, 0x215ba00, 0x215be60, 0x215d4a0, 0x219cfb0, 0x224, 0x247eb60, 0x248bc10, 0x348, 0x3bd0, 0x46208908, 0x534f4c56, 0x5d859dc8, 0x620ee0, 0x6210c0, 0x6212a0, 0x621480, 0x621660, 0x78, 0x7fffffff, 0x80, 0xb88).
- Wrong addresses: 0x219cfb0 calc_perso -> 0x219cfb0 gestion_popu_warville (identity correction); 0x224 -> 0x228 (importance_action B field).
- Flagged value (not a wrong address): 0xd34f4c56 = 0x534f4c56 | 0x80000000 (bit-31-set SOLV); verified in C011/C046.
- Slot rows: 120 original; 102 non-zero tags survived; 0 non-zero tags removed; 18 zero rows removed. Verdict among non-zero tags: 102 verified, 0 corrected, 0 removed.
- Alias pairs: 18 survived, 0 removed (f_215b3a0): SCO2->SCOL, HOS2->HOSP, ARM2->ARME, CUL2->CULT, FAM2->FAMI, FEM2->FEMM, HUM2->HUMA, JUS2->JUST, LOG2->LOGE, REC2->RECH, TFO4->4SEC, SPO2->SPOR, TFO1->1DIC, TFO2->2LIB, TFO3->3CAP, TFO5->5IND, TRA2->TRAN, VIE2->VIEU. 17 `return` statements + 1 assignment for TFO5.
- Claims removed: 10 (C009, C012, C020, C026, C027, C028, C036, C040, C041, C045).
- Claims added: 3 (C031, C044, C046).
- Claims corrected: 10; claims verified: 26.

### Search commands used for the slot sweep

```text
# code immediate (per tag; BE u32 per D-009)
grep -c "0x31444943" recomp/full_fns.clean.c
# code string
grep -n "\"PRIV\"" recomp/full_fns.clean.c
# data key
grep -l "POP_ABMO" data/gt/*.tsv
```
