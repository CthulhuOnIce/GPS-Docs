# Geopolitical Simulator 5 (GPS5) Engine Whitepaper
## System Architecture, Mathematical Models & Data Structures

---

### Executive Summary

**Geopolitical Simulator 5** (Eversim; build 1.18.a / PE32+ x64 executable `gps5_unpacked.exe`, 42 MB code section `.text`, 1.8 GB virtual memory space) is a hybrid macroeconomic, geopolitical, military, and sociological simulation engine. Rather than a modern object-oriented or entity-component-system (ECS) design, the engine employs a **monolithic, procedural data-oriented C/C++ architecture** operating over a massive contiguous global state database known as the **World Store** (`es_base`, inflating to ~8.48 GB in memory).

This whitepaper provides a comprehensive architectural and mathematical analysis of the game's core simulation engines:
1. **Simulation Core & Temporal Architecture**: The tick cycle, the 26-turn economic calendar ($\Delta t = 1/26$ year), the daily pipeline (`calc_monde`), and the multi-frame Etape state machine.
2. **Macro and Microeconomics**: The 364 economic sectors (`DFAC`), the micro-production tree (124,000 `CORP`, 248,000 `CRPP`, 620,000 `CBAT`), Leontief input-output matrices, the 26 ministerial budgets, the "Budget en Étoile" (Stars) non-linear curve, tax incidence, sovereign debt (`BankManager`), inflation, and international trade contracts.
3. **Military & Warfare Systems**: The 120,000 unit pool (`UNIT`, stride `0x750`), unit parameters (air, sea, land, space, strategic ballistic missiles), procurement and maintenance algorithms, spatial combat grids, damage resolution, and conflict records (`CONF`).
4. **Politics, Popularity & Social Dynamics**: The 120-theme political opinion ledger per nation, orientation tensors (`ORIE`), non-linear policy impact calculations (`importance_action`), damping kernels (`theme_delta`), the Head of State popularity equation, and the 15,000 social/pressure groups (`GROU`).
5. **Personality & Psychological Profiling**: Character simulation (`PERS`, 51,000 actors, stride `0xad0`) and psychological profiles (`PEPR`, 12,000 records, stride `0x41b8`), loyalty, corruption evaluation (`is_perso_corruptible`), affinity networks, and the interactive diplomatic/meeting negotiation tree (`gere_rdv`).
6. **The Legislative Engine & Law Execution**: How laws are stored (`SCEN`, `DEFA`, `xan.gt`), discrete bitfields vs. continuous sliders, the 5-axis ideological tensor ($TGO_1$–$TGO_5$), forecasting (`calc_effet_loi`), bicameral parliamentary vote tallying (`parlement_vote_lawpack_chambre1_estimation`), decree vs. legislative branching, and dynamic subsystem code dispatch.
7. **Demography, Healthcare & Environmental Simulation**: Cohort-component population dynamics, epidemiological spread models (`gestion_epidemies`), CO2 carbon budgeting, and climate feedback loops.

---

### Master Catalogue of Primary Simulation Records

The engine registers 58 core entity serializers (`schema/types.tsv`). The entire world state is persisted either via raw chunk dumps (mode 5/6) or tagged delta streams (mode 1). Below are the primary entity records governing simulation logic:

| FourCC Tag | Serializer RVA | Record Size (bytes) | Typical Instance Count | Memory Base Pointer / Global | Semantic Role in Simulation |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `NATI` | `0x147d030` | `0x645310` (6.57 MB) | 300 nations | `*0x46208908` | Complete national state (macro indicators, 120-theme ledger, 364 sector sub-records at `+0xcbc8`, legal flags) |
| `DFAC` | `0x1463940` | `0x2670` (9.84 KB) | 364 activities | `G_4bbb8880` | Economic sector archetype definitions (energy, industry, raw materials, agriculture, 26 ministries) |
| `CORP` | `0x145c5d0` | `0x1f60` (8.03 KB) | ~124,000 companies | Pointer in `.data` | Corporate entities, balance sheets, parent-subsidiary trees, market capitalization, headcount |
| `CRPP` | `0x9a2490` | `0x1cc0` (7.36 KB) | ~248,000 plants | Dynamic pool | Company production installations and geographic output allocations across 300 nations |
| `CBAT` | `0x9a3d20` | `0x68` (104 bytes) | ~620,000 buildings | Dynamic pool | Individual physical factories, power plants, farms, offices; linked to spatial map units |
| `UNIT` | `0x14a96e0` | `0x750` (1.87 KB) | ~120,000 units | `G_61d0deb0` | Military formations, ships, aircraft, satellites, silos, city fortifications; tactical battle stats |
| `PERS` | `0x1493660` | `0xad0` (2.77 KB) | ~51,000 actors | `*0x5d85ada0` | Characters: Heads of State, ministers, deputies, party leaders, generals, union bosses, billionaires |
| `PEPR` | `0x1496bb0` | `0x41b8` (16.82 KB)| ~12,000 profiles | Dynamic pool | Detailed psychological and behavioral matrices, historical relationships, policy stances |
| `GROU` | `0x146e300` | `0xb88` (2.95 KB) | ~15,000 groups | `*0x5d859dc8` | Social groups, political parties, labour unions, professional lobbies, demographic factions |
| `ORIE` | `0x1492480` | `0xac` (172 bytes) | ~153,000 entries | `G_1cd474b0` | Ideological position coordinates across policy dimensions |
| `SCEN` | `0x14a3080` | `0xb8` (184 bytes) | ~90,000 proposals | `G_5f9830d0` | Legislative proposals, active bills, scenario conditions, lawpack containers |
| `DEFA` | `0x1461ca0` | `0x3b0` (944 bytes) | ~2,570 actions | `G_47938840` | Master action catalogue (statutory laws, presidential decrees, executive actions) |
| `CONF` | `0x14585e0` | `0x12d0` (4.82 KB) | ~2,000 conflicts | `&DAT_433f40e0` | Wars, military alliances, mutual defense treaties, sanctions, ceasefire treaties |
| `TROU` | `0x14a7a40` | `0x348` (840 bytes) | ~6,000 troubles | `G_60b65f30` | Civil unrest, strikes, riots, insurgencies, terrorist campaigns, martial law states |
| `LOAN` | `0x1477fd0` | `0xec` (236 bytes) | ~20,000 loans | Linked via Bank | Sovereign and corporate debt contracts, maturity schedules, coupons, creditor links |
| `REGI` | `0x149d6b0` | `0x910` (2.32 KB) | 1,309 regions | `G_47e23350` | Sub-national administrative regions (provinces, states, oblasts), regional infrastructure |

---

### Global Variable Storage (`variable_store`)

In addition to dedicated entity structures, the engine maintains an indexed scalar store for rapid mathematical evaluation:
- **Location**: Heap block allocated via `variable_store_get()` (`0x8b9490` $\to$ base `0x32d1920`, total size `0x40d9908` bytes / ~68 MB).
- **Format**: An array of 4,000,000 fixed slots:
  ```cpp
  struct VariableSlot {
      uint32_t id;         // Variable Unique Identifier (1 .. 3,999,999)
      uint32_t flags;      // Status / evolution bitmask (e.g. 0x2000 = alias chain)
      double   value;      // Current IEEE 754 64-bit floating point value
  };
  uint8_t used_bitmap[4000000]; // 1 byte per slot indicating active allocation
  ```
- **Access Pipeline**:
  - Direct read: `f_1cc8ac0(p)` reads current value for today.
  - Historical / Time-stepped read: `f_1cca540(p, day)` traverses the evolution record table at `G_524012e0` (stride `0x38`) to interpolate historical values or evaluate dynamic expressions.
  - UI Notification: When monitored variables cross thresholds, `f_1cc8d90` dispatches Windows messages `0x4d`..`0x51` directly to the `Cgps` module message pump.

---

### Whitepaper Structure

This report is organized into seven modular volumes located within `_whitepaper/`:
- `01_SIMULATION_CORE.md`: Pipeline architecture, frame splitting, time dilation, and database synchronization.
- `02_ECONOMY.md`: Micro-macro economic engine, production trees, Leontief matrix, public finances, star budgets, debt markets, and trade contracts.
- `03_MILITARY.md`: Unit specifications, combat formulas, procurement, maintenance costs, and geopolitical warfare.
- `04_POLITICS_AND_POPULARITY.md`: Political ledger, opinion dynamics, damping kernels, popularity formulation, and interest groups.
- `05_PERSONALITY_AND_CHARACTERS.md`: Personality psychology, corruption checks, diplomatic negotiations, and cabinet management.
- `06_LAWS_AND_LEGISLATION.md`: Complete legal architecture: storage, UI editing, parliamentary vote calculations, decree execution, and dynamic subsystem dispatch.
- `07_DEMOGRAPHY_AND_ENVIRONMENT.md`: Cohort-survival population mechanics, epidemics, environmental systems, and carbon accounting.
