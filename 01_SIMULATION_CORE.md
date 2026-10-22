# Volume 1: Simulation Core & Temporal Engine

This volume is validated against `recomp/full_fns.clean.c` (the corpus of record, per D-001). Every structural or numeric claim below carries a D-002 citation: `[fn 0x... name]` for a function header, `[global 0x... type]` for a `// GLOBALS` entry, `[qty 0x... role]` for a stride/size/count literal, and `[tag 0x... "TAG"]` for a four-character immediate. Runtime artifacts (`calc_monde_steps.txt`, `dayprof.json`, `etapes_*.tsv`) are marked **observed, not decompiled** and are used only as corroboration. Anything the corpus does not contain is either deleted or named in a "What the code does not establish" subsection.

---

## 1. Temporal Architecture & Clocks

The corpus contains three distinct time representations, not three independent clocks: a tick counter read by a timer function, a game-hour accumulator that builds a millisecond timestamp, and a separate macro-turn day counter. The data flows through code; it is not a free-running multi-clock scheduler.

```mermaid
flowchart TD
    T["f_25b6690: ticks / rate -> seconds"] --> S["f_1c37d70: seconds -> game hours"]
    G1["global 70ee6528 ticks"] --> T
    G2["global 70ee6524 rate"] --> T
    S --> H["global 464ddc98 whole game hours"]
    S --> F["global 464ddc9c fractional hours"]
    H --> D["global 4813c0a8 millisecond timestamp"]
    D --> Y["global 46494828 year"]
    D --> M["global 46494824 month"]
    D --> O["global 46494810 day-of-year"]
    D --> N["global 464916a8 day serial"]
    MC["global 50ef4788 macro day"] --> CT["calc_monde_fin_tour"]
    N --> CT
    CT -->|"bank_step(clock + increment)"| MC
```

### 1.1 The timer function

`f_25b6690` [fn 0x25b6690 f_25b6690] returns a `float`. When the smoothing coefficient at [global 0x70ee6548 f64] is non-zero it computes a smoothed tick estimate from [global 0x70ee6528 f64], [global 0x70ee6524 i32], [global 0x70ee651c f32], [global 0x70ee6540 f64] and [global 0x70ee6530 f64], clamps the advance, and divides by the rate. Only the `else` branch is the simple `ticks / rate` form:

```c
if (_DAT_70ee6548 != 0.0) {
    d = ((double)_DAT_70ee651c + _DAT_70ee6540) * _DAT_70ee6548
        + (1.0 - _DAT_70ee6548) * _DAT_70ee6528;
    if ((double)_DAT_70ee6524 * _DAT_70ee6530 < d - _DAT_70ee6528)
        d = _DAT_70ee6528 - (double)_DAT_70ee6524 * _DAT_70ee6530;
    return (float)(d / (double)_DAT_70ee6524);
}
return (float)(_DAT_70ee6528 / (double)_DAT_70ee6524);
```

So the old volume's "Elapsed Seconds = Ticks / Rate" is the fallback path, not the whole function [fn 0x25b6690 f_25b6690]. The smoothing constants are read but their physical meaning is not named by the corpus.

### 1.2 Game hours and the date object

`f_1c37d70` [fn 0x1c37d70 f_1c37d70] takes `mutex_scope(0xf,1)` and, in its normal branch, computes game hours as

```
hours = ((timer_seconds - _DAT_464ddc8c) - _DAT_46054ecc) / (_DAT_464ddc88 / _DAT_322a4f4)
```

where the dividend uses [global 0x464ddc8c f32] and [global 0x46054ecc f32], and the divisor is the ratio of [global 0x464ddc88 f32] to [global 0x322a4f4 f32]. The code does not name the units of that ratio; the "speed multiplier" and "time base" labels are interpretive, though the ratio form is real.

The returned hours are split into a fractional part at [global 0x464ddc9c f32] and a whole-hour accumulator at [global 0x464ddc98 f32] (the `floorf` of the accumulated fraction is added to the whole-hour value). The integer cast `(int)_DAT_464ddc98` is then passed to `f_25a7630`, which builds `((hours * 60 + minutes) * 60 + seconds) * 1000`, and `f_25a7940` writes that millisecond value into [global 0x4813c0a8 i64] [fn 0x25a7630 f_25a7630] [fn 0x25a7940 f_25a7940].

The eight-byte object at [global 0x4813c0a8 i64] is therefore a millisecond timestamp, not a `COleDateTime`: `f_25cafe0` [fn 0x25cafe0 f_25cafe0] returns `(int)floor(ms / 86400000.0)` and `f_25dd540` [fn 0x25dd540 f_25dd540] returns `(ms % 86400000) / 3600000`. The code never constructs an OLE Automation date and never uses a Julian epoch.

From that timestamp `f_1c37d70` derives:
- [global 0x46494828 u32] = year, via `f_25e9b60` [fn 0x25e9b60 f_25e9b60];
- [global 0x46494824 i32] = month, via `f_25df0d0` [fn 0x25df0d0 f_25df0d0];
- [global 0x464916a8 i32] = day serial, via `f_25cafe0`; this is `floor(ms / 86400000)`, not a Julian day counter;
- [global 0x46494810 i32] = day-of-year, computed as `(_DAT_464916a8 - day_serial(Jan 1 of year)) + 1`.

[global 0x4649165c i32] gates many branches (`== 0` in the common path). The corpus compares it against 0 and against the negative values -1, -2, -3, -4, -6, -7, -8, -9 and -10 in different functions, but it does not define what `-6` means. The old volume's "`< -6` represents menus/paused state" and "`0` represents active simulation" are therefore not established; only "the value is used as a mode gate, with 0 as the normal-path value" is supported.

### 1.3 The macro-turn clock

The macro-turn clock is the `double` at [global 0x50ef4788 f64]. It is advanced by exactly one non-initialising assignment in the corpus, inside `calc_monde_fin_tour` [fn 0x140e5d0 calc_monde_fin_tour], at corpus line 2050107:

```
dVar55 = ((double)(iVar40 - iVar14) / 365.0) * 0.038461538461538464 * 365.25
         + _DAT_50ef4788;
...
iVar40 = bank_step(dVar55);
if (_DAT_464916a8 == iVar40 + 4) {
    DAT_4649484b = '\0';
    _DAT_50ef4788 = dVar55;
}
```

`iVar40` and `iVar14` are the day serials of 1 January of the next and current year, so `iVar40 - iVar14` is the number of days in the current year. The constant `0.038461538461538464` is exactly 1/26. The increment is therefore

```
(days_in_year / 365.0) * (1/26) * 365.25
```

For a 365-day year that is 14.048076923… days; for a 366-day year it is 14.086564805… days [fn 0x140e5d0 calc_monde_fin_tour]. The old volume's "14.03846 days" equals 365/26 and does not match the code.

`bank_step` is [fn 0x1378460 bank_step], not `f_140e000`. It only rounds a `double` to an `int`: for a non-negative input it returns `trunc(x)` unless the fractional part is at least 0.5, in which case it returns `trunc(x) + 1`; for a negative input it decrements when the fractional part is below -0.5. It contains no catch-up loop and no comparison against `today`.

The turn-advance comparisons are in `calc_monde_fin_tour`, not in `bank_step`: the corpus compares the current day serial [global 0x464916a8 i32] against `bank_step(dVar55) + 1`, `+2`, `+3` and `+4` in different branches. There is no single `today - day(clock + Δt + 5) > 0` inequality. The initialisation of the clock is also elsewhere: `f_239f3b0` [fn 0x239f3b0 f_239f3b0] sets [global 0x50ef4788 f64] to the current day serial when it is more than 100 days behind (corpus line 4494244).

### 1.4 Pseudocode (temporal core)

```pseudocode
function timer_seconds():                         # f_25b6690
    smoothing     = global 70ee6548               # 0.0 disables smoothing
    last_tick     = global 70ee6528
    rate          = global 70ee6524
    max_step      = global 70ee6530
    raw_fraction  = global 70ee651c
    correction    = global 70ee6540

    if smoothing is not zero:
        estimate = (raw_fraction + correction) * smoothing
                   + (1 - smoothing) * last_tick
        if rate * max_step < estimate - last_tick:
            estimate = last_tick - rate * max_step
        return estimate / rate
    return last_tick / rate


function update_game_clock():                     # f_1c37d70
    lock the clock mutex (id 0xf, count 1)
    status = 0

    if clock is not frozen:                       # freeze flag: global 46209e1c
        if resync_requested:                      # global 464ddc84
            base_offset = (timer_seconds() - now_reference) - speed * 24.0
            status = -1

        if pending_step < 1:                      # global 46495ec4
            if speed == 0 or (decoder_flags + 2) & 0xfffffffb == 0:
                base_offset = timer_seconds() - now_reference
            else:
                if pending_step == -1:
                    pending_step = 0
                    base_offset = timer_seconds() - now_reference

                if mode == 0 and sub_mode == 0:   # globals 4649165c, 4649166c
                    elapsed = (timer_seconds() - base_offset - now_reference)
                              / (speed / speed_base)
                else:
                    elapsed = (-base_offset - now_reference)
                              / (speed / speed_base)

                hours = elapsed
                if mode == 0:
                    if base_offset > 1.0:
                        base_offset = 1.0
                    hours = min(elapsed, 1.0)

                anchor = base_offset              # captured after the 1.0 clamp
                advance = max(hours, 0.0)
                fraction = fraction + advance
                whole = floor(fraction)
                if whole != 0:
                    fraction = fraction - whole
                    day_hours = floor((whole_hours + whole) / 24.0)
                    if day_hours > last_day_hours
                       and (clock_flag != 0
                            or day_hours > saved_day_hours + 1
                            or budget_check() == 0)
                       and mode == 0:
                        base_offset = timer_seconds() - now_reference
                                      - (speed / speed_base) * fraction
                        fraction = 1.0
                        whole = 0                 # re-anchor suppresses the whole-hour add
                    whole_hours = whole_hours + whole
        else:                                     # pending_step >= 1: catch up
            base_offset = timer_seconds() - now_reference - pending_step
            anchor = base_offset
            advance = max(pending_step,
                          floor(timer_seconds() - base_offset - now_reference))
            whole_hours = whole_hours + advance
            pending_step = -1
            status = 1

        if mode == 0:
            base_offset = timer_seconds() - base_offset
        else:
            base_offset = -anchor
        if mirror_flag != 0:                      # global 46209e20
            saved_whole_hours = whole_hours
            saved_fraction = fraction
    else:
        whole_hours = saved_whole_hours
        fraction = saved_fraction

    hours_value = whole_hours as an integer
    timestamp = build_millisecond_timestamp(hours_value)   # hours -> minutes -> seconds -> ms
    write timestamp into the 8-byte date object 4813c0a8   # millisecond timestamp

    year          = year_of(timestamp)
    month         = month_of(timestamp)
    day_of_month  = day_field_of(timestamp)      # the field the February test reads
    hour          = hour_of(timestamp)           # (ms % 86,400,000) / 3,600,000
    aux_field     = f_25da840(timestamp)         # another field of the same date split
    previous_day  = current_day_serial
    current_day   = day_serial_of(timestamp)     # floor(ms / 86,400,000)
    day_of_year   = current_day - day_serial_of(1 January of year) + 1
    last_day_hours = day_serial_of(date at whole_hours)

    if current_day changed and month == 2 and day_of_month == 1 and nations > 1:
        for each active nation i (1 .. nations-1):
            for each active nation j (1 .. nations-1):
                clear mask 0xfffeffffffffffff at (i, j)
                if mask 0x800000000000 is set at (i, j):
                    clear mask 0xffff7fffffffffff at (i, j)
                    set   mask 0x1000000000000  at (i, j)

    days_this_year = day_serial_of(1 January year+1) - day_serial_of(1 January year)
    days_last_year = day_serial_of(1 January year)   - day_serial_of(1 January year-1)
    week_of_epoch  = (year - epoch_year) % 0x1d + 1        # 0x1d = 29

    clock_value = saved_mirror_value
    if clock is not frozen:
        clock_value = fraction + whole_hours
    minutes = (clock_value - whole_hours) * 60.0 as an integer
    minutes = clamp(minutes, 0, 0x3b)                       # 0x3b = 59
    unlock the clock mutex
    return status


function advance_macro_day_clock(turn_state):     # calc_monde_fin_tour
    # (not read: the other 552 corpus lines of calc_monde_fin_tour, 2048643-2051194)
    if mode == 0 and calendar_flag == 0:          # globals 4649165c, 463d1a7c
        next_year_start = 1 January of (year of macro_day + 1)
        this_year_start = 1 January of (year of macro_day)
        days_in_year = day_serial_of(next_year_start) - day_serial_of(this_year_start)

        step = (days_in_year / 365.0) * (1 / 26) * 365.25 + macro_day
        # 0.038461538461538464 is exactly 1/26 (corpus 2050103); 365.25 is the mean year

        rounded = bank_step(step)                 # nearest-integer rounding helper
        if today == rounded + 3:
            new_year_flag = 1
        if today == rounded + 4:
            new_year_flag = 0
            macro_day = step
```
> Evidence: timer_seconds [fn 0x25b6690], update_game_clock [fn 0x1c37d70], advance_macro_day_clock [fn 0x140e5d0 calc_monde_fin_tour], corpus 4847341-4847356, 3294553-3294761, 2050082-2050107; 0.038461538461538464 is exactly 1/26 at corpus 2050103.

**What the code does not establish.** The corpus does not name a class for [global 0x4813c0a8 i64], does not state the epoch of the day serial, does not give units for the ratio of [global 0x464ddc88 f32] to [global 0x322a4f4 f32], does not define the meaning of `4649165c < -6`, and does not explain why the macro increment uses 1/26. Those are stated as not established rather than filled in from domain knowledge.

---

## 2. The Daily Execution Pipeline (`calc_monde`)

`calc_monde` is the function at [fn 0x13f3290 calc_monde] (mangled `_geolgame__Cgps__calc_monde`), 93,356 bytes and 15,853 corpus lines. It takes `mutex_scope(3,1)` at corpus line 2030531 and logs the literal `"Attend mutex IDME_cmt_permanent"` at line 2030787; the log string is present, and the lock call is the `mutex_scope` at that site [fn 0x13f3290 calc_monde].

The old volume's "36 sequential phases" is removed. Counting the profiler markers in the body gives one `prof_begin` call [fn 0x12bb2b0 prof_begin] at line 2030705 and 56 `prof_end` calls [fn 0x12d04b0 prof_end]. `prof_end` is a three-byte `return;` function with no label argument; `prof_begin` takes and returns one opaque argument. The corpus contains no phase-index constant and no profiler label string. The label sequence in `calc_monde_steps.txt` is a runtime observation from the emulator harness and is tagged **observed, not decompiled**.

The ordered top-level work listed below is corroborated by that observation, but each row is anchored to a direct call that is present in `calc_monde`:

```mermaid
flowchart TD
    A["calc_monde: mutex_scope(3,1)"] --> B["f_1c37d70: update date"]
    B --> C["prof_begin (1 call)"]
    C --> D["maj_sat -> maj_chemin_nation -> maj_agenda"]
    D --> E["gere_requetes -> gere_rdv -> gere_histo"]
    E --> F["calc_actions -> calc_unites"]
    F --> G["org_adh_loop -> reactions_process"]
    G --> H["f_1053e40 -> calc_monde_debut_tour_etapes"]
    H --> I["gereAccordMainOeuvreLissee if stage < 0"]
    I --> J["EmissionDette -> generateDebts -> updateInterestRate"]
    J --> K["calcCorpsDebutTrimestre -> bank_step -> calc_nation"]
    K --> L["calc_monde_fin_tour -> org_adh_demand"]
    L --> M["... daily subsystems ..."]
    M --> N["56 x prof_end"]
```

| Observed label (runtime) | Direct callee verified in `calc_monde` |
| --- | --- |
| `maj_sat` | [fn 0x1a3aa70 maj_sat] |
| `maj_chemin` | [fn 0x18c0970 maj_chemin_nation] |
| `maj_agenda` | [fn 0x15a7be0 maj_agenda] |
| `gere_requetes` | [fn 0x1d087a0 gere_requetes] |
| `gere_rdv` | [fn 0x15a5e20 gere_rdv] |
| `gere_histo` | [fn 0x1e2eef0 gere_histo] |
| `calc_actions` | [fn 0x1cfad00 calc_actions] |
| `calc_unites` | [fn 0x24c5200 calc_unites] |
| `org_adh_loop` | [fn 0x2128f80 org_adh_loop] |
| `trigger` | [fn 0x22c3590 reactions_process] (the corpus name is `reactions_process`) |
| `plansociaux` | [fn 0x1053e40 f_1053e40] (38-byte helper; see below) |
| `calc_monde_debut_tour` | [fn 0x1409f40 calc_monde_debut_tour_etapes] |
| `gereAccordMainOeuvreLissee` | [fn 0x1c7d720 gereAccordMainOeuvreLissee] |
| `EmissionDette` | [fn 0x1705700 f_1705700] |
| `Bank generateDebts` | [fn 0x17101f0 f_17101f0] |
| `updateInterestRate` | [fn 0x170fd40 bank_updateInterestRate] |
| `calcCorpsDebutTrimestre` | [fn 0x980cc0 calcCorpsDebutTrimestre] |
| `bank` | [fn 0x1378460 bank_step] (rounding helper; not the whole bank system) |
| `calc_nation` | [fn 0x14134a0 calc_nation] |
| `calc_monde_fin_tour` | [fn 0x140e5d0 calc_monde_fin_tour] |
| `org_adh_demand` | [fn 0x2128b00 org_adh_demand] |
| `calc_PersoPrinc_evol_effectifs_cacher` | [fn 0x218eaf0 calc_PersoPrinc_evol_effectifs_cacher] |
| `ordi_strategie_cyber` | [fn 0x20edc60 ordi_strategie_cyber] |
| `calc_astronautes` | [fn 0x893170 calc_astronautes] |
| `calc_programmes_spatiaux` | [fn 0x8939f0 calc_programmes_spatiaux] |
| `stat_popularite_add` | [fn 0x1cc4440 stat_popularite_add] |
| `maj_simu_lan_clients` | [fn 0x1f5b9b0 maj_simu_lan_clients] |
| `calc_monde_annee` | [fn 0x1c38a50 calc_monde_annee] |
| `ia_programmes_spatiaux` | [fn 0x8ac300 ia_programmes_spatiaux] |
| `gestion_epidemies` | [fn 0xd53a40 gestion_epidemies] |
| `calcBourseJour` | [fn 0x910ec0 calcBourseJour] |
| `calcCorpsJour` | [fn 0x980f70 calcCorpsJour] |
| `majClassementConsortiums` | [fn 0x9e17c0 majClassementConsortiums] |
| `majClassementMilliardaires` | [fn 0x9e1950 majClassementMilliardaires] |

Two rows of the old table need explicit correction:

- The old `plansociaux` row cited [fn 0x1053e40 f_1053e40] and called it the corporate-redundancy system. The body is 38 bytes and returns `(int)((_DAT_1c168090 / f_104d590()) * 212.0)`; it is not a redundancy solver. The runtime label is kept as an observation, but the "corporate redundancies, plant closures, severance" description is removed.
- The old `classer_joueur` row cited [fn 0x959710 f_959710]. That function is a three-byte `return;` no-op. The leaderboard description is removed; the address is not the described system.

`calc_monde` performs no "36-step" state machine itself. It calls the stage driver [fn 0x1409f40 calc_monde_debut_tour_etapes] and other subsystems in source order. The old 36-row phase table is replaced by the verified-call table above.

### 2.1 Pseudocode (daily driver)

```pseudocode
function calc_monde(...):                          # 0x13f3290
    take the permanent-world mutex (id 3, count 1)      # corpus 2030531
    log "Attend mutex IDME_cmt_permanent"               # corpus 2030787

    update the game clock and calendar (update_game_clock)   # corpus 2030564

    start the profiler once (prof_begin)                    # corpus 2030705

    # the order below is the runtime label order; it is observed, not decompiled
    refresh satellites, nation routes, agenda
    handle requests, appointments, history
    run actions, then units
    run the organisation loop, then process reactions
    run f_1053e40 (38-byte helper)
    stage_index = run_stage_frame(global 324da9c)           # corpus 2031956-2031957
    store stage_index back into global 324da9c
    if stage_index as a signed integer < 0:
        run the smoothed labour-agreement pass
    # (not read: setup before the first temporal call, and the body between the listed calls)
    ... the remaining daily subsystems ...
    turn_state = calc_monde_fin_tour(turn_state)            # corpus 2033073
    store the result back into global 324daa0
    call prof_end 56 times in total
```
> Evidence: calc_monde [fn 0x13f3290], corpus 2029428-2045280 (header and ordered call sites read, not the full 93,356-byte body); calc_monde_fin_tour call at 2033073, one prof_begin at 2030705, 56 prof_end.

**What the code does not establish.** The corpus does not define "phase", does not carry the `calc_monde_steps.txt` label strings, and does not state that any named subsystem is the "heaviest". The table's descriptions beyond the callee names are not in the corpus.

---

## 3. The Multi-Frame Stage Driver (`calc_monde_debut_tour_etapes`)

The driver is [fn 0x1409f40 calc_monde_debut_tour_etapes], 16,575 bytes. It takes a single `uint param_1`, and `calc_monde` passes the global stage index [global 0x324da9c u32] into it and writes the return value back (corpus 2031956-2031957). The driver returns the unsigned value -1 (all 32 bits set) on completion (corpus line 2048638; body-relative line 3358); `calc_monde` tests `(int)_DAT_324da9c < 0` and then runs the post-turn code.

The old "20 Etapes" count is corrected by counting the source labels in the body. The body contains exactly 19 `/* label "EtapeN" */` markers: Etape1 through Etape9 and Etape11 through Etape20. There is no Etape10 marker; the gap is between corpus lines 2047134 (`Etape9`) and 2047287 (`Etape11`). The labels are the counting method: `grep 'label "Etape'` on the function body returns 19 matches [fn 0x1409f40 calc_monde_debut_tour_etapes]. The driver's early work is additionally gated by `if ((int)param_1 < N)` for N = 1..7. These are early-work gates, not additional numbered Etapes: `< 1` (2045628-2045773) and `< 2` (2045775-2045795) run before the Etape1 label at 2045796; `< 3` (2045867) is inside the Etape1 block; `< 4..< 7` (2046284, 2046334, 2046368, 2046396) are inside the Etape2 block. The Etape1 label is outside both early gates.

The `< 7` gate at corpus 2046396 is the two-conjunct form `((int)param_1 < 7) && (bVar42 == 0)`, where `bVar42` is `_DAT_4649165c != 0` (set at 2045459, stored at 2045460, reloaded at 2046210). The restyled pseudocode and mermaid G7 therefore state the same gate as `stage < 7` together with `4649165c == 0`; the decompiler spelling is quoted here rather than in the pseudocode, per D-016.

```mermaid
flowchart TD
    CM["calc_monde"] -->|"passes the stage index (global 324da9c)"| DRV["calc_monde_debut_tour_etapes"]
    DRV --> G0{"stage == 0?"}
    G0 -->|yes| SETUP["set the two Debut/ECO flags; label Debut"]
    G0 -->|no| G1{"stage < 1"}
    SETUP --> G1
    G1 -->|yes| D1["early work: NOUVEAU JOUR logging, f_140e000, f_1a252a0, nation loop"]
    G1 -->|no| G2{"stage < 2"}
    G2 -->|yes| D2["early work: f_1cc9370, f_1cc8d90"]
    G2 -->|no| L1["label Etape1 (2045796)"]
    D1 --> G2
    D2 --> L1
    L1 --> G3{"stage < 3"}
    G3 -->|yes| D3["f_1a4e8d0, f_1a50500, f_1a4d9f0"]
    G3 -->|no| L2["label Etape2 (2046212)"]
    D3 -->|"return value 0 and flag 463d1b41 == 0"| RET["early return 2, skips Etapes 2-19 (2045876-2045879 -> 2048636)"]
    D3 --> L2
    L2 --> G4{"stage < 4"}
    G4 -->|yes| D4["f_159bfe0 + person loop, stride 0xad0"]
    G4 -->|no| G5{"stage < 5"}
    G5 -->|yes| D5["f_1c386d0"]
    G5 -->|no| G6{"stage < 6"}
    G6 -->|yes| D6["f_1c408f0"]
    G6 -->|no| G7{"stage < 7 and 4649165c == 0?"}
    G7 -->|yes| D7["clock/timing only"]
    G7 -->|no| L3["label Etape3 (2046427)"]
    D4 --> L3
    D5 --> L3
    D6 --> L3
    D7 --> L3
    L3 --> L4["Etape4"]
    L4 --> L5["Etape5"]
    L5 --> L6["Etape6"]
    L6 --> L7["Etape7"]
    L7 --> L8["Etape8"]
    L8 --> L9["Etape9"]
    L9 --> L11["Etape11 (no Etape10 label)"]
    L11 --> L12["Etape12"]
    L12 --> L13["Etape13"]
    L13 --> L14["Etape14"]
    L14 --> L15["Etape15"]
    L15 --> L16["Etape16"]
    L16 --> L17["Etape17"]
    L17 --> L18["Etape18"]
    L18 --> L19["Etape19"]
    L19 --> L20["Etape20"]
    L20 -->|"return -1 as unsigned"| POST["calc_monde post-turn"]
    RET --> POST
```

| Label (corpus line) | Direct callees verified in that block |
| --- | --- |
| Etape1 (2045796) | [fn 0x1a4e8d0 f_1a4e8d0], [fn 0x1a33cf0 f_1a33cf0], [fn 0x255f6b0 f_255f6b0], [fn 0x1a3aac0 f_1a3aac0] |
| Etape2 (2046212) | [fn 0x21bf540 f_21bf540], [fn 0x10f2060 f_10f2060], [fn 0x159bfe0 f_159bfe0], [fn 0x21428c0 f_21428c0] |
| Etape3 (2046427) | [fn 0x193af90 f_193af90] |
| Etape4 (2046505) | [fn 0x157fb20 f_157fb20], [fn 0x2118290 f_2118290] |
| Etape5 (2046658) | [fn 0x18e77a0 f_18e77a0], [fn 0x155cc10 f_155cc10], [fn 0x153c510 f_153c510], [fn 0x1557510 f_1557510] |
| Etape6 (2046792) | [fn 0x18e7550 f_18e7550] |
| Etape7 (2046903) | [fn 0xedcf60 f_edcf60], [fn 0xededa0 f_ededa0] |
| Etape8 (2047008) | [fn 0x1558b00 f_1558b00], [fn 0x1558620 f_1558620] |
| Etape9 (2047134) | [fn 0x9de510 f_9de510], [fn 0x15427b0 f_15427b0] |
| Etape11 (2047287) | [fn 0x1bea500 f_1bea500], [fn 0x1c3c060 f_1c3c060] |
| Etape12 (2047413) | [fn 0x9df3b0 f_9df3b0], [fn 0x15445f0 f_15445f0], [fn 0x9ebf20 f_9ebf20], [fn 0x9eac60 f_9eac60] |
| Etape13 (2047696) | [fn 0x158c1f0 f_158c1f0], [fn 0x1ca2e50 f_1ca2e50], [fn 0x157ddb0 f_157ddb0], [fn 0x15741e0 f_15741e0] |
| Etape14 (2047812) | [fn 0x19a4be0 f_19a4be0] |
| Etape15 (2047883) | [fn 0x9800d0 f_9800d0] |
| Etape16 (2047957) | [fn 0x981a00 f_981a00] |
| Etape17 (2048027) | [fn 0x25276f0 f_25276f0] |
| Etape18 (2048108) | [fn 0x8732b0 f_8732b0], [fn 0x25a7610 f_25a7610], [fn 0x25df0d0 f_25df0d0], [fn 0x25e9b60 f_25e9b60] |
| Etape19 (2048496) | [fn 0x24cd530 f_24cd530] |
| Etape20 (2048568) | [fn 0x12bd280 f_12bd280] (three-byte function) |

The per-Etape kernel addresses that survive are listed in the table above; the old descriptions are not verified. Specific corrections:

- Etape1's "heaviest step, iterates all companies × buildings" is removed; the block calls [fn 0x1a4e8d0 f_1a4e8d0] and [fn 0x1a33cf0 f_1a33cf0] among others, but the corpus does not label it "heaviest".
- Etape3's "12,000 records of stride [qty 0x41b8 stride]" is corrected: the stride literal is real at corpus lines 2046292 and 2046501, but the count 12,000 is not present at that site. The base is [global 0x1e660190 ulonglong].
- Etape5's "Leontief input-output matrix" is removed; the block calls [fn 0x18e77a0 f_18e77a0] and [fn 0x155cc10 f_155cc10], but no matrix arithmetic is named by the corpus.
- Etape12's "124,000 companies" is removed from this row; the block calls the corporate functions, but the 124,000 bound is not used at that site.
- Etape20's "increments turn counter; resets `_DAT_324da9c` to 0" is corrected: the driver returns the unsigned value -1, and the caller handles the negative value. `f_12bd280` is a three-byte function.

### 3.1 Pseudocode (stage driver)

```pseudocode
function run_stage_frame(stage):                   # calc_monde_debut_tour_etapes, 0x1409f40
    mode_value = global 4649165c
    mode_active = (mode_value != 0)
    f_2793510()
    # label "calc_monde_debut_tour"
    if stage == 0:
        flag_326caa01 = 1
        flag_326caa02 = 1
    # label "Debut"
    # (not read: setup and logging before the stage < 1 gate)

    # early-work gate, NOT Etape1
    if stage < 1:
        if mode_value == 0:
            # (not read: logging block 2045629-2045663)
        f_140e000()
        if new_year_flag == 0 and stage < 1:
            f_1a252a0(nation_root)
        if mode_value < -1:
            # nation loop: f_9ebf20, f_9eac60, f_9defe0, f_9de5d0,
            # f_1a38620, f_1a36d10, f_9b4df0, f_977360, f_9e8fb0
            # (not read: full loop)
        # (not read: clock/timing guard 2045767-2045772)

    # early-work gate, NOT Etape2
    if stage < 2:
        f_1cc9370()
        f_1cc8d90(current_day_serial, ...)
        # (not read: clock/timing guard 2045781-2045794)

    # label "Etape1" (corpus 2045796)
    if stage < 3:
        if flag_326caa01 != 0:
            profile = f_1a4e8d0()
            f_1a50500(profile)
        profile = f_1a4e8d0()
        f_1a4d9f0(profile, global 4649168c, not mode_active, 0)
        if last_call_result == 0 and flag_463d1b41 == 0:
            return_value = 2
            flag_326caa01 = 0
            goto finish                            # early return 2, skips Etapes 2-19
        flag_326caa01 = 1
        # (not read: clock/timing guard 2045882-2045924)
    # (not read: the rest of the Etape1 block up to the Etape2 label at 2046212)

    # label "Etape2" (corpus 2046212)
    if stage < 4:
        f_159bfe0()
        for each person record (count at person_root+4, stride 0xad0):
            # f_21bf540 / f_10f2060 on the 0x41b8 profile record
            # (not read: full loop)
        # (not read: clock/timing guard 2046310-2046327)

    if stage < 5:
        f_1c386d0()
        # (not read: clock/timing guard 2046339-2046367)

    if stage < 6:
        f_1c408f0()
        # (not read: clock/timing guard 2046379-2046395)

    if stage < 7 and mode_value == 0:              # gate is (stage < 7) and (4649165c == 0); corpus 2046396
        # clock/timing only

    # (not read: the rest of the Etape2 block up to the Etape3 label at 2046427)
    # label "Etape3" (corpus 2046427): f_193af90 on 0x41b8 * index + PEPR base
    # labels Etape4 .. Etape20 (19 labels total; there is no Etape10)
    # (not read: the remaining stage bodies)

    return_value = all bits set (unsigned -1)      # corpus 2048635
finish:
    f_12bd280()
    return return_value                            # corpus 2048636-2048638
```
> Evidence: calc_monde_debut_tour_etapes [fn 0x1409f40], corpus 2045281-2048638; the stage < 7 gate needs 4649165c == 0 (corpus 2046396) and the stage < 3 block can return 2 via 2045876-2045879 to 2048636.

**What the code does not establish.** The corpus does not map each Etape to a domain responsibility, does not prove that one Etape runs per rendered frame (the blocks contain `clock()`/`f_91de00()` budget checks, but the frame relationship is not stated), and does not contain an Etape10 label. The `0x41b8` stride is used with [global 0x1e660190 ulonglong], but the record count is not established.

---

## 4. World Database Serializer and Memory Layout (`es_base`)

The serializer is [fn 0x143acd0 es_base] (mangled `Cdatabase__es_base`), 39,622 bytes. The decompiled signature is

```
ulonglong es_base(uint param_1, ushort *param_2, ulonglong param_3, undefined8 param_4)
```

so the old `void es_base(int mode, const wchar_t* path, int version = -1)` is corrected: the return type is `ulonglong`, the path is a `ushort *` (wide string), and there are four parameters. `param_3` is threaded into the per-field read/write helpers, but the corpus does not name it "version" and does not show a default of -1.

The mode dispatch is a chain at corpus lines 2073817-2073839:

```
if (param_1 == 5)      uVar58 = io_read(puVar30, 0x40d9908);
else if (param_1 == 1) uVar58 = f_146c790(puVar30, 1, p2, ...);
else if (param_1 == 6) uVar58 = io_write(puVar30, 0x40d9908);
else if (param_1 == 2) uVar58 = f_146c790(puVar30, 2, p2, ...);
else if (param_1 == 3) uVar58 = f_146c790(puVar30, 3, p2, ...);
else if (param_1 == 4) { inlined_stub_45(puVar30, 0, 0x40d9908);
                         uVar58 = f_146c790(puVar30, 3, p2, ...); }
else if (param_1 == 7) uVar58 = f_146c790(puVar30, 3, p2, ...);
```

Thus:
- `Mode 5` is a binary read through [fn 0x25b58e0 io_read];
- `Mode 6` is a binary write through [fn 0x25b5ea0 io_write];
- `Mode 1` is the field walker [fn 0x146c790 f_146c790] with selector 1;
- modes 2, 3, 4 and 7 are also implemented; the old volume listed only 5, 6 and 1.

The old claim that mode 1 performs "delta compression against a base reference file ([global 0x70eeaa50 undefined4])" is removed. `70eeaa50` is a file handle: the caller at 4846113 does `_DAT_70eeaa50 = f_25eeb70(...)`, and `io_read_raw` / `io_read_buffered` consume it [fn 0x25b58e0 io_read] [fn 0x25b5ea0 io_write]. No second reference buffer or delta base is visible in the mode-1 walker.

```mermaid
flowchart TD
    A["es_base(mode, path, serial_state, extra_arg)"] --> B{"mode"}
    B -->|5| R["io_read 0x25b58e0"]
    B -->|6| W["io_write 0x25b5ea0"]
    B -->|1| F1["f_146c790 selector 1"]
    B -->|2| F2["f_146c790 selector 2"]
    B -->|3,7| F3["f_146c790 selector 3"]
    B -->|4| F4["stub + f_146c790 selector 3"]
    F1 --> H["NATI header: 4-byte tag + 4-byte size"]
    H --> N["nation records at 0x46208908, stride 0x645310"]
```

### 4.1 Record header

The old `SaveChunkHeader { fourcc; record_size; }` struct is not a named type in the corpus, but the field sequence is observable in [fn 0x147d030 f_147d030] (corpus 2111594-2111600):

```
uStack_12088 = 0x645310;       // 4-byte record size
uStack_1208c = 0x4e415449;     // 4-byte tag "NATI"
f_8604f0(fd, &uStack_1208c, param_3);
f_8604f0(fd, &uStack_12088, param_3);
```

`f_8604f0` reads/writes four bytes per call, selecting `io_read` for mode 5, `io_write` for mode 6, and the field helpers for the other modes. The tag is [tag 0x4e415449 "NATI"] and the size is the nation stride [qty 0x645310 stride]. So the NATI record does begin with an 8-byte tag/size pair; the corpus does not define a general struct with that name for every record type.

### 4.2 Arrays and strides

| Object | Evidence |
| --- | --- |
| Nation array | Base [global 0x46208908 longlong]; record stride [qty 0x645310 stride]. `f_8c5310`, inside `if (param_1 == 0x166)`, runs 299 iterations: `lVar2 = 299; ... lVar2 = lVar2 + -1; } while (lVar2 != 0);` [fn 0x8c5310 f_8c5310]. The literal `300` is in `f_8c5500` (`while (iVar6 < 300)`, with `iVar6` starting at 1), which also runs 299 iterations [fn 0x8c5500 f_8c5500]. No assignment of the literal 300 to [global 0x322a3f4 i32] was found. |
| Variable store | Base [global 0x32d1920 undefined4]; `variable_store_get` calls `f_27933fc(&DAT_32d1920, 0x10, 4000000, f_1cc89c0)`, i.e. 4,000,000 records of [qty 0x10 record size] bytes [fn 0x8b9490 variable_store_get]. |
| Persons array | Base [global 0x5d85ada0 longlong]; stride [qty 0xad0 stride], e.g. `*param_1 * 0xad0 + _DAT_5d85ada0` [fn 0x869870 f_869870]. The 51,000 count is not established at that site. |
| Companies array | Base [global 0xe1e9910 longlong]; stride [qty 0x1f60 stride], e.g. `iVar10 * 0x1f60 + _DAT_e1e9910` [fn 0x914a20 f_914a20]. The 124,000 bound is used against a different base [global 0xd65da58 longlong] in `f_8dc870`; it is not established for `e1e9910`. |
| PEPR stride | [qty 0x41b8 stride] used with [global 0x1e660190 ulonglong] at corpus 2046501 [fn 0x1409f40 calc_monde_debut_tour_etapes]. |

The old "8.48 GB in-memory state" figure is removed: it is a runtime observation, not a value in the corpus.

### 4.3 NATI sector region

The sector region is real but the old arithmetic is wrong. The code computes `nation_base + sector_index * 0x4068 + 0xcbc8` [qty 0x4068 sector stride] [qty 0xcbc8 sector base offset], e.g. at corpus 114703 in `f_8c5310`: `(longlong)param_2 * 0x4068 + lVar3 + _DAT_46208908 + 0xcbc8`. `0x4068` is 16,488 bytes. The old product `364 × 0x4068 = 6,108,864` is arithmetically wrong: 364 × 16,488 = 6,001,632. The corpus does not state a 364-sector count at that site, so the count and the product are removed.

### 4.4 Pseudocode (serializer and layout)

```pseudocode
function save_load_world(mode, path, serial_state, extra_arg):   # Cdatabase__es_base, 0x143acd0
    # (not read: setup and header path handling before the mode dispatch)
    store = variable_store_get()
    f_1376270(...)

    if mode == 5:
        result = io_read(store, 0x40d9908)            # block size 0x40d9908
    else if mode == 1:
        result = walk_fields(store, selector 1, path, ...)
    else if mode == 6:
        result = io_write(store, 0x40d9908)
    else if mode == 2:
        result = walk_fields(store, selector 2, path, ...)
    else if mode == 3:
        result = walk_fields(store, selector 3, path, ...)
    else if mode == 4:
        inlined_stub_45(store, 0, 0x40d9908)
        result = walk_fields(store, selector 3, path, ...)
    else:
        result = incoming_result
        if mode == 7:
            result = walk_fields(store, selector 3, path, ...)
    # (not read: the rest of the 39,622-byte function)


function write_nati_header(file, ...):                 # f_147d030, 0x147d030
    write 4-byte tag  "NATI"  (0x4e415449)
    write 4-byte size 0x645310             # nation record stride
    each write goes through f_8604f0, 4 bytes per call


function scan_nation(nation_id, sector_index):         # f_8c5310, 0x8c5310
    if nation_id == 0x166:                 # special selector; corpus 114696
        best = 0.0
        offset = 0x645310                  # nation stride
        record = active_flags              # one int per nation
        repeat 299 times:                  # 299, not 300 (corpus 114698)
            if record's flag > 0:
                value = f_1a37c40(nation_base + sector_index * 0x4068
                                  + offset + 0xcbc8)
                best = max(best, value)
            offset = offset + 0x645310
            record = record + 1
        return best
    return f_1a37c40(nation_base + nation_id * 0x645310
                     + sector_index * 0x4068 + 0xcbc8)


function variable_store_get():                         # 0x8b9490
    # The guard reads a counter through a per-thread pointer chain at
    # [field 0x514e64 in variable_store_get]; the displacement is a field
    # offset, not an address (D-010).
    if the counter is below the allocation threshold:
        create the store
        if creation returned -1:            # allocation-failure sentinel
            reserve 4,000,000 records of 16 bytes each
            mark the store initialised
            register it once
            release the temporary allocation handle
            return the store
    return the store                        # shared, created lazily
```
> Evidence: Cdatabase__es_base [fn 0x143acd0], corpus 2072351-2073839; NATI header f_147d030 [fn 0x147d030], corpus 2111594-2111600; nation scan f_8c5310 [fn 0x8c5310], corpus 114686-114716; variable_store_get [fn 0x8b9490], corpus 105569-105588.

**What the code does not establish.** The corpus does not define a `SaveChunkHeader` type, does not name `param_3` a version, does not implement delta compression in the visible mode-1 path, does not contain the 8.48 GB figure, and does not state a 364-sector count or the old byte product. Those are removed rather than approximated.

---

### Citation and evidence summary

Old-volume RVAs and tokens were classified against `.team/corpus_index.tsv` and `.team/globals_index.tsv`. The per-claim ledger, including the closing statistics, is in [`evidence/01_CLAIMS.md`](evidence/01_CLAIMS.md). The short form:

- 91 unique address tokens in the old volume: 66 resolve as functions, 19 as globals, 6 as quantities, 0 as tags.
- One wrong function address was corrected: `f_140e000` (called `bank_step`) is an 8-byte global, not a function header; the real `bank_step` is [fn 0x1378460 bank_step].
- Two targets whose named behaviour is absent were replaced by "not established": [fn 0x1053e40 f_1053e40] (38-byte helper, not `plansociaux`) and [fn 0x959710 f_959710] (three-byte no-op, not `classer_joueur`).
- The old "36 phases", "20 Etapes with no gap", "COleDateTime", "Julian day", "8.48 GB", "delta compression", and the 14.03846-day step are removed or corrected in place.

Quantities used by the new text are literals in the named functions: [qty 0x645310 stride], [qty 0x4068 sector stride], [qty 0xcbc8 sector base offset], [qty 0x1f60 stride], [qty 0xad0 stride], [qty 0x41b8 stride], [qty 0x40d9908 byte count], and [qty 0x10 record size]. They are not function or global headers and are not presented as such.
