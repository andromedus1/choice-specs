---
description: "Campaign 3 (Input & Interaction) synthesis — head-tilt verdict, gesture vocabulary, library nav UX, and peripheral form factor"
type: brief
campaign: 03-input-and-interaction
program: choice-specs-feasibility
research_method: deep-research
content_type: campaign-synthesis
confidence: speculative
status: draft
updated: 2026-04-25
---

# Campaign 3 — Input & Interaction: Synthesis

## Campaign question

How does the user *control* a head-locked, monocular, e-ink AR-glasses bedtime book reader without using their hands, voice, or eyes — and what fallback peripheral covers the cases where the primary input fails?

This campaign covered eight sub-briefs spanning the novel territory (head-tilt as primary input in supine posture) and the well-trodden territory (BT HID rings as fallback). It also took ownership of the entire library navigation UX — both the *screens* and the *input modality together* — moved here from Campaign 2 for single ownership.

## Decision: Head-tilt feasibility verdict

**Conditional yes — head-tilt is the primary input, but only with the full mitigation stack.**

C3 brief 1 establishes that naïve head-tilt has 10–25% false-positive rates in supine posture (pillow noise overlaps the gesture distribution). This is unusable. The literature converges on a five-mitigation stack that drops residual FP to **<1 unintended page-turn per 60-min session** (the design target):

1. **Compound gesture (tilt + 400 ms hold)** — the largest single FP reducer.
2. **Per-session calibration (1.5 sec stillness handshake on session start)** — defines body-fixed reference frame in the actual reading posture.
3. **Stillness re-anchor / ZUPT during pauses ≥1 sec** — eliminates ~80% of long-session drift between gestures.
4. **Velocity-envelope activity gate (ω > 30°/s for ≥50 ms)** — only candidate gestures reach the classifier.
5. **800 ms refractory window after each fire** — prevents return-to-neutral double-trigger.

**Without all five, the device fails principle 5 (sleep-grade reliability).** The mitigations are well-understood from accessibility tech (GlassOuse 0.5 sec dwell), pedestrian navigation (ZUPT), and gesture HCI (Banks-class compound design). Implementation is the work, not invention.

**Empirical FP rates remain unmeasured for choice-specs's specific posture / pillow / IMU placement.** Prototype testing must produce its own data; the order-of-magnitude estimate (<1/session) is robust but specific number is TBD.

## Decision: Sensor fusion approach + library

**Mahony filter at 100 Hz on RK3566, BMI270 IMU above the bridge, deep-sleep between gestures with any-motion wake.**

Per C3 brief 2:
- **Mahony beats Madgwick / Kalman / complementary** for choice-specs's profile: ~3.5° accuracy (within 0.4° of Kalman), lowest CPU, two PI gains that handle bias drift naturally, no matrix operations.
- **`ahrs` Rust crate** is the recommended library; battle-tested reference implementations also exist in C/C++ (`Reefwing-AHRS`, xioTechnologies/Fusion).
- **BMI270 IMU** in raw mode (not the on-chip BNO055-style fused mode) — we need explicit control of ZUPT and per-session calibration that on-chip fusion doesn't expose.
- **Above-the-bridge placement** keeps the IMU on the rotational axis and minimizes the asymmetric-temple-weight signal coupling from C1's monocular optical engine.

**Power architecture:** Option B — SoC sleeps between gestures, IMU runs in any-motion-wake mode at <50 µA, Mahony cold-starts on wake using accelerometer-only gravity reading, converges in ~200 ms. Estimated **average ~30–40 mW** during a reading session. 1.85 Wh battery → ~50 hours of active reading.

**Caveat for the MentraOS path** (Campaign 2 will resolve): if choice-specs targets Vuzix Z100 / Even G1 via MentraOS rather than custom hardware, IMU access through the MentraOS SDK may not expose the low-level controls needed for the supine-mitigation stack. **Confirm with C2 before committing.**

## Decision: Gesture vocabulary + segmentation

**Four gestures, all compound (tilt + 400 ms hold):**

| Gesture | Head motion | Action |
|---|---|---|
| Roll right (~15°) + 400 ms hold | next page (in book); next item (in list) |
| Roll left (~15°) + 400 ms hold | previous page (in book); previous item (in list); also undo |
| Pitch up (~15°) + 400 ms hold | exit book → library (in book); back to library (in settings) |
| Pitch down (~15°) + 400 ms hold | open / select (in library and settings); no-op or recalibrate (in book) |

**Segmentation state machine:**

```
IDLE → ω > 30°/s → ACTIVE → settled at >12° displacement on candidate axis → HOLD
HOLD → 400 ms held → FIRE → 800 ms REFRACTORY → IDLE
HOLD → motion >5° from candidate → CANCEL → IDLE
```

End-to-end **~750 ms** from start of tilt to page rendered. ~1.5 sec between consecutive gesture fires. Latency is not the binding constraint — reliability is.

**Feedback:** corner-pixel visual indicator (always; partial-refresh e-ink); BT ring haptic pulse (when ring paired). No audio (partner sleeping). No frame-mounted vibration motor (weight cliff).

## Decision: Calibration + drift state machine

**Three-layer pattern, per C3 brief 4:**

1. **Per-session calibration on session start.** 1.5 sec stillness handshake during donning; captures neutral, gravity vector, and gyro bias. Triggered by the temple power button, the BT ring's middle button, or pitch-down (after handshake bootstraps).
2. **Continuous ZUPT during stillness.** Whenever ω <2°/s for ≥1 sec, reset velocity, update gyro bias, snap orientation to accelerometer-implied gravity. Eliminates ~80% of long-session drift; ~90% of session time is stillness for a reader.
3. **Auto-recalibration on sustained deviation.** If orientation departs >20° roll or >15° pitch from neutral for >5 sec at low ω, silently re-capture neutral. Triggered only from IDLE state. Handles "user shifted in bed" / "rolled onto side and back" cases.

**Manual recalibrate** via long-press of BT ring middle button or long pitch-down (3 sec) — for "I just rolled over, reset where you think 'up' is."

**Deferred to v2:** explicit doze-off detection, sit-up "pause" sub-state.

## Decision: BT pairing flow

**Just-Works BLE pairing, glasses act as HID Host, no companion phone:**

1. User holds the ring's pairing button (5 sec).
2. User holds glasses' temple power button (5 sec) OR completes long pitch-up gesture (3 sec).
3. Both go into Just-Works pairing mode. Bond completes silently in <2 sec.
4. Visual indicator: corner-pixel fill animation during pairing.
5. Auto-reconnect on subsequent sessions via directional advertising → general advertising fallback.

**Latency:** <350 ms end-to-end (BT button press → page rendered). Comparable to Kindle physical button.

**Antenna placement:** dominant-eye temple (same side as optical engine, minimum-distance line-of-sight to user's hand on that side).

**Blanket attenuation is not a meaningful concern** — 2.4 GHz through cotton/down bedding contributes 2–4 dB; BLE link budget over 60 cm has ≥80 dB margin. **Body absorption is a concern** (20–30 dB through torso) — mitigated by ring being on the dominant hand resting on top of the body, not under it.

## Decision: Library navigation UX

**This is the most concrete deliverable of the campaign.** Three-screen architecture: **Library / Reading / Settings.**

### Library (entry screen)

- **Vertical list, 6 items per screen**, recents-first, currently-reading at top, monospaced 14-pt text.
- Each item: title (truncated ~25 chars) + author last name + progress bar.
- Roll right/left = next/prev item. Hold past 400 ms = fast-scroll (6 items per 800 ms).
- Pitch down = open selected book at last-read position.
- Pitch up = no-op (or settings via long-press).
- Partial-refresh e-ink for selection feedback (~100 ms); full-refresh only on screen transitions.

### Reading

- Full-page text, head-locked at 50 cm virtual distance (per C1).
- Corner-pixel gesture-feedback indicator + page count.
- All four gestures functional (roll right/left = page; pitch up = exit; pitch down = recalibrate).

### Settings

- Same vertical-list paradigm. Items: font size, brightness, refresh mode, sleep timeout, pair BT ring, calibrate now, back to library.
- In-place value editing: roll right/left increments/decrements while item selected; pitch down commits.

### Library scaling

- 1–50 books: native list works.
- 50–200 books: alphabet bar appears above list; roll-right at top selects A→Z, pitch-down jumps to first matching letter.
- 200–1000+ books: deferred to v2 (search via phone companion).

### Critical UX behaviors

- **Currently-reading book at top of list always.** "Exit and resume" is symmetric.
- **Books resume at last-read position**, not cover.
- **No grid view, no search, no folders, no nested views in v1.**
- **No "are you sure?" flows** — every action has a one-gesture inverse.

### State diagram

```
POWER OFF → CALIBRATING (1.5s) → LIBRARY ⇄ READING ⇄ LIBRARY ⇄ SETTINGS → ...
```

## Decision: Peripheral form factor + button layout

**Primary: Gevo BT Page Turner Ring** ($20–30, USB-C charging case, 14 hr active / 60 d standby, BLE HID).

**Backup: Generic BT clicker as palm dongle** ($15) for users who don't want a ring. Add BLE-locator-chirp feature for lost-dongle recovery.

**Rejected:** wrist clicker (bulky in supine), pendant (cold/noisy/slip), finger sleeve (no commercial product).

**Deferred to v2:** Genki Wave Ring for haptic-confirm path, custom finger sleeve.

**Button layout (3-button minimum):**

| Button | Single press | Long press |
|---|---|---|
| **Top** | Next page / Next item | (no-op) |
| **Bottom** | Previous page / Previous item | (no-op) |
| **Middle** | Library (in book) / Select (in list) | Force-recalibrate / Settings |

**Find-in-dark:** long-press temple power button → ring chirps audible + haptic.

The ring is a fallback / accelerator, not a requirement. Head-tilt remains primary; the ring just makes navigation faster in the library.

## Cross-cutting findings

### 1. The C3 mitigation stack is non-negotiable

No single mitigation closes the supine-FP gap. Compound gesture + per-session calibration + ZUPT + velocity gate + refractory window all stack multiplicatively. **Choice-specs MUST ship all five.** Skipping any one drops residual FP rate by an order of magnitude.

### 2. Self-inverse gesture vocabulary eliminates confirm flows

Every action in choice-specs has a one-gesture undo path. Roll-right page advance is undone by roll-left. Pitch-up exits to library; selecting same book in library re-enters at last position. **No "are you sure?" flows** are needed. This raises the FP tolerance — even if the system fires once unintentionally per session, the recovery is a single follow-up gesture. The user never gets stranded.

### 3. The "fail quiet" rule is load-bearing

When input is ambiguous, the device does nothing. The state machine returns to IDLE on cancel or timeout. There's no error UI; missed gestures just feel like nothing happened. This costs nothing functionally (the user re-issues) and prevents the worst failure mode (firing the wrong action).

### 4. Power architecture and IMU choice are coupled

Mahony filter is the algorithm choice; **deep-sleep-between-gestures is the power architecture**. The IMU's any-motion wake mode bridges the two. If we'd chosen continuous-fusion (e.g., always-on Kalman with magnetometer), the SoC couldn't sleep, and the power budget falls apart. The fusion choice was constrained as much by C4-class power needs as by accuracy.

### 5. The asymmetric optical engine (C1 inheritance) makes IMU placement non-trivial

Monocular pancake on the dominant eye + e-ink panel + driver = ~25 mm of weight on one temple. Bridge-mounted IMU is the only placement that gets clean rotational signals. Frame depth at the nose is the trade-off. **C4 must accommodate.**

### 6. The library nav UX is the single most-concrete deliverable in C3

This is what an implementer can build from. Three screens. Six items per page. Four gestures. Three ring buttons. Five hundred lines of UI code. Anyone reading this brief should be able to imagine the device behavior precisely. This level of concreteness is what C3 trades on — input/interaction is the part of the system that users actually touch.

### 7. Sol Reader's handheld-remote baseline does not solve our use case

Sol uses a handheld remote that requires the hand to be out of the blanket holding it. Choice-specs's primary differentiation per principle 2 is hands-stay-under-blankets. **Head-tilt as primary input is the axis of differentiation** even though it's the harder engineering bet. The BT ring is a category compromise (the user holds something, but only a ring, and it stays on the hand) — but the head-tilt path is what makes choice-specs structurally different from Sol.

## Open questions for downstream campaigns

### To Campaign 4 (Industrial Design, Power, Safety)

1. **Power budget integration.** C3's estimated ~30–40 mW average for IMU + Mahony + BLE-listening combines with C1's display power (~10–30 mW for e-ink with frontlight) and C2's compute power (TBD). Total system active should land in the 60–150 mW range. **Battery sizing in C4 must accommodate** — 500–800 mAh single-cell LiPo gives a wide margin.
2. **IMU bridge placement adds frame depth at the nose.** ~5–8 mm extra over a fashion-glasses bridge. C4 needs to accommodate ergonomically.
3. **Single physical button on dominant temple** for power / pairing / wake. Industrial design must include it; placement matters for one-finger reach.
4. **Doze-off detection** is partly a C3 calibration problem and partly a C4 sleep-context-safety problem. Coordinate: C3 detects "no gesture + slow drift for 10 min"; C4 owns the safe-shutdown flow (don't keep display on; battery cell safety; recovery from power-off-while-on-face).
5. **BLE antenna routing** — dominant temple, near optical engine. Tight against optical components. C4 needs to fit the antenna in.
6. **BT ring + pairing case storage.** A bedside docking solution would be nice. Out of scope for v1 but flagged.

### To Campaign 2 (Compute & Software Stack) — back-references

1. **MentraOS IMU access.** C3 brief 2's recommended architecture (Mahony + ZUPT + per-session calibration + any-motion wake) requires low-level IMU control. **C2's MentraOS evaluation must confirm whether the SDK exposes this** or if MentraOS forces choice-specs onto a custom-hardware path.
2. **EPUB renderer integration with the gesture state machine.** Whichever renderer C2 picks (KOReader / Plato / custom) must accept page-next / page-prev events from the gesture system at <50 ms latency. Both KOReader and Plato have framebuffer-event hooks that suffice.
3. **Library list rendering.** C3 brief 6 specifies the screen layout but C2 owns the rendering pipeline. Confirm partial-refresh capability for selection-change events on the chosen e-ink stack (Modos Caster).
4. **Settings persistence.** Per-user dwell time, font size, brightness must persist across sessions. C2 owns the storage layer.

### Unresolved at C3 (for prototype)

1. **Empirical FP rate in supine posture on prototype hardware.** The single most important measurement.
2. **Wake-to-orientation latency on real hardware** — the 200 ms estimate must be validated.
3. **400 ms hold duration tuning** — may want 300 ms or 500 ms in final.
4. **Pitch-up "library" gesture conflicting with "look at ceiling" sleep-context behavior.** Empirical only.
5. **Cross-pillow generalization** — different pillow types may need different ZUPT thresholds.
6. **User-in-loop drift as user gets sleepy.** End-of-session FP rate vs start-of-session.
7. **Ring battery in choice-specs's session pattern** — Gevo's 14 hr is on its own assumption; verify with our HID event load.

## Brief index

- [01 — Head-Tilt Reliability in Supine / Reclined Posture](briefs/01-head-tilt-supine-reliability.md) — feasibility verdict + the five-mitigation stack
- [02 — IMU Sensor Fusion Approaches](briefs/02-imu-sensor-fusion.md) — Mahony @ 100 Hz on BMI270 + deep-sleep wake
- [03 — Compound Gesture Design and Segmentation](briefs/03-compound-gesture-design.md) — 4-gesture vocabulary + state machine + feedback
- [04 — Calibration and Drift Handling](briefs/04-calibration-drift-handling.md) — three-layer state machine
- [05 — BT HID Integration Patterns](briefs/05-bt-hid-integration.md) — Just-Works pairing + dominant-temple antenna
- [06 — Library Navigation UX (Screens + Input Together)](briefs/06-library-nav-ux.md) — three-screen design with mockups and state diagram
- [07 — Accessibility Input Prior Art Deep Dive](briefs/07-accessibility-input-prior-art.md) — five patterns adopted (dwell timing, fail-quiet, self-inverse, etc.)
- [08 — Peripheral Form Factor](briefs/08-peripheral-form-factor.md) — Gevo BT ring primary; palm dongle backup
