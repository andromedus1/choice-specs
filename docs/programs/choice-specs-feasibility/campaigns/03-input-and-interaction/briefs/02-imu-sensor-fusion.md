---
description: "IMU sensor-fusion algorithm choice for choice-specs — Mahony recommended; Madgwick fallback; deep-sleep is the load-bearing constraint"
type: brief
campaign: 03-input-and-interaction
program: choice-specs-feasibility
research_method: deep-research
confidence: speculative
status: draft
updated: 2026-04-25
---

# IMU Sensor Fusion Approaches

## Question

Which sensor-fusion algorithm should choice-specs use to convert raw 6/9-DoF IMU samples into a stable orientation estimate? Candidates: complementary filter, Kalman / EKF, Madgwick, Mahony, ML-based (LSTM / 1D-CNN). The choice is constrained by:

- **Compute target** — RK3566 (quad A55, plenty of headroom) or possibly ESP32-S3 (much tighter) if MentraOS path is taken (Campaign 2 will resolve).
- **Latency** — gesture detection must feel instantaneous; budget is ~50–100 ms from head motion to page render.
- **Drift over 60+ minute sessions** — the C3 brief 1 finding: long-session reliability requires drift mitigation regardless of fusion choice, but the fusion choice sets the floor on residual error.
- **Calibration friction** — anything that requires careful tuning makes the device less "sleep-grade reliable."
- **Interaction with deep-sleep** — and this is the load-bearing one. **Sensor fusion typically requires continuous IMU sampling.** If the system can't deep-sleep between page turns, the always-on IMU + fusion power cost dominates the battery budget.

## Findings

### Algorithm comparison

[Comparative Analysis of Sensor Fusion Algorithms (Padova 2023)](https://www.dei.unipd.it/~badia/papers/2023_07_ISITIA), [Madgwick vs Kalman (qsense-motion)](https://qsense-motion.com/qsense-imu-motion-sensor/madgwick-filter-sensor-fusion/), [On the Functional and Extra-Functional Properties of IMU Fusion (PMC 2021)](https://pmc.ncbi.nlm.nih.gov/articles/PMC8069451/):

| Algorithm | CPU (rel.) | RAM | Avg error | Tuning friction | Notes |
|---|---|---|---|---|---|
| Complementary | 1× (lowest) | tiny | ~5° | minimal (1 gain) | simplest, but worst drift behavior |
| **Mahony** | **1.2×** | tiny | **~3.5°** | low (2 gains: P, I) | fastest fusion with PI controller; PI gain handles bias drift |
| Madgwick | 1.5× | tiny | ~3.5° | low (1 gain: β) | gradient-descent based; simpler structure than Mahony |
| EKF / Kalman | 5–10× | matrix ops, 100s of bytes | ~3.0° | high (Q, R matrices) | matrix mults; requires tuning of process / measurement noise |
| LSTM / 1D-CNN | 50–500× | MB | depends on training data | training-set engineering | overfits to training conditions; opaque to debug |

Quantitatively from [Comparative Analysis of Sensor Fusion (PMC)](https://pmc.ncbi.nlm.nih.gov/articles/PMC8069451/): "the error of Madgwick filter and Mahony filter are on average only 0.4° higher than the error from the Kalman filter for an overall average error of 3.4°." For a head-tilt system that needs to discriminate ~15° intentional tilts from <5° pillow noise, all three (Mahony, Madgwick, Kalman) are accurate enough. **Kalman's marginal accuracy gain doesn't justify its 5–10× compute and matrix-tuning cost.**

ML-based approaches are dismissed as overkill for our 4–6 class problem with reasonable IMU SNR. They also work against principle 5 (sleep-grade, boring beats clever): an LSTM is a debugging black box at 2am.

### Recommendation order

**Primary: Mahony filter** at 50–100 Hz IMU sample rate.
- Fastest fusion with the lowest compute footprint per [PMC 2021 IMU fusion comparison](https://pmc.ncbi.nlm.nih.gov/articles/PMC8069451/).
- The PI structure naturally handles gyro bias drift, which is exactly the long-session failure mode we care about.
- Two gains (Kp, Ki) are tunable but defaults from the original Mahony paper work for almost all use cases.
- Easy to override with explicit ZUPT calls during stillness.

**Fallback: Madgwick filter** — equivalent accuracy, slightly higher compute, single tuning gain (β). If the Mahony PI integrator interacts oddly with our calibration scheme, swap to Madgwick.

**Rejected for primary use: complementary filter** — the simplest option but has higher residual drift and less principled bias handling. Acceptable for a quick proof-of-concept but not for the shipping prototype.

**Rejected: Kalman / EKF** — accuracy gain doesn't justify compute and tuning cost for this application. On a more capable CPU (RK3566) Kalman is affordable; tuning friction is the more important rejection criterion for a prototype where we want to iterate gesture vocabulary, not algorithm gains.

**Rejected: ML-based** — opaque, training-data-dependent, hostile to principle 5.

### Library choice

- **Rust:** [`ahrs`](https://crates.io/crates/ahrs) crate provides Mahony and Madgwick implementations; well-maintained; pure Rust, no_std capable. Use this if the firmware is Rust (likely path given C2's expected stack).
- **C/C++:** [`Reefwing-AHRS`](https://github.com/Reefwing-Software/Reefwing-AHRS) (Arduino-friendly) and the original [Madgwick reference C code](https://github.com/xioTechnologies/Fusion). Battle-tested.
- **Python (prototyping):** [`ahrs` Python package](https://github.com/Mayitzin/ahrs) for offline analysis of recorded IMU traces during prototype tuning. Not for on-device.
- **MicroPython on ESP32-S3:** the ESP-NOW-class `mpy` ecosystem has Madgwick ports; Mahony is straightforward to port if needed.

### IMU sensor choice (cross-references C1 hardware list)

C1's component sourcing called out the **Adafruit BNO055** ($35, I²C, 9-DoF). The BNO055 has **on-chip sensor fusion** — it produces a quaternion estimate directly without us running Mahony on the SoC. This is a real consideration:

- **Pro:** offloads fusion compute entirely; SoC just reads the quaternion at 100 Hz.
- **Con:** the on-chip fusion is opaque; we can't ZUPT-on-stillness or inject the per-session calibration except through the chip's sparse calibration API. Long-session drift behavior is determined by Bosch's algorithm, not ours.
- **Verdict:** **use raw IMU mode + our own Mahony** rather than fused mode. We need the explicit control over re-anchor and calibration. The BNO055's on-chip fusion is convenient for upright-posture robotics but constrains us for the supine-specific mitigations we need.

Alternative IMU candidates for choice-specs:
- **Bosch BMI270** ($3, low-power, 6-DoF) — if we control fusion ourselves, the cheaper 6-DoF is fine; we don't need the magnetometer in a bedroom.
- **STMicro LSM6DSOX** ($5, 6-DoF, low-power, ML core for activity detection) — another good option; built-in ML core could optionally run gesture classification.
- **Bosch BMI088** ($8, 6-DoF, automotive-grade low-noise) — overkill but exceptionally low gyro drift; could be the production IMU.

**Recommendation:** **BMI270 for prototype** ($3, well-documented, Linux drivers exist). Drop the magnetometer entirely — we don't need yaw absolute reference indoors. The IMU is mounted above the bridge per C1 inheritance.

### Deep-sleep interaction (the load-bearing constraint)

This is the critical question. Sensor fusion runs continuously; if the SoC can't sleep between page turns, the always-on IMU + fusion cost dominates power.

**Three architectural options:**

#### Option A — SoC always awake, IMU + fusion continuous
- Power cost: ~150 mW typical for RK3566 active-idle + IMU + fusion + framebuffer hold.
- Battery cost: 500 mAh @ 3.7 V = 1.85 Wh; 1.85 / 0.15 = **~12 hours active per charge.**
- Acceptable for 2-hour session + 2-week standby? **No** — standby would need a separate sleep mode.

#### Option B — SoC sleeps between page turns; IMU on-chip wake-on-motion ✅ **Recommended**
- Power cost: ~5 mW SoC-asleep + ~0.5 mW IMU-wake-on-motion = ~5.5 mW idle; ~150 mW active.
- IMU runs in low-power "any-motion" interrupt mode when SoC is asleep. BMI270 supports this natively at <50 µA. Wakes SoC on >threshold motion.
- On wake: SoC runs Mahony for ~1–2 seconds to converge orientation, evaluate gesture, render new page, sleep again.
- Battery cost: assume 80% of session time is "between gestures" (1 page-turn every ~30 sec at 250 wpm). 0.2 × 150 mW + 0.8 × 5.5 mW = **34.4 mW average.** 1.85 Wh / 34 mW = **~54 hours of active reading.** Margin to spare.
- **Crucial caveat: orientation convergence after wake is not instantaneous.** A cold Mahony filter takes ~0.5–2 sec to converge. To bridge this, the IMU's gravity-vector reading at wake (single accelerometer sample) provides initial orientation; Mahony then refines over the next 200 ms.
- **Subtler caveat:** drift accumulated during sleep. If the SoC sleeps for 30 sec between page-turns, the IMU isn't running fusion — but the head also wasn't moving (any-motion would have woken us). On wake, the gravity reading defines a fresh reference. **This actually helps** — sleeping resets accumulated drift.

#### Option C — Always-on MCU coprocessor (e.g., Apollo4 + RK3566)
- A tiny MCU runs IMU + Mahony continuously at <5 mW; wakes the main SoC only on detected gesture.
- Adds BOM cost ($5–10) and code complexity (two firmware images).
- Better than B only if wake latency in B is unacceptable. Empirically B's ~200 ms wake-to-render should feel responsive.
- **Hold in reserve.** Implement only if B's wake latency is unacceptable in prototype testing.

### Recommended architecture (final)

- **IMU:** BMI270 above the bridge, 6-DoF, raw mode (no on-chip fusion).
- **Algorithm:** Mahony filter at 100 Hz when active; pure-accelerometer gravity reading when waking from sleep.
- **Library:** `ahrs` Rust crate.
- **Power architecture:** Option B (deep-sleep between turns; IMU any-motion wake).
- **Calibration:** explicit ZUPT-on-stillness during ≥1 sec pauses; per-session "neutral" capture at session start.

## Recommendation

**Mahony at 100 Hz on RK3566, BMI270 IMU above the bridge, deep-sleep between gestures with any-motion wake.** Use the `ahrs` Rust crate. Estimated average power 30–40 mW during a reading session including all wake/render cycles.

If the path forks to ESP32-S3 + MentraOS (Campaign 2 will decide), Mahony still applies — same crate ecosystem, slightly tighter compute headroom, identical algorithm.

## Open questions / Gaps

1. **Wake-to-orientation latency on real hardware** — the 200 ms estimate is an extrapolation. Bench test on prototype.
2. **Mahony PI gains for our specific IMU + posture** — the literature defaults assume upright. We may need to retune slightly for supine. Defer to bench testing.
3. **BMI270 vs LSM6DSOX vs BMI088** — final IMU choice should be validated by gyro noise floor measurement at <0.1°/s. All three should pass; pick by sourcing convenience.
4. **MentraOS hardware (Vuzix Z100 / Even G1) IMU access** — if the path becomes "skip hardware via MentraOS" we lose control of IMU settings. The supine mitigation stack may not be implementable on shipping AR glasses without OS-level IMU access. **Confirm with C2's MentraOS evaluation.**

## Sources

- [Comparative Analysis of Sensor Fusion Algorithms (Padova ISITIA 2023)](https://www.dei.unipd.it/~badia/papers/2023_07_ISITIA)
- [Madgwick vs Kalman filter — qsense-motion](https://qsense-motion.com/qsense-imu-motion-sensor/madgwick-filter-sensor-fusion/)
- [On the Functional and Extra-Functional Properties of IMU Fusion (PMC 2021)](https://pmc.ncbi.nlm.nih.gov/articles/PMC8069451/)
- [Basic, Madgwick and Mahony comparison (NDSU)](https://web.cs.ndsu.nodak.edu/~siludwig/Publish/papers/SPIE20181.pdf)
- [Reefwing-AHRS (GitHub)](https://github.com/Reefwing-Software/Reefwing-AHRS)
- [Madgwick reference implementation (xioTechnologies)](https://github.com/xioTechnologies/Fusion)
- [`ahrs` Rust crate](https://crates.io/crates/ahrs)
- [Bosch BMI270 datasheet](https://www.bosch-sensortec.com/products/motion-sensors/imus/bmi270/)
- [STMicro LSM6DSOX datasheet](https://www.st.com/en/mems-and-sensors/lsm6dsox.html)
- [Adafruit BNO055 — fused 9-DoF IMU](https://www.adafruit.com/product/2472)
