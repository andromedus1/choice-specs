---
description: "Calibration UX and drift-handling state machine for choice-specs — per-session capture + ZUPT-on-stillness + auto-recalibration on sustained deviation"
type: brief
campaign: 03-input-and-interaction
program: choice-specs-feasibility
research_method: deep-research
confidence: speculative
status: draft
updated: 2026-04-25
---

# Calibration and Drift Handling

## Question

IMU drift over 1+ hour reading sessions is documented and real. The Mahony filter alone does not eliminate it. Combined with the supine-posture problem (gravity vector lives in the wrong axis until calibrated), choice-specs needs a **calibration + drift state machine** that:

1. Establishes a body-fixed reference frame at session start matching the user's actual reclined posture.
2. Maintains that reference across a 60+ minute session as the user shifts position.
3. Recovers gracefully when the user moves significantly (rolls onto side, sits up to drink water, repositions pillow).
4. Costs negligible user friction — the calibration UX must feel like part of donning the device, not a setup step.

The principle "sleep-grade reliability" forces this to be invisible-when-working and recoverable-when-broken.

## Findings

### What's actually drifting

Three distinct sources of drift accumulate over a session:

1. **Gyroscope bias drift.** A real-world MEMS gyroscope at rest reports nonzero angular velocity (typically ±0.05–0.5°/s); over time, integration accumulates a yaw/pitch/roll error. [IMU Reliability over 1hr Sessions (Nature SciRep 2021)](https://www.nature.com/articles/s41598-021-99459-7) measured ~3–8° drift per hour without correction.
2. **Reference-frame drift.** Even with perfect orientation tracking, the *user's posture* shifts. The pillow compresses, the head settles, the user repositions. The "neutral" defined at session start gradually no longer matches the user's actual rest pose. By 30 min, the user's "neutral" may be 5–10° rotated from what was captured.
3. **Temperature drift.** Gyro bias is temperature-sensitive. The IMU on the bridge of choice-specs warms during use. [BMI270 datasheet](https://www.bosch-sensortec.com/products/motion-sensors/imus/bmi270/) cites ±0.1°/s/°C zero-rate temperature coefficient. Over a 60-min session with a few °C warm-up, this contributes ~1–3° additional drift.

### Mitigations

#### 1. Per-session calibration (UX)

**Pattern:** Capture neutral at session start with a 1.5-second stillness window, with light visual feedback.

**The flow:**
1. User dons glasses, settles into reading position.
2. User triggers session start. (Trigger options: physical power button on temple → preferred; pitch-down "confirm" gesture → bootstrap problem since calibration not done; BT ring button → fine if paired.)
3. Display shows "Hold still..." with a 3-frame e-ink animation (dot growing).
4. After 1.5 sec of <3°/s motion, calibration captures: (a) gravity vector → defines vertical axis, (b) accelerometer-mean → bias estimate, (c) current orientation → "neutral" reference.
5. Display fades to first page.

**Failure modes:**
- User can't hold still for 1.5 sec → extend window up to 3 sec; if still failing, use a more permissive threshold (5°/s) and warn ("calibration approximate").
- User triggers session before settling → at first ZUPT (next stillness >1 sec), silently re-capture neutral.

**This is not "set up the device" — it's a 1.5-second handshake.** Comparable to opening a Kindle and seeing the home screen draw. The UX cost is real but small.

#### 2. Stillness re-anchor / ZUPT (continuous mitigation)

Whenever the head is still (angular velocity <2°/s) for ≥1 second between gestures:
- Reset velocity integration to zero.
- Update gyro bias estimate from steady accelerometer reading.
- Fold accelerometer-only orientation back into the Mahony filter aggressively (high gain).

Per [Magnetometer-Based Drift Correction During Rest in IMU Arm Motion Tracking (PMC 2019)](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC6471153/), rest-based correction reduces accumulated drift by ~80% over 30-min trials. For reading, where 90%+ of time is "still between gestures," ZUPT is exceptionally well-suited. **Drift between gestures is essentially eliminated.**

#### 3. Auto-recalibration on sustained deviation

If for >5 sec the orientation differs from the calibrated neutral by >20° in roll or >15° in pitch, *and* angular velocity has been low (<5°/s) for that interval, assume the user has shifted to a new resting posture (rolled onto side; rolled back; sat up). **Silently re-capture neutral** to the new orientation.

The threshold tuning matters:
- Too sensitive → recalibrates during a held gesture, breaking the gesture mid-hold.
- Too lax → user shifts but old neutral stays anchored, all gestures drift directionally.

Proposed: 20° roll + 15° pitch + 5 sec sustained + ω <5°/s. Recalibration is silent (no UI), takes 100 ms, instantly remaps.

**Design note:** the gesture state machine (C3 brief 3) must check "is ACTIVE/HOLD/REFRACTORY" before allowing recalibration; recalibration only fires from IDLE.

#### 4. Temperature-aware bias

A simple linear temperature correction: store bias-vs-temp coefficients from a one-time factory calibration. At runtime, subtract `bias(t) = bias_0 + coef * (T - T_0)` from gyroscope readings. The BMI270 reports temperature on a register; reading it costs 100 µs every 10 sec.

For prototype, this can be skipped — ZUPT will absorb temperature drift implicitly. But for product, it matters.

### State machine

```
                               session-start trigger
                                       │
                                       ▼
                              ┌──────────────────┐
                              │  CALIBRATING     │
                              │  (1.5 s still)   │
                              └────────┬─────────┘
                                       │ neutral captured
                                       ▼
                  ┌─────────────────────────────────────┐
                  │              READING                │
                  │  (idle, gesture-detect, ZUPT loop)  │
                  └─┬──────────────┬──────────────┬─────┘
       ω > 30°/s    │              │  >5s sustained 20°+ │
                    ▼              ▼  deviation @ low ω  │
            ┌────────────────┐     │                     │
            │  GESTURE-ACTIVE│     ▼                     │
            │  (per brief 3) │  ┌──────────────────┐     │
            └────────┬───────┘  │  RECALIBRATING   │     │
                     │ fire/    │  (silent, 100ms) │     │
                     │ cancel   └────────┬─────────┘     │
                     ▼                   │ new neutral   │
                  back to READING        ▼               │
                                    back to READING      │
                                                         │
   doff / explicit-end ─────────────────────────────────► SESSION-END
```

### Calibration when user moves significantly

The interesting case is "user reaches for water, sits up briefly, lies back down." The naive flow:
- During sit-up: orientation deviates >20° pitch from neutral.
- Auto-recalibration fires → neutral now points "up" (the sitting pose).
- User lies back down → again deviates >20° from this new neutral.
- Auto-recalibration fires again → neutral points back roughly where it was.

This **double-recalibration is fine** functionally — the user ends up with a correct neutral. But during the sit-up window, all gestures are referenced to the wrong neutral (the lying-down one) and would mostly fail to fire (which is desired — we don't want page-turns from sitting up). 

A potential improvement: **detect sit-up explicitly** (sustained pitch >45° for >2 sec) and enter a "paused" sub-state where gestures are disabled. Resume on lie-back-down. **Defer to v2.** For v1, the auto-recalibration handles it adequately.

### Calibration when user is dozing off

As the user gets sleepy, head positions drift slowly: chin sinks toward chest, head rolls slightly to one side. Auto-recalibration absorbs this as long as the drift is gradual and matches the "low ω + sustained deviation" pattern.

The risk: an accidental gesture fires during dozing. This is the "session end" boundary case — the device should detect "no gesture in 10 min + heavy drift" and prompt to end the session. Or better: detect doze-off via a slowly-tilting orientation and explicitly enter sleep mode. **Out of scope for v1; flagged for C4 (sleep-context safety).**

## Recommendation

**Three-layer calibration + drift state machine:**

1. **Per-session calibration on session start.** 1.5 sec stillness handshake during donning; captures neutral, gravity vector, gyro bias.
2. **Continuous ZUPT during stillness.** Whenever ω <2°/s for ≥1 sec, reset velocity, update gyro bias, snap orientation to accelerometer-implied gravity. Eliminates ~80% of long-session drift.
3. **Auto-recalibration on sustained deviation.** If orientation departs >20° roll or >15° pitch from neutral for >5 sec at low ω, silently re-capture neutral. Triggered only from IDLE state, not during gesture detection.

**Defer for v2:** sit-up detection / pause-on-extreme-posture, explicit doze-off detection.

**Calibration UX is a 1.5-second handshake**, not a setup screen. It looks like the device "waking up" rather than asking the user to do something specific.

## Open questions / Gaps

1. **The "user dozes off" boundary.** Detection of fall-asleep state (slow drift, heavy/tilted head, no gestures for >10 min) and graceful auto-end has implications for both calibration *and* C4 (sleep-context safety). Coordinate with C4.
2. **First-time-on-device calibration.** Beyond per-session, is there a one-time factory calibration that ships with the device? The BMI270 has a Bosch-recommended factory cal. Probably yes; but per-session ZUPT may make this less critical.
3. **Cross-pillow generalization** — different pillows produce different micro-motion baselines. The 2°/s ZUPT threshold is a guess; needs measurement.
4. **What if the user reads while changing posture (e.g., shifts every minute)?** Frequent auto-recalibration might cause occasional "gesture fired but interpreted relative to wrong neutral" errors. Probably tolerable but unmeasured.
5. **Mid-session "manual recalibrate" gesture?** E.g., explicitly hold pitch-down for 3 sec to force recalibration. Useful for "I just rolled over and want to keep reading." **Probably yes for v1**; design it as part of the BT-ring path (long-press = recalibrate).

## Sources

- [IMU Reliability Study over 1hr Sessions (Nature SciRep 2021)](https://www.nature.com/articles/s41598-021-99459-7)
- [Magnetometer-Based Drift Correction During Rest (PMC 2019)](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC6471153/)
- [Enhanced Heuristic Drift Elimination with Adaptive ZUPT (PMC 2020)](https://pmc.ncbi.nlm.nih.gov/articles/PMC7070454/)
- [Wavelet-based Postural Event Detection (PMC)](https://pmc.ncbi.nlm.nih.gov/articles/PMC3755105/)
- [Head Tracker v2.2 — IMU, Calibration and Drift](https://headtracker.gitbook.io/head-tracker-v2.2/getting-started/imu-calibration-and-drift)
- [Bosch BMI270 datasheet](https://www.bosch-sensortec.com/products/motion-sensors/imus/bmi270/)
- [Hybrid Framework for Mitigating Heading Drift (MDPI 2021)](https://www.mdpi.com/2076-3417/11/4/1902)
