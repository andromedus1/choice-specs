---
description: "Research program plan for choice-specs feasibility — 4 campaigns covering display/optics, compute/software, input/interaction, and industrial design/power"
type: program-parent
content_type: program-plan
status: in-progress
updated: 2026-04-25
---

# Research Program: choice-specs Feasibility

*Created: 2026-04-25*

## Megatopic

Standalone AR-glasses bedtime book reader feasibility — establishing whether the device described in [north-star.md](../../architecture/north-star.md) is technically buildable, optically comfortable, and ergonomically wearable for the target use case (90-minute bedtime reading sessions, hands-under-blankets, ≤50g, e-ink-class display, EPUB only, head-tilt or BT clicker input).

## Decomposition

Four campaigns chosen along technical-discipline boundaries:

| # | Campaign | Slug | Sub-briefs |
|---|---|---|---|
| 1 | Display & Optics | `01-display-and-optics` | 10 |
| 2 | Compute & Software Stack | `02-compute-and-software` | 7 |
| 3 | Input & Interaction | `03-input-and-interaction` | 8 |
| 4 | Industrial Design, Power, & Safety | `04-id-power-safety` | 8 |

**Total:** 33 sub-briefs.

**Decomposition shape:** Layer + faceted. Display→compute→input→physical engineering forms a stack; each layer is also faceted internally.

**Distinctness check:** Three independent decompositions converged on roughly these 4. One overlap (library navigation UX) flagged and resolved by moving full ownership to Campaign 3.

## Dependencies

```
C1 (Display & Optics) — gating, no dependencies
   ├── C2 (Compute & Software) — depends on C1 (display tech constrains memory/SoC requirements)
   ├── C3 (Input & Interaction) — depends on C1 (optical path constrains IMU placement, peripheral pairing UX)
   └── C4 (Industrial Design, Power, & Safety) — depends on C1, C2, C3 (synthesizes physical constraints)

After all campaigns complete:
   → Cross-campaign synthesis (super-parent.md)
   → Program evaluation (program-report.md)
```

## Dispatch Strategy

**Hybrid sequential + parallel:**
- C1 dispatched first, alone.
- C2 + C3 dispatched in parallel after C1 completes.
- C4 dispatched after C2 and C3 complete.
- Synthesis and evaluation after C4.

**Incremental execution:** This program is being executed incrementally across multiple sessions to manage context budget. Campaign 1 is being dispatched in this session (2026-04-25); subsequent campaigns will dispatch in future sessions.

## Status

| Campaign | Status | Started | Completed | Notes |
|---|---|---|---|---|
| C1 — Display & Optics | ✅ complete | 2026-04-25 | 2026-04-25 | monocular pancake + e-ink @ 50cm. Sol Reader status corrected (discontinued, verified directly). |
| C2 — Compute & Software | ✅ complete | 2026-04-25 | 2026-04-25 | i.MX6 Solo + Buildroot + KOReader. Validate via Kobo Clara HD. MentraOS rejected. |
| C3 — Input & Interaction | ✅ complete | 2026-04-25 | 2026-04-25 | Head-tilt conditional primary (5-mitigation stack); 3-screen library nav; Mahony @ 100Hz on BMI270. |
| C4 — ID, Power, Safety | pending (next 5hr window) | — | — | depends on C1+C2+C3 |
| Cross-campaign synthesis | pending (next 5hr window) | — | — | after all campaigns |
| Program evaluation | pending (next 5hr window) | — | — | independent pass |

## Budget

- **Program ceiling:** $200
- **Per-campaign:** ~$40 default
- **Synthesis + evaluation:** ~$40
- **Rationale:** Above the $100 default given the project's stated preference for going broader/deeper. Well under the $400 hard ceiling.

## Constraints

- **Nested agent spawning blocked** (per /research-program skill notes). Each campaign Lead executes /deep-research workflow as a single Opus agent, not as parallel-specialist orchestration. Quality remains strong (skill demos: 0.86 coverage, 4.2/5 coherence on first programs) but campaigns are sequential internally.

## Output Paths

```
docs/programs/choice-specs-feasibility/
├── program.md                          ← this file (Planner's plan)
├── super-parent.md                     ← (TBD) cross-campaign synthesis
├── program-report.md                   ← (TBD) program evaluation
└── campaigns/
    ├── 01-display-and-optics/
    │   ├── parent.md                   ← Campaign Lead's synthesis
    │   └── briefs/                     ← 10 specialist briefs
    ├── 02-compute-and-software/
    ├── 03-input-and-interaction/
    └── 04-id-power-safety/
```

## Related Documents

| Document | Purpose |
|----------|---------|
| [North Star](../../architecture/north-star.md) | Vision, principles, domain model, research plan |
| [Scout Landscape](../../architecture/scout-landscape.md) | Prior art map; primary input to this program |
