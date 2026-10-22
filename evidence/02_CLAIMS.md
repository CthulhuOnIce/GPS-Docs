# Volume 2 (Economy) — claims ledger

Source of record: `recomp/full_fns.clean.c` (D-001). Classes follow D-002 / D-008 /
D-010: `function`, `global`, `tag`, `quantity`, `data`, `field`; `model` marks a
construct with no executable token at all. `rva` is the old token (or `—` when the
old text had none). `verdict` is `verified`, `corrected` or `removed`.

| claim_id | old_text_verbatim | class | rva | verdict | corpus_evidence (function + line) | replacement_summary |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| C-001 | "620,000 physical buildings and industrial plants (`CBAT`)" | quantity | 0x1e460 | corrected | `f_96a610` call @ 236307 (0x1e460 = 124,000); `f_96afa0` call @ 236644 (620,000); `f_96a840` call @ 236376 (0x3c8c0 = 248,000) | The counts are literal call arguments. `CORP`, `CRPP`, `CBAT` and `DFAC` are real save-block tags written by their serializers (`f_145c5d0`, `f_9a2490`, `f_9a3d20`, `f_1463940`); "physical buildings" is the old role label, not a corpus name. |
| C-002 | "364 consolidated sectors (`DFAC`)" | global | 0x461c1f14 | corrected | `f_1395d30` call @ 2075614 uses `_DAT_461c1f14`; reset to 0x17c (380) in `f_1cbac20` @ 3345956 | `DFAC` is a real block tag (`0x44464143`, serializer `f_1463940`); 364 is the record count of the shipped 3,581,760-byte block at stride 0x2670, while the runtime sector count resets to 380, so "364 sectors" is not a code constant. |
| C-003 | "Output_i = var(slot_i + 0x58) × var(slot_i + 0x28) × Φ(unit_i)" | function | 0x9bc150 | corrected | `f_9bc150` @ 269438-269467 | Actual: `var(+0x28) * (type_match && f_9df600 ? var(+0x58) : 0)`. No general Φ factor and no exp/capacity/worker/modernity/health term. |
| C-004 | "Φ(unit_i) = 1.0 if linked map unit exists, is alive, and not under strike/siege ... 0.0 if destroyed, occupied, or severed" | function | 0x9df600 | corrected | `f_9df600` @ 290942-290964 | Actual gate: `(byte+0x50 & 0x11)==0`, activity index at `+0x4` into a 0x750-stride array, a 40-bit mask written 0x8000380000 (10 hex digits), masks 0x80000000/0x10000/0x4000000 and a byte-8 test. The listed semantics are not in the code. |
| C-005 | "total domestic production Q_dom(n,s) is summed ... Address = NationBase(n) + 0xcbc8 + s × 0x4068" | function | 0x1a33cf0 | corrected | `f_1a33cf0` @ 2932439; recursion @ 2932544-2932546 | The address formula is verified. The sum is over linked corp/building lists (`f_9bc150` per building), not over a CRPP tier; no 364-sector sum. |
| C-006 | "top-down Leontief general equilibrium ... 364 consolidated sectors" | model | — | removed | `f_1cd6220` @ 3360806 (code->index tree) and `f_1a33cf0` @ 2932439 (single-sector recursion) | No matrix read/product. Replaced by per-sector record lookup and per-sector recursion. |
| C-007 | "Intermediate Demand D_int(i,j) = a_ij × Q_target(j)" | model | — | removed | no function contains an a_ij table or a row/column product; `f_1a33cf0` reads one sector record | Removed; no such demand equation in the corpus. |
| C-008 | "β_j = min_i(Supply(i) / sum_k a_ik Q_target(k)); Q_realized = Q_target × β_j" | model | — | removed | no min-over-ratios function in the production path (`f_9bc150`, `f_1a33cf0`, `f_1409f40` contain none) | Removed; no bottleneck factor or throttling arithmetic. |
| C-009 | "`AGBL` Wheat / Cereals ... `INAC` Steel ... `INAU` Automobile" | data | — | removed | `data/gt/xae.gt.tsv` first column has 410 four-character keys (359 uppercase); `AGBL`, `INAC`, `INAU` are absent | Replaced by real keys (e.g. `EMPC`, `EENU`); the three old sector rows are not in the table. |
| C-010 | "`EENU` ... $120.00 / MWh ... RDEN × 0.5 ... YBAT + TAXP" | data | — | corrected | `data/gt/xae.gt.tsv` EENU row: price 380, booster `RDEN+RDIN+RDMA+ICME*0.2`, invest `YBAT*10+ICME*100+TKNU*10000` | Data cell values corrected to the actual row. |
| C-011 | "Fiscal policy is handled by `gps_taux.cpp` (`0x23e8e70`)" | function | 0x23e8e70 | corrected | 0x23e8e70 is `calc_incitation_etud_sup` @ 4547717; its own `gps_taux.cpp` assert is @ 4547832; the star asserts are @ 4552175 (`f_23f1c80`) and @ 4552249 (`f_23f1d40`) | 0x23e8e70 is a student-incentive function; the `gps_taux.cpp` file attribution is real but the old claim that 0x23e8e70 was the tax function is not. |
| C-012 | "`init_taxe` (`0x1c904b0`)" as tax computation | function | 0x1c904b0 | corrected | `init_taxe` @ 3318429 | Only builds localized labels (`"%d"`, text helpers); no rate arithmetic. |
| C-013 | "VAT Receipts = tau_vat × sum_s(Q_conso(s) × P_ht(s))" | model | — | corrected | `[data xan.gt.tsv "211100"]` row: VAT min 0/max 0.5/init 0.196; no VAT-specific receipt function located | The rate parameter is real and table-driven; the old VAT-specific receipt equation is still not established. |
| C-014 | "Q_conso(s,tau) = Q_conso(s,0) × (1 - epsilon_s × tau); epsilon ~ 0.4 ... ~ 1.2" | model | — | removed | no 0.4/1.2 demand-elasticity constants in the tax paths; `f_21b4680` only reads rates | Removed; no elasticity equation. |
| C-015 | "Pi_c = Revenue - Inputs - Wages - SocialCharges - Interest; delta_div ~ 0.40" | model | — | corrected | `[data xan.gt.tsv "211113"]` row: corporate tax min 0/max 0.75/init 0.34; `f_1bef780` @ 3256103 sums a per-corp aggregate over the 0x1f60 list | A corporate tax aggregate exists; the old profit-and-dividend formula and `delta_div ~ 0.40` are still not located. |
| C-016 | "W_net = W_gross × (1 - tau_urst); Total Labor Cost = W_gross × (1 + tau_urse)" | data | — | corrected | `[data xan.gt.tsv "211111"]` (N_URST) and `[data xan.gt.tsv "211112"]` (N_URSE); getters in `f_21b4680` @ 4148561-4148582 | The rates are real table parameters read from NATI+0x6fd0/+0x6fd4; the old closed-form payroll equations are not established. |
| C-017 | "Budget(S) = PopBase × K_domain × (S/(10-S+delta))^gamma; gamma ~ 1.8" | function | 0x1c388c0 | corrected | `f_1c388c0` @ 3294872-3294925: `x=x*x; f=clamp(x,0,1); f=f+f` times a finite ratio; 0.01 floor @ 3294914 | Actual factor is `2 × clamp(S^2,0,1) × ratio` with a 0.01 ratio floor; no power, gamma or PopBase/K_domain. |
| C-018 | "exponential lag smoother in `f_23f1c80` ... lambda ~ 0.05" | function | 0x23f1c80 | corrected | `get_valeur_etoile_lisse` @ 4552148 is a clamped read of `+0x67a4`; the 0.125 step is in `f_23f1d40` @ 4552228 | Replaced by the clamped read and the `(value - stored)*0.125` step clamped to ±0.25 @ 4552235-4552237. |
| C-019 | "E = (15 - MinisterHonesty) × TimeUntilElection / ElectionCycle" | function | 0x1c366b0 | corrected | `f_1c366b0` @ 3293950 has `iVar4 = 0xf - iVar4`; no election-period division in the function | Only the `15 - x` fragment is real; the election ratio is removed. |
| C-020 | "The financial system is orchestrated by the `BankManager` singleton (`0x1705700`) ... `EmissionDette` (`0x1705700`)" | function | 0x1705700 | corrected | `f_1705700` @ 2551240: allocates 0x30×0x12f and 0xc×4 tables, writes -1.0f sentinels; `EmissionDette` is only a debug string | 0x1705700 is an unnamed constructor; `BankManager` appears only in an assert string; no emission function. |
| C-021 | "`EmissionDette` generates new debt tranches in the `LOAN` pool (stride `0xec`, 20,000 slots)" | tag | 0x4c4f414e | corrected | `f_1477fd0` @ 2108728 writes tag LOAN and stride 0xec; `f_139f3c0` call @ 2065175 passes 20000; `f_1705c30` @ 2551505 allocates 0xec×20000 | The LOAN stride/count are verified; the `EmissionDette` tranche-generation attribution is not (see C-020). |
| C-022 | "R_sovereign = R_global + RiskPremium(DebtRatio) + InflationPremium(INFL) + InstabilityFactor" and the piecewise 0.01/0.08 formula | function | 0x170fd40 | corrected | `bank_updateInterestRate` @ 2558076: per-nation target = `f_128c8e0` = 0.5*(INFL-CROI), step 0.0025/0.005/0.0075/0.01 with a 0.01/0.03/0.05 dead-band | Replaced by the dead-banded step table; no risk-premium or yield-spread formula. |
| C-023 | "If DebtRatio > 1.20 ... sovereign debt spiral, credit rating downgrades" | function | 0x17074f0 | removed | `f_17074f0` @ 2552604 contains the only ratio gate: `103.0 <= (field+0xc404 / array+0xc0478)*100.0`; no 1.20 constant, no rating | Removed; only the 103% comparison exists and its label is not established. |
| C-024 | "Bi-Weekly Macro Turn: bank_step deducts Debt Service" | function | 0x1378460 | corrected | `bank_step` @ 1980293 is a 60-byte round-to-nearest helper | Replaced by the rounding helper; it does not drive turns or deduct debt service. |
| C-025 | "Gross Domestic Product (`NPIB` at `+0x621768`)" | field | 0x621768 | corrected | 0x621768 has no function/global/tag/data hit; `get_variable` NPIB case @ 4829436 writes/reads `+0x6254` (`f_13f0ed0` @ 2028928) | Field anchor corrected to `[field 0x6254 in NATI]`; the old token is retained only as `[unresolved 0x621768]`. |
| C-026 | "Inflation Rate (`INFL` at `+0xc5c0`)" | field | 0xc5c0 | corrected | `get_variable` INFL case @ 4828501 writes/reads `+0x62e0`; `f_128c8e0` reads `+0x62e0` and `+0x62ca`; CROI writes `+0x62ca` @ 4833222 | Corrected to `[field 0x62e0 in NATI]`; +0xc5c0 is a different real float not named by the tag. |
| C-027 | "Dollar Exchange Rate (`CRSS` at `+0xc594`)" | field | 0xc594 | corrected | `get_variable` CRSS case @ 4833212 uses `*(int*)(nation+0x518)*0x1e0` and `_DAT_461c1f08`; division evaluated @ 4830232-4830233 | CRSS is a ratio of two f64 table entries, not a stored field at +0xc594. |
| C-028 | "GDP = sum_{s=1}^{364}(ValueAdded(s)) + Taxes_indirect - Subsidies" | model | — | removed | `f_13f0ed0` @ 2028928 writes NPIB from a linked list of 0x910-stride records; no 364-sector sum | Removed; NPIB is a stored f32 at NATI+0x6254. |
| C-029 | "Delta INFL = theta1*(dM/M - dY/Y) + theta2*dP_energy + theta3*dW_smig" | model | — | removed | `get_variable` INFL case @ 4828501 is a straight store/read; no theta constants | Removed; no inflation equation. |
| C-030 | "CRSS(t) = CRSS(t-1) × (1 - psi × (Exports - Imports)/GDP)" | model | — | removed | CRSS branch @ 4833212 and division @ 4830232-4830233 is a two-entry f64 ratio; no psi and no trade balance | Removed; no exchange-rate equation. |
| C-031 | "No progressive/marginal income-tax computation ... no function in the corpus computes brackets" | function | 0x1ca3e10 | corrected | `f_1ca3e10` @ 3330555 reads slots 0x7078/0x707c/0x7080, blends them @ 3330612-3330647 and writes 0x6fa8 @ 3330651/3330661; `f_1bf00f0` @ 3256401 uses `powf` 7/2/0.5 @ 3256517-3256520 and forms three additive groups A/B/C @ 3256528-3256531, with the 0x799c/0x70a4 branch @ 3256524-3256525 | The income tax is real: three tranche rates are blended into an effective rate and the amount is the sum of groups A (high/middle/low shares times 4*reference*base*wh), B (reference*base*wm times (1-q7)*middle+q7*low) and C (x*base*wl*low). The exact numeric income cut-offs are not exposed; the blend and the three groups are. |
| C-032 | "The old `CRPP` block has no named user ... so its economic role is not established at all" | function | 0x1a373a0 | corrected | `f_1a373a0` @ 2934493 walks CORP 0x1f60 records into CRPP 0x1cc0 lists @ 2934526, bound 0x3c8bf (247,999), summing `f_9b4df0` @ 2934530; `f_9b4df0` @ 265850 maps CRPP+4 -> CORP and +0x6a sector @ 265858-265862; `f_15741e0` consumes it @ 2248724/2248736 | The CORP↔CRPP↔sector link is real. The old four-tier *ordering* remains unproven, but "no consumer / role not established" is false. |
| C-033 | "no function in the corpus links a record of one block to a record of the next as a production stage" | function | 0x1a33cf0 | corrected | `f_1a33cf0` @ 2932439 walks a corp's CBAT 0x68 list; `f_1a373a0` @ 2934493 walks CORP->CRPP 0x1cc0; `f_9b4df0` @ 265850 maps CRPP->CORP | CORP→CBAT and CORP→CRPP links exist. The old four-tier ordering is still unproven. |
| C-034 | "no exponential saturation production equation anywhere" / incomplete exp/pow enumeration | function | 0x15741e0 | corrected | `f_15741e0` @ 2248652 (exp @ 2248687, sqrt/0.982 @ 2248688-2248689) is called from `calc_monde_debut_tour_etapes` @ 2047786; `f_158c1f0` @ 2261789 (pow @ 2262035/2262037) is called @ 2047778 | Two economic exp/pow paths exist: the sector-capacity adjustment (`f_15741e0`) and the policy-driven compounding (`f_158c1f0`). `f_9bc150` still has no exp/pow and there is no Cap_max saturation production equation. |
| C-035 | `[data xan.gt.tsv "B_TTVA"]`, `"B_TTIS"`, `"N_URST"`, `"N_URSE"`, `"B_TISF"` | data | — | corrected | first-column keys are `211100`, `211113`, `211111`, `211112`, `211102`; the B_* / N_* strings are column-12 cells | Data citations corrected to the first-column action ids; the names are given in prose. |
| C-036 | `[qty 0x8000380000 mask]` in the f_9df600 citation | quantity | 0x8000380000 | corrected | `f_9df600` @ 290956 uses the 40-bit mask 0x8000380000 (10 hex digits); cite.py only accepts 5-8 digits | The mask is cited by function + line with an explicit width note; it is not treated as a 5-8-digit RVA. |
| C-037 | "`f_9bc150` (corpus 269438-269468)" | function | 0x9bc150 | corrected | `f_9bc150` ends at corpus 269467; the next header starts at 269469 | Range corrected to 269438-269467. |
| C-038 | "`f_1c388c0` ... 2*clamp(x^2,0,1)" without the ratio floor | function | 0x1c388c0 | corrected | `f_1c388c0` @ 3294914: if `0 < ratio < 0.01`, the ratio is set to 0.01 | The 0.01 ratio floor is part of the cost function and is now documented. |

## Closing statistics

**Unique RVAs in this ledger** (token column, excluding `—`): 22.
`0x1e460`, `0x461c1f14`, `0x9bc150`, `0x9df600`, `0x1a33cf0`, `0x23e8e70`,
`0x1c904b0`, `0x1c388c0`, `0x23f1c80`, `0x1c366b0`, `0x1705700`, `0x4c4f414e`,
`0x170fd40`, `0x17074f0`, `0x1378460`, `0x621768`, `0xc5c0`, `0xc594`,
`0x1ca3e10`, `0x1a373a0`, `0x15741e0`, `0x8000380000`.
Four of these are not RVAs: `0x621768` (removed
anchor), `0xc5c0` and `0xc594` (rejected field anchors), and `0x8000380000` (a
40-bit mask, 10 hex digits).

**Verdict counts**: `verified` 0, `corrected` 29, `removed` 9 (total 38 rows).

**Per-class counts** (the `class` column): function 18, global 1, tag 1, quantity 2,
data 4, field 3, model 9.

**Wrong addresses / anchors**:

| Old | New | Reason |
| :--- | :--- | :--- |
| `0x23e8e70` "gps_taux.cpp" | `0x23f1c80` / `0x23f1d40` | 0x23e8e70 is `calc_incitation_etud_sup`; its own `gps_taux.cpp` assert is @ 4547832; the star asserts are @ 4552175/4552249. |
| `0x1705700` "EmissionDette / BankManager" | `0x1705700` (constructor only) | No emission function; `BankManager` is an assert string. |
| `+0x621768` NPIB | `+0x6254` (`[field 0x6254 in NATI]`) | `0x621768` is unresolved; the NPIB tag writes +0x6254. |
| `+0xc5c0` INFL | `+0x62e0` (`[field 0x62e0 in NATI]`) | The INFL tag writes +0x62e0; +0xc5c0 is a different float. |
| `+0xc594` CRSS | `[tag 0x43525353 "CRSS"]` | CRSS is a ratio of two f64 table entries, not a field at +0xc594. |
| `f_1c366b0` star curve | `f_1c388c0` + `f_1c37980` | The quadratic cost lives in f_1c388c0; f_1c366b0 is the scheduling function. |
| `f_23f1c80` "exponential smoother" | `f_23f1d40` | f_23f1c80 is a clamped read; the 0.125 step is in f_23f1d40 @ 4552228. |
| `B_TTVA` / `B_TTIS` / `N_URST` / `N_URSE` / `B_TISF` as keys | first-column keys `211100` / `211113` / `211111` / `211112` / `211102` | The B_*/N_* strings are column-12 cells, not first-column keys. |

**Claims removed**: 9 rows with verdict `removed` (C-006, C-007, C-008, C-009,
C-014, C-023, C-028, C-029, C-030). The non-row sub-claims of `EmissionDette`
tranches, AAA–D ratings, yield spreads and the 1.20 spiral threshold are recorded
in the not-implemented list below.

**Claims added** (new mechanisms verified this pass):

1. `f_9bc150`: `var(+0x28) * gated var(+0x58)`.
2. `f_9df600`: activity/flag gate with the 40-bit mask and callbacks.
3. `f_1a33cf0`: linked corp/building aggregation and the
   `0xcbc8 + sector*0x4068` recursion.
4. `f_1cd6220`: red-black-tree sector-code lookup.
5. `f_1a373a0` / `f_9b4df0`: CORP→CRPP→sector link; `f_15741e0` consumer.
6. `f_15741e0`: sector-capacity adjustment with `exp`, `sqrt` and 0.982.
7. `f_158c1f0`: policy-driven per-sector compounding with two `pow` calls.
8. `f_21b4680`: runtime tax keys NTVV/NCPV/NCSV/NISV/NEXV -> NATI variable slots
   +0x6fa4/+0x6fd4/+0x6fd0/+0x6fd8 with a 100.0f cap.
9. `f_1ca3e10`: three-tranche blend from 0x7078/0x707c/0x7080 to 0x6fa8, with
   0.45/0.55, 4.0, the derived mid_share and the 0x7090/94/98 weights.
10. `f_1bf00f0`: income-tax amount with `powf` 7/2/0.5; `f_1bef3e0`/`f_1bf00b0`
    class 6/9; `f_1bef780` corporate aggregate.
11. `f_157c040`: the tax subtraction chain.
12. `f_1c388c0`: `2*clamp(S^2,0,1) * ratio` star cost with the 0.01 floor;
    `f_1c37980` flag multipliers `5k`.
13. `f_1ca0690`: star rounded to 0.1 and clamped; `f_1378420` gives 0..10.
14. `f_23f1d40`: `(value - stored)*0.125` clamped to ±0.25.
15. `f_170fd40`: dead-banded 0.0025/0.005/0.0075/0.01 interest-rate step toward
    `0.5*(INFL-CROI)`.
16. `f_17074f0`: the 103.0 ratio comparison; `f_1706fc0`: loan-entry erasure rule.
17. `f_1477fd0`/`f_139f3c0`/`f_1705c30`: the `LOAN` tag, stride 0xec, 20,000 entries.
18. `get_variable` + `f_13f0ed0`: NPIB tag -> NATI+0x6254; INFL -> +0x62e0;
    CROI -> +0x62ca; CRSS ratio of two f64 entries.
19. The block tags `CORP`/`CRPP`/`CBAT`/`DFAC` and their serializers.
20. 26 ministry keys `MFIN` ... `MSOC`; `MDEF` not `MDEFN`.

## Every claimed construct the corpus does not implement

1. the four-tier `CBAT -> CRPP -> CORP -> DFAC` *ordering* (the links exist; the
   ordering does not);
2. a `Cap_max * (1 - exp(-alpha*Workers/Workers_req)) * Modernity * HealthCoeff`
   production equation;
3. a Leontief input-output matrix, `a_ij`, intermediate demand, `beta_j`, capacity
   throttling and supply-chain cascade;
4. explicit income-tax bracket cut-offs (the code blends three rates; it does not
   expose a numeric cut-off schedule);
5. a VAT-specific receipt formula;
6. a corporate profit, depreciation and dividend-payout formula;
7. a VAT demand-elasticity equation with `epsilon ~ 0.4/1.2`;
8. a power-law star-cost curve with `gamma ~ 1.8`;
9. an exponential star smoother with `lambda ~ 0.05`;
10. `(15 - MinisterHonesty) * TimeUntilElection / ElectionCycle`;
11. `EmissionDette`, sovereign bond tranches, 5/10/30-year tenors and rollover;
12. AAA-to-D credit ratings and rating-driven downgrades;
13. yield spreads and a `RiskPremium(DebtRatio)` function;
14. `DebtRatio = Debt/GDP` and the 1.20 debt-spiral threshold;
15. a GDP value-added summation;
16. a theta-weighted inflation equation;
17. a trade-balance exchange-rate recursion;
18. the `BankManager` singleton semantics and the `LOAN` "issuance" label.

## Searches run

- `grep -c "exp(" recomp/full_fns.clean.c` = 5 (lines 2248687, 2253838, 2268076,
  2268077, 2273465). `grep -c "pow("` = 37. The two economic paths are `f_15741e0`
  (exp at 2248687) and `f_158c1f0` (pow at 2262035/2262037); `f_9bc150` and
  `f_1a33cf0` contain neither.
- `grep -c "0x434f5250"` = 15 (CORP tag), `0x43525050` = 1 (CRPP), `0x43424154` = 1
  (CBAT), `0x44464143` = 1 (DFAC); serializers `f_145c5d0`, `f_9a2490`, `f_9a3d20`,
  `f_1463940`.
- `f_1a373a0` and `f_9b4df0` are the CRPP link: `grep -n "f_9b4df0("` shows the call
  at 2934530; `f_15741e0` calls `f_1a373a0` at 2248720/2248724/2248736.
- `grep -n "f_1ca3e10("` shows the call in `f_157c040` at 2253597;
  `action_varmap.tsv` maps 211101 -> 0x6fa8 and 211300/301/302 -> 0x7078/707c/7080.
- `grep -c "B_TIMP" data/gt/xan.gt.tsv` = 0; the first-column income-tax rows are
  211101 (parent) and 211300/211301/211302 (tranches).
- `grep -c "AGBL\|INAC\|INAU" data/gt/xae.gt.tsv` = 0; those old sector keys do not
  exist.
- `grep -c "BankManager" strings_rdata.txt` = 1 (the assert text
  `BankManager::ins().dismissContract(inat);`); `EmissionDette` = 1 (debug string
  only).
- `grep -c "AAA" strings_rdata.txt` = 2, both in `testEco (AAA):` debug text; no
  rating scale or yield-spread table exists.
- `python -I tools/cite.py 621768` = unresolved in function, global, literal and
  data searches; the only unresolved token in this volume.
- `xae.gt.tsv`: 118 columns, 456 non-blank data rows, 411 non-empty first-column
  keys, of which 359 are four-character uppercase keys.
