---
description: "Accessibility input prior art — five patterns to adopt: dwell timing, refractory windows, scan-and-select, inverse-undo, fail-quiet"
type: brief
campaign: 03-input-and-interaction
program: choice-specs-feasibility
research_method: deep-research
confidence: speculative
status: draft
updated: 2026-04-25
---

# Accessibility Input Prior Art Deep Dive

## Question

Accessibility tech has been solving "input without hands" for decades. Choice-specs is in the same problem space — input must function without hand or arm motion, in a constrained sensory environment, with high reliability for an exhausted user. What patterns from accessibility tech transfer? Which are direct match, which are instructive, which don't apply?

Specifically interested in:
- **Dwell timing.** Too short = false positives; too long = annoying.
- **Confirmation patterns** for irreversible actions.
- **Recovery / undo** patterns.
- **Calibration UX** for users who can't be expected to do precise setup.
- **Fail modes** — what does the system do when input is ambiguous?

## Findings

### A. GlassOuse (head-mouse with bite-switch)

[GlassOuse V1.4](https://glassouse.com/) is the most direct prior-art match: glasses-worn 9-axis gyroscope head-tracker for accessibility users, BLE, 150-hour battery. The wearer moves a cursor with head motion; clicks via:
- **Bite switch** on a 3.5mm jack (most popular).
- **Dwell click** ("hold head still 0.5 sec → click").
- **Sip/puff** or other adaptive switches.

[GlassOuse V1.4 manual](https://glassouse.com/wp-content/uploads/2022/09/GlassOuse-V1.4-user-manual.pdf) details the calibration: "A short calibration process ensures that the device adapts to the user's needs to generate a 100% movement and selection accuracy."

**Key transferable patterns:**

1. **Dwell-click default at 500 ms.** Configurable. Below this, hand/head jitter causes false clicks; above 700 ms, users report frustration. The 400 ms compound-hold in C3 brief 3 is in the same ballpark — maybe could go to 500 ms.
2. **The calibration is a "short process," not a setup screen.** UX-wise, GlassOuse normalizes calibration as part of donning. Choice-specs's 1.5 sec auto-calibration matches.
3. **The click is always the hard part.** Pure head-tracking handles cursor movement well; the *fire* event is where false positives live. **Choice-specs sidesteps this by making the gesture itself BE the action** — there's no separate click. A roll-right *is* the page turn. This is dramatically simpler than GlassOuse's split.

**Lesson #1 — Dwell timing in the 400–600 ms range is the empirical sweet spot.** Confirms the C3 brief 3 choice.

### B. Tobii eye trackers / dwell click

[Tobii Dynavox dwell-click guides](https://us.tobiidynavox.com/blogs/support-articles/how-do-i-adjust-gaze-interaction-settings-within-all) describe configurable dwell-time selections — "less than one second" is typical. Eight discrete levels are exposed.

**Key transferable patterns:**

1. **Dwell time is per-user tunable.** Some users prefer 300 ms; some 1000 ms. Choice-specs should expose a dwell-time setting (default 400 ms; adjustable 300–600 ms in settings).
2. **Dwell + visual countdown.** Tobii draws a circle that fills as the user gazes. The user can pull away if they didn't intend to select. Choice-specs's gesture-feedback dot on the page corner serves the same function — visually grows during the 400 ms hold, allowing abort by moving head before fire.

**Lesson #2 — A visual progress indicator during dwell is critical.** Without it, dwell feels like Russian roulette. Choice-specs already builds this in (C3 brief 3 corner-dot indicator).

### C. iOS / Android Switch Control (single-button scanning)

[Apple Switch Control](https://support.apple.com/en-us/119835), [Android Switch Access](https://support.google.com/accessibility/android/answer/6301497?hl=en): single-button users navigate iOS/Android via timed scanning — items highlight in sequence; user presses at the desired item.

**Key transferable patterns:**

1. **Group/row-column scanning** scales single-button input to arbitrary lists. Choice-specs probably won't go to single-button (head-tilt provides 4 inputs, BT ring 3 buttons), but the pattern matters.
2. **Scanning is exhausting for long lists** — users prefer multi-step input where available. Confirms choice-specs's use of head-tilt next/prev (multi-step) over a single-button scan-and-select.
3. **Auto-scan timeout is a critical friction point.** Too fast → users miss; too slow → users give up. Defaults: 1.5 sec per item is iOS's choice. Choice-specs's hold-to-fast-scroll (C3 brief 6) has a similar pacing question.

**Lesson #3 — When list cardinality grows, paged scrolling beats per-item scanning.** Choice-specs's 6-item-per-page library list with half-page advance is the right pattern.

### D. Sip-and-puff, bite-switch, jaw-clench, breath-hold

For users with no hand or head control, alternative effectors (sip-and-puff, jaw clench, breath, blinks) provide fire events. None of these apply directly to choice-specs (the user has full hand/head control by the design constraint), but the *pattern* of "binary fire event" decoupled from "navigation event" is instructive.

[Recent breath-control work](https://dl.acm.org/doi/10.1145/3411764.3445065): users found sustained-blow gestures fatiguing within 5 minutes. Lesson: **any fire event must be brief.** A 400 ms hold is at the edge.

**Lesson #4 — Long sustained efforts don't scale to multi-hour sessions.** 60 page-turns/hour × 400 ms hold = 24 sec/hour of "held effort." Acceptable. If we lengthened to 1 sec, that's 60 sec/hour — perceptibly fatiguing.

### E. Brain-computer interfaces (Neurable, Emotiv)

Consumer BCIs (Neurable's MW75 headphones with EEG; Emotiv's Insight) use slow-cortical-potential or P300 paradigms for "select" events. Latency is multi-second. **Not relevant for choice-specs** — IMU is faster, cheaper, more reliable.

But the BCI literature does emphasize: **session-start calibration is the dominant friction**. BCI users abandon devices that require >2 minutes of setup per session. Choice-specs's 1.5 sec calibration is well below this threshold.

### F. Head-tracking accessibility software (Camera Mouse, Smyle Mouse)

[Camera Mouse](https://www.cameramouse.org/) and similar: webcam-based head-tracker for cursor control on desktop. Free, mature. Click via dwell at user-configurable timing.

**Key transferable insight:** drift is a chronic complaint in this category. Users describe needing to "re-center" the cursor mid-session. Choice-specs's auto-recalibration (C3 brief 4) addresses this.

### G. The "fail quiet" principle

[OpenAssistive](https://openassistive.org/) and accessibility-device design literature converge on: **when input is ambiguous, do nothing rather than guess.** False positives are far more damaging than missed inputs because they require recovery.

Concretely:
- If the gesture is ambiguous (between roll-right and pitch-up), **fire neither**.
- If calibration is uncertain (no recent ZUPT confirmation), **prefer to not fire**.
- If BLE link drops mid-session, **continue with head-tilt; never crash; never show error UI**.

This is consonant with principle 5 (sleep-grade reliability) — boring beats clever, missed-input is fine, accidental-input is a problem.

### H. Inverse / undo gestures

Across accessibility tech, the rule is: **for any potentially-irreversible action, require either confirmation or trivial undo.** Choice-specs achieves this by making the gesture set self-inverse:

- Roll-right → next page; roll-left → previous page (immediate undo).
- Pitch-up → exit book; pitch-up again → re-exit (returns to library); selecting the book in library → re-enters at last position (full reversibility).
- No "delete book" gesture exists. Library deletion happens off-device.

**Lesson #5 — Designing every action to have a one-gesture inverse eliminates the need for explicit confirm-irreversible flows.** Confirms C3 brief 3 vocabulary.

### Pattern-by-pattern transfer summary

| Accessibility pattern | Choice-specs adoption |
|---|---|
| 400–600 ms dwell timing | ✅ adopt (400 ms hold per C3 brief 3) |
| Visual countdown during dwell | ✅ adopt (corner-dot growing per C3 brief 3) |
| Per-user dwell-time setting | ✅ adopt (Settings: dwell time, 300–600 ms slider) |
| Per-session calibration (1–2 sec) | ✅ adopt (1.5 sec, C3 brief 4) |
| Auto-recalibration on drift | ✅ adopt (sustained-deviation re-anchor, C3 brief 4) |
| Switch scanning for navigation | ❌ skip (we have 4 directional gestures, no need to scan) |
| Sustained-effort gestures (blow, hold) | ❌ avoid (any held effort >500 ms fatigues at session length) |
| BCI fire events | ❌ irrelevant |
| "Fail quiet" on ambiguity | ✅ adopt (gesture state machine returns to IDLE on cancel) |
| Self-inverse gesture set | ✅ adopt (roll-right ↔ roll-left; pitch-up exits both directions) |
| No irreversible actions in primary UX | ✅ adopt (no delete, no confirm flows in v1) |

## Recommendation

Adopt these five accessibility-tech patterns:

1. **Dwell timing in 400–600 ms with visual countdown.** Default 400 ms. User-tunable in Settings.
2. **Brief per-session calibration as part of donning.** ≤2 sec. Auto-recalibrate on sustained deviation.
3. **Fail quiet on ambiguous input.** Never guess; always return to IDLE; let the user re-issue.
4. **Self-inverse gesture set.** Every action has a one-gesture undo path; no "are you sure?" flows needed in v1.
5. **No sustained-effort gestures.** Hold durations capped at ≤500 ms cumulative; no blowing, no clenching, no minute-long dwells.

**These patterns reinforce decisions already made in C3 briefs 3 and 4.** The accessibility-tech survey doesn't introduce new mechanisms — it validates the gesture vocabulary and timing budget choices.

## Open questions / Gaps

1. **Are there UX patterns from sleep-tech accessibility (e.g., sleep mask + accessibility) that we missed?** Probably not — the intersection of "wear at bed" + "input without hands" is essentially empty.
2. **Should choice-specs ship with multiple gesture vocabularies?** A "low-mobility profile" (longer dwells, simpler gestures) and a "default profile" (current). Out of scope for v1.
3. **Multi-modal input fusion** — could BT ring + head-tilt complement each other (e.g., ring confirms, head-tilt navigates)? Already implicit in C3 brief 6's input mapping. Worth explicitly testing in prototype.
4. **What about a phone-app-mediated accessibility configuration tool** for those who want fine-grained tuning? Out of scope for v1; phone-tether-free is principled.

## Sources

- [GlassOuse V1.4 (manufacturer)](https://glassouse.com/)
- [GlassOuse V1.4 User Manual (PDF)](https://glassouse.com/wp-content/uploads/2022/09/GlassOuse-V1.4-user-manual.pdf)
- [Tobii Dynavox Gaze Interaction Settings](https://us.tobiidynavox.com/blogs/support-articles/how-do-i-adjust-gaze-interaction-settings-within-all)
- [Tobii Dynavox I-Series User Manual](https://download.mytobiidynavox.com/I-Series/documents/I-Series_User_manual/I-13%20and%20I-16/Tobii%20Dynavox%20I-Series%20I-13_I-16%20Users%20Manual_v1-0-2_en-US.pdf)
- [Apple Switch Control](https://support.apple.com/en-us/119835)
- [Android Switch Access](https://support.google.com/accessibility/android/answer/6301497?hl=en)
- [iOS 14 Switch Control User Guide (AbleNet PDF)](https://www.ablenetinc.com/content/html/Downloads/Switch_Downloads/ios_14_switch_control_user_guide_english.pdf)
- [tecla — Switch Control mode in iOS](https://gettecla.com/blogs/news/15538916-what-is-switch-control-mode-in-apples-ios)
- [Camera Mouse (Boston College)](https://www.cameramouse.org/)
- [BlyncSync (CHI 2020)](https://dl.acm.org/doi/10.1145/3313831.3376132)
- [99% Invisible — Single Handed: E-Ink Devices & Peripherals](https://99percentinvisible.org/article/accessible-ereader-accessories/)
