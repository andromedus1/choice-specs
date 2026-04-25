---
description: "Gesture vocabulary, segmentation, timing, and feedback for choice-specs head-tilt input — 4 gestures, tilt+400ms hold, 800ms refractory window"
type: brief
campaign: 03-input-and-interaction
program: choice-specs-feasibility
research_method: deep-research
confidence: speculative
status: draft
updated: 2026-04-25
---

# Compound Gesture Design and Segmentation

## Question

What is the actual gesture vocabulary choice-specs should ship? How are gestures *segmented* (start/end detected)? What's the timing budget? What user-visible feedback closes the loop?

The constraints from C3-brief-01: any single-axis tilt-as-trigger has 10–25% false-positive rate from pillow noise. The mitigation is compound design — adding a temporal or multi-axis structure that pillow noise can't accidentally satisfy. But "more compound" trades against intentional-gesture friction (the user has to do more to register intent).

The principles "hands stay under the blankets" + "sleep-grade reliability" pull in opposite directions on this trade-off. A compound gesture that takes 1.5 seconds is reliable but feels sluggish; a single-axis 200ms threshold is responsive but unreliable.

## Findings

### What's the minimum gesture vocabulary?

Choice-specs needs to cover:
1. **Next page** (most-frequent action; ~120/hour at 250 wpm)
2. **Previous page** (rare; <5/hour but essential for accidental-trigger recovery)
3. **Open library / exit book** (transitions screens; ~2-5/session)
4. **Confirm / select** (used in library and settings; only when a list is active)

That's **4 gestures**. Anything more must be earned by frequency.

Notable exclusions:
- **Sleep / wake** — handled by donning/doffing detection (C4 territory) plus explicit power button on the temple, not gestures.
- **Brightness / font size** — settings screens, not gestures. Goes through the gesture vocabulary above.
- **Bookmark / highlight** — explicitly out of scope per north-star.

### Gesture mapping to head motion

Two physical axes are clean in supine posture: **roll** (tilt left/right, ear-to-shoulder) and **pitch** (chin-up / chin-down). Yaw (head shake) is constrained by pillow geometry — the head can't rotate freely on the horizontal plane when resting.

Proposed mapping:

| Gesture | Head motion | Compound structure |
|---|---|---|
| **Next page** | Roll right (~15°) | + 400 ms hold |
| **Previous page** | Roll left (~15°) | + 400 ms hold |
| **Library / menu** | Chin up (pitch +15°) | + 400 ms hold |
| **Confirm / select** | Chin down (pitch −15°) | + 400 ms hold OR ring-click |

**Why this mapping is right:**
- Roll right → next page is consistent with reading direction (LTR; the user's right side advances). Mirror for RTL languages but choice-specs v1 is English-only.
- Pitch up → library mirrors the natural "look at the bookshelf above me" affordance.
- Pitch down → confirm mirrors "nodding agreement."
- All four are 15° amplitudes — well above pillow noise (~5°) but small enough to perform without disturbing pillow position.

### Segmentation: how to detect gesture start and end

**Activity-detection gating** per [Head Gesture Recognition Combining Activity Detection and DTW (MDPI 2024)](https://www.mdpi.com/2313-433X/10/5/123):

1. **Idle state.** Mahony filter runs continuously. Track angular velocity magnitude.
2. **Trigger state entered when** angular velocity exceeds 30°/s for ≥50 ms (start of intentional gesture).
3. **Hold state entered when** orientation has settled (angular velocity <5°/s) and a candidate axis (roll right, roll left, pitch up, pitch down) has accumulated >12° displacement from neutral.
4. **Confirm state** — if the user holds within ±3° of the candidate position for 400 ms, the gesture fires.
5. **Refractory state** — after firing, suppress detection for 800 ms (allow head return to neutral without triggering the opposite gesture).

State machine summary:

```
IDLE → (ω > 30°/s) → ACTIVE → (settled, axis identified) → HOLD
HOLD → (400 ms held) → FIRE → REFRACTORY (800 ms) → IDLE
HOLD → (movement >5° from candidate) → CANCEL → IDLE
```

This is the [Smartwatch motion gesture delimiter pattern (Hindawi 2022)](https://onlinelibrary.wiley.com/doi/10.1155/2022/6879206) adapted to head-mounted: angular velocity thresholds delimit start, hold confirms intent.

### Timing budget

| Phase | Duration | Cumulative |
|---|---|---|
| Activity detection | 50 ms | 50 ms |
| Tilt motion (15° at ~50°/s peak) | 200 ms | 250 ms |
| Hold (compound) | 400 ms | 650 ms |
| Page render + display refresh | 100–300 ms | 750–950 ms |
| Refractory window (head returns) | 800 ms | 1550–1750 ms |

**End-to-end ~750 ms** from start-of-tilt to "page is turned." With a refractory window, the user can issue the next gesture ~1.5 sec after the previous fires. At 250 wpm and ~30 seconds per page, this is dramatically faster than the page rate. **Latency is not the binding constraint.** Reliability is.

### User-visible feedback

The display is e-ink with ~1 sec refresh time, which is a long perceptual delay for a confirmation signal. Feedback design must bridge this:

1. **Page-corner indicator (visual).** Reserve ~5×5 pixels in a corner of the page. As the user tilts, this indicator changes:
   - Idle: blank
   - Activity-detected (50 ms in): single dot
   - Hold confirmed (mid-400-ms): dot grows / fills
   - Fire: brief flash before page turns
   - This needs only partial-refresh, not full-page refresh — fast enough on Modos Caster-driven e-ink.

2. **Haptic confirmation (BT ring only).** If the BT ring is paired and worn, fire a brief 30 ms haptic pulse on gesture-fire. Does not depend on visual attention. Genki Wave Ring class haptic. [Genki Wave Ring](https://genkiinstruments.com/) confirms haptic at this pulse length is comfortable.

3. **No audio.** Per principle 2 (partner sleeping).

4. **No frame-mounted vibration motor.** Adds weight against principle 6 (≤50g). The ring's haptic is enough.

The visual indicator handles the no-ring case; the ring's haptic upgrades the no-ring experience to "feel the page turn" without lifting head.

### Why not gesture-to-gesture chains?

Some HCI research proposes "tilt-right-then-tilt-up" as a third gesture. **Rejected for choice-specs:**
- Confusing in low-attention bedtime context.
- Doubles the time per action.
- The 4-gesture vocabulary above is sufficient.

### Why not double-tap-tilt?

A "tilt-right twice in 500 ms" gesture as an alternative to the 400 ms hold is appealing — feels less effortful. **Rejected** per [Low-False-Positive Gesture (SciTePress 2016)](https://www.scitepress.org/papers/2016/57019/57019.pdf): double-tap gestures had higher false-positive rate than tilt+hold in their study. Pillow micro-rebounds occasionally produce double-event signatures.

### What about "look-and-dwell" gestures (no tilt, just sustained head pose)?

Equivalent to tilt+hold but typically uses pure orientation rather than rate-then-hold. The activity-detection gate is then bypassed. Pure dwell is *more* susceptible to pillow noise because pillows can passively hold the head at non-neutral angles for minutes. **Rejected.**

## Recommendation

**Ship 4 gestures, all compound (tilt + 400 ms hold), with a state-machine segmenter using angular-velocity thresholds and a refractory window:**

| Gesture | Roll/Pitch | Mapping |
|---|---|---|
| Roll right (15°) + 400 ms hold | next page |
| Roll left (15°) + 400 ms hold | previous page (also: undo accidental next) |
| Pitch up (15°) + 400 ms hold | exit book → library / open menu |
| Pitch down (15°) + 400 ms hold | confirm selection in library / menu |

**Segmentation state machine:** IDLE → ACTIVE (ω > 30°/s) → HOLD (settled at >12° displacement) → FIRE (400 ms held) → REFRACTORY (800 ms) → IDLE.

**Feedback:** corner-pixel visual indicator (always); BT ring haptic pulse (when ring paired). No audio.

**End-to-end latency:** ~750 ms gesture-to-page-rendered. Acceptable for reading cadence (~30s/page).

## Open questions / Gaps

1. **400 ms hold duration.** May feel too long once tested. Could go down to 300 ms if FP rate stays in budget; could climb to 500 ms if not. Tunable in firmware; default 400 ms.
2. **15° amplitude threshold.** Could be too aggressive in supine (pillow may constrain). Lower threshold = more FPs, higher = more user effort. Calibration UX could capture user-specific amplitude per session.
3. **Pitch-up for "library."** Conflicts with "look at the ceiling to think" common bedtime micro-motion. Worth empirical testing — perhaps swap to two-step "pitch up + roll right" for library, leaving pitch up alone less risky.
4. **Confirmation needed for irreversible actions.** Currently nothing is irreversible in choice-specs (every page turn can be undone with the inverse gesture). If features are added (e.g., "delete book"), they need a different gesture or a two-step confirm.
5. **Empirical FP rates per gesture.** Roll-axis pillow noise ≠ pitch-axis pillow noise. Pitch-down might have more FPs (settling deeper into pillow). The actual per-gesture FP rate must be measured on prototype.

## Sources

- [Design of Low-False-Positive Gesture (SciTePress 2016)](https://www.scitepress.org/papers/2016/57019/57019.pdf)
- [Head Gesture Recognition Combining Activity Detection and DTW (MDPI J. Imaging 2024)](https://www.mdpi.com/2313-433X/10/5/123)
- [HeadGesture (ACM IMWUT 2018)](https://dl.acm.org/doi/10.1145/3287076)
- [Motion Gesture Delimiters for Smartwatch Interaction (Hindawi 2022)](https://onlinelibrary.wiley.com/doi/10.1155/2022/6879206)
- [Smartwatch-based Early Gesture Detection (ACM IMWUT 2018)](https://dl.acm.org/doi/10.1145/3191771)
- [Robust and Deployable Gesture Recognition for Smartwatches (CHI 2022)](https://dl.acm.org/doi/fullHtml/10.1145/3490099.3511125)
- [Genki Wave Ring product page](https://genkiinstruments.com/)
