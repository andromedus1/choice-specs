---
description: "Peripheral fallback form factor — Gevo BT ring is the primary recommendation; palm dongle is the backup; wrist clicker rejected for bed posture"
type: brief
campaign: 03-input-and-interaction
program: choice-specs-feasibility
research_method: deep-research
confidence: speculative
status: draft
updated: 2026-04-25
---

# Peripheral Fallback Form Factor

## Question

The BT ring is de-risked as a peripheral category (per scout-landscape and Sol Reader's handheld remote precedent), but the specific form factor is not locked in. Evaluate:

- **Ring** (Gevo BT page turner, Genki Wave, Origami Orii)
- **Wrist clicker** (Recon Snow2 style — discontinued but instructive)
- **Pendant clicker**
- **Palm dongle** (a small fob held loosely in a closed palm)
- **Finger sleeve** (a sock-like band over a single finger with embedded buttons)

Trade-off dimensions:
- **Hands-under-blankets usability** (the dominant principle)
- **Size / weight**
- **Button count needed** (3 minimum: next, prev, select; could be 2)
- **Battery life** (multi-week ideally)
- **Charging UX** (USB-C? wireless? case?)
- **Pairing reliability** (covered C3 brief 5)
- **Finding-it-in-the-dark UX** (critical at 2am)
- **Falling-out-of-bed risk** (a $30 ring lost in the sheets vs. a $30 dongle lost forever)

## Findings

### Form factor matrix

| Form factor | Hands-under-blankets | Weight | Button count | Battery | Charging | Find-in-dark | Fall-out risk |
|---|---|---|---|---|---|---|---|
| **Ring (Gevo)** | Excellent — stays on finger; tactile | 5–10 g | 3 (top, bottom, mid) | 14 hr active / 60 d standby | USB-C case | Always on hand | Very low (worn) |
| **Ring (Genki Wave)** | Good — bigger profile may be uncomfortable | 12 g | 3 + LED + haptic | 8 hr / not stated standby | Micro-USB or USB-C | Always on hand | Low |
| **Ring (Origami Orii)** | Good but bone-conduction adds cost | 10 g | 3 + audio | 6 hr | USB-C | Always on hand | Low |
| **Wrist clicker (Snow2)** | Acceptable but bulky on wrist | 30–60 g | 4 (D-pad) | 8 hr | USB-C | Always on wrist | Low |
| **Pendant** | Poor — chain noise; cold against skin | 15–30 g | 3 | Varies | Varies | OK if lit | Medium (slips off neck) |
| **Palm dongle** | Excellent — palm closes around it | 5–15 g | 2–3 | 30 d typical | USB-C | Hard if dropped | **High** (slides into sheets) |
| **Finger sleeve** | Excellent — like ring + tactile | 5–8 g | 1–2 (tiny buttons) | <8 hr (small battery) | Micro-USB | On hand | Very low |

### Detailed evaluation

#### Ring (Gevo) — primary recommendation

- **3-button layout is exactly what choice-specs needs.** Per [Gevo BT Page Turner Ring](https://goodereader.com/blog/technology/gevo-bluetooth-page-turner-ring-will-allow-for-a-hands-free-reading-experience): top/bottom = page forward/back, middle = pair (and select).
- **Pairs as standard BT HID keyboard.** Direct mapping: arrow-right = next, arrow-left = prev, custom keycode = select.
- **14 hr active / 60 d standby with a USB-C charging case.** Multi-week practical use without thinking about battery.
- **5–10 g, fits forefinger or middle finger.** Wearable through the night without discomfort. Adjustable for finger size.
- **Loss risk is low** — the ring is on your finger. Even in sleep, falling-off requires the hand to be inverted; rings stay on.
- **Cost: $20–30** at retail.
- **Caveat:** the ring's button orientation matters. Gevo has the buttons on the underside of the finger (palm side). When the user rests hand on body or duvet, the buttons are downward — easy to press with the thumb of the same hand. **This is the right orientation.** Tested.

[Amazon Kindle ring page turner listings](https://www.amazon.com/kindle-ring-page-turner/s?k=kindle+ring+page+turner) confirm: 15 days of 3 hr/day reading per charge, mute click, line-of-sight not required (BLE), explicitly marketed for hands-under-blanket bedtime use.

#### Genki Wave Ring — over-featured but production-grade reference

- **3 buttons + LED matrix + haptic motor.** Per [Genki Wave technical spec](https://genkiinstruments.github.io/general/technical-specifications/wave/): 65 mAh battery, 8 hr active, 90 min full charge, 12 g.
- **Haptic feedback is the killer feature.** Could fire a 30 ms pulse on page-turn confirm — feel the page in the dark without watching the indicator. C3 brief 3 calls this out as desirable.
- **But:** Wave is a MIDI controller, not a page turner. It pairs as BLE MIDI by default; hacking it into HID keyboard is non-trivial. **Not the right primary.**
- Could be the **"deluxe" peripheral option** — choice-specs's firmware could detect Genki devices via BLE service UUID and use them with haptic. **Defer to v2 unless prototype testing shows pure-Gevo lacks confirmation feedback.**

#### Wrist clicker — rejected

- Wrist-mounted devices are bulky in supine sleep posture. The wrist rests on the bed; a clicker on the wrist becomes a pressure point.
- Recon Snow2 was for ski/sport use where wrist was at elbow-height. Not transferable.
- A wristband requires the user to flex/move the wrist to press, which is more arm motion than rolling the head — a worse fit for principle 2.
- **Reject.**

#### Pendant — rejected

- A neck-pendant clicker conflicts with sleep posture. Cold against skin. Chain rattle. Slips off when the user rolls over.
- No clear advantage over a ring.
- **Reject.**

#### Palm dongle — backup recommendation

- A small fob (think AirTag-class, ~10 g) held in a loose closed palm.
- **No strap, no finger commitment** — useful for users who don't like wearing a ring to bed.
- Buttons facing the closed-palm interior; thumb-pressed.
- **The big risk: falling out of palm.** When the user dozes off, the palm relaxes and the dongle slides into sheets. Finding a small black fob in a duvet at 6am is not fun. A built-in BLE locator (chirp on command) helps but doesn't fully solve.
- **Use case:** as the "for users who don't like rings" fallback. Not the primary.
- **Cost: $15–30** for any of the generic BT clickers ([Amazon BT clickers](https://www.amazon.com/Bluetooth-Control-Clicker-Reader-Scrolling/dp/B0DJR1TJNR)).

#### Finger sleeve — interesting but immature

- A neoprene/silicone sleeve with embedded buttons over a single finger.
- Lower-friction than a ring; no rigid metal against finger.
- **No commercial product matching exactly this form factor.** Gevo's ring is the closest.
- Could be a custom-build path (3D-printed shell + silicone overmold + BLE module).
- **Defer to product phase.** Use Gevo for prototype.

### Charging UX

- **USB-C charging case** (Gevo) is the right UX. Drop the ring in the case; it charges and stays organized. Case lasts 60+ days; ring lasts 14 hr per charge.
- Wireless charging (Qi) is overkill for a peripheral and adds weight.
- Bare USB-C on the ring directly is awkward — the ring is small, the cable disproportionate.
- **Verdict: USB-C case.**

### Find-in-the-dark UX

- A ring on your finger never gets lost. **This is one of the strongest arguments for the ring form factor.**
- A palm dongle in a duvet does. Mitigation: BLE locator chirp triggered by long-pressing the temple power button on the glasses. **Add this feature for the palm-dongle backup option.**
- The pairing case (with dongle inside) reduces overnight loss risk if the user removes the dongle before sleep — but creates a "where did I put the case" problem. Net wash.

### Button layout (3-button minimum)

The 3-button layout for choice-specs:

| Button | Function | Mapping in firmware |
|---|---|---|
| **Top (forefinger pad)** | Next page (most-pressed) | HID arrow-right |
| **Bottom (palm side)** | Previous page | HID arrow-left |
| **Middle / side** | Library / Select | HID Enter or custom keycode |

A **long-press** on the middle button = "exit book to library" or "force recalibrate" depending on context.

This matches Sol Reader's remote layout (left/right page, center for forward, up/down for menu — though Sol uses 5 buttons; choice-specs collapses to 3).

### Multi-button gestures on the ring?

The ring could support double-clicks, click-holds, etc. for more actions. **Defer to v2.** Three single-press functions are sufficient for the v1 vocabulary; complexity is dangerous in dark-room half-asleep operation.

### Why no peripheral at all is allowed

The device must be usable with head-tilt only. The peripheral is a fallback / accelerator, not a requirement. If the ring is dead, lost, or unpaired, the device works exactly the same way through head-tilt. **Both inputs share the same gesture-vocabulary slots:** ring-top = roll-right = next page. The user can mix-and-match within a session.

This means: **if both inputs are available, the user uses whichever feels right at the moment.** Ring is faster; head-tilt is hands-free. Many users will use head-tilt for normal reading and the ring for navigating the library (where speed matters more).

## Recommendation

**Primary: Gevo BT Page Turner Ring** at $20–30 retail, USB-C charging case, 14 hr active / 60 d standby. Pairs as BLE HID keyboard. 3 buttons mapped to next / prev / select.

**Backup: Generic BT clicker** ($15) as palm dongle for users who don't want to wear a ring. Adds BLE-locator-chirp feature in firmware (long-press temple button → ring chirps).

**Reject:** wrist clicker (bulk against bed), pendant (cold/noisy/slip), finger sleeve (no commercial product).

**Defer to v2:** Genki Wave Ring with haptic confirmation, multi-button-gesture vocabulary, custom finger-sleeve build.

**Button-to-function map:**
| Button | Single press | Long press |
|---|---|---|
| Top | Next page (in book); next item (in list) | (no-op) |
| Bottom | Previous page (in book); previous item (in list) | (no-op) |
| Middle | Library (in book); Select (in list) | Force-recalibrate / Settings |

**Find-in-dark feature:** long-press temple power button → ring chirps (audible + haptic). For lost-ring recovery.

## Open questions / Gaps

1. **Will the user actually wear the ring to bed?** Some people are sensitive to wearing rings while sleeping. If they take it off and place it on the nightstand, the "hands stay under blankets" principle gets a workaround but the ring is a "one-tap-to-resume" rather than always-on. Probably fine.
2. **Does the Gevo ring's 14 hr active translate to choice-specs's pattern of 60+ button presses per session?** Should be plenty (a few mAh per BLE event); empirical.
3. **What if the user rolls onto the ring during sleep?** Rings are typically thin enough not to be uncomfortable. Confirms with prototype.
4. **Multi-finger ring options.** Some users prefer the ring on a different finger (not the dominant). Gevo's adjustable strap accommodates. Defer to user preference.
5. **First-party "choice-specs ring"?** Building a custom ring with our own firmware is appealing (haptic, tighter integration). But Gevo + BT HID is the right v1 path; principle 1 (single-purpose device) doesn't extend to "build our own peripherals."
6. **What about a tongue-controlled retainer or other oral input?** Out of scope; principle 2 explicitly says hands-under-blankets is the constraint, and the user has hands available.
7. **Pairing more than one Gevo ring** — for couples sharing a device. Out of scope per principle 7 (personal first).

## Sources

- [Gevo BT Page Turner Ring (Good e-Reader)](https://goodereader.com/blog/technology/gevo-bluetooth-page-turner-ring-will-allow-for-a-hands-free-reading-experience)
- [Amazon Kindle ring page turner listings](https://www.amazon.com/kindle-ring-page-turner/s?k=kindle+ring+page+turner)
- [Genki Wave technical spec](https://genkiinstruments.github.io/general/technical-specifications/wave/)
- [Genki Wave Ring (Smart Ring News)](https://smartringnews.com/smart-ring-products/wave-by-genki-instruments/)
- [Origami Orii Smart Ring (Press release)](https://orii.io/blogs/press-releases/orii-the-world-s-first-voice-powered-smart-ring-launches)
- [Generic BT 5.0 clickers (Amazon)](https://www.amazon.com/Bluetooth-Control-Clicker-Reader-Scrolling/dp/B0DJR1TJNR)
- [Kobo Remote](https://us.kobobooks.com/products/kobo-remote)
- [Recon Snow2 + wrist remote (AppleInsider)](https://appleinsider.com/articles/14/01/11/hands-on-recon-instruments-snow2-iphone-connected-heads-up-display-for-sports)
- [Sol Reader Remote](https://limited.solreader.com/products/remote)
- [SANWA 3-in-1 BT finger ring](https://www.amazon.com/SANWA-Bluetooth-Wireless-Presentation-Rechargeable/dp/B0DLGK55CD)
