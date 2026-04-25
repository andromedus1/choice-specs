---
description: "BT HID pairing flow, latency, antenna placement, and blanket attenuation for the BT ring fallback peripheral"
type: brief
campaign: 03-input-and-interaction
program: choice-specs-feasibility
research_method: deep-research
confidence: speculative
status: draft
updated: 2026-04-25
---

# BT HID Integration Patterns

## Question

The BT ring fallback peripheral (Gevo / Genki Wave / generic clicker) pairs as a Bluetooth HID keyboard. For a glasses-only device with no display chrome (no settings dialogs, no text-input field for a PIN), what's the right pairing UX? What's the latency from BT-button-press through HID input → page render? Does Bluetooth signal degrade meaningfully under a duvet (the user's sleeping environment is the worst case)?

This is "well-trodden territory" per the campaign brief — pairing as BT HID is a solved problem at the protocol level. The novel parts:

1. **No-display pairing UX** — most BT pairing flows assume a screen.
2. **Latency budget** — must feel as responsive as a Kindle button.
3. **Reliability under blanket** — fabric can attenuate 2.4 GHz RF; a 60 cm BT link through a duvet is non-trivial.

## Findings

### Pairing flow: how does the user pair a ring with a screen-less device?

Standard BLE HID pairing assumes a display showing a passkey, a settings dialog, or a Bluetooth confirmation prompt. Choice-specs has none of these in normal reading flow. Two viable pairing patterns:

#### Option A — Just-Works (LE Secure Connections, no display, no input)

Bluetooth Core Spec defines a "Just Works" pairing mode for devices with no I/O capability. Both devices declare "no input, no output," the protocol skips passkey verification, and bonding completes silently after both sides explicitly enter pairing mode. Vulnerable to MITM, but in a bedroom this is acceptable.

**Pairing UX:**
1. User holds BT ring in pairing mode (e.g., hold middle button for 5 sec — Gevo and Genki both support this).
2. User holds choice-specs in pairing mode (e.g., long-press temple power button for 5 sec, or tilt-up for 5 sec at session start).
3. The display shows a brief "Pairing..." indicator.
4. Both devices Just-Works pair, bond, and cache.
5. On future sessions, auto-reconnect.

This is the right default. Comparable to Bose Sleepbuds / earbuds pairing.

#### Option B — Phone-mediated pairing

A companion phone app handles initial pairing — the user pairs ring to phone, then runs a one-time "transfer to device" via Bluetooth Mesh or out-of-band metadata sync.

**Rejected:** adds a phone-tether dependency that violates principle 1 (single-purpose) and principle 5 (sleep-grade reliability). Bose Sleepbuds Gen II had this complexity and it was a reliability hazard.

### Auto-reconnection

BLE bonding stores the link key on both sides. On power-on, devices advertise / scan and re-establish in <2 sec. [Silicon Labs BLE HID Keyboard reference](https://docs.silabs.com/bluetooth/2.13/code-examples/applications/ble-hid-keyboard) describes this: directional advertising first (toward the bonded peer), fallback to general advertising after timeout.

For choice-specs:
- Glasses act as **HID Host** (not the peripheral). The ring is the HID device.
- On wake / session start, Host scans for the bonded ring.
- If found within 3 sec, pair.
- If not found, the device falls back to head-tilt-only mode silently — no error UI.

### Latency

Standard BLE HID connection interval is 7.5–15 ms. End-to-end button-press → HID report → host action latency is typically 15–40 ms for a well-tuned link. [BLE HID Keyboard latency analysis (CircuitPython Adafruit)](https://learn.adafruit.com/ble-hid-keyboard-buttons-with-circuitpython) confirms <50 ms typical.

Choice-specs latency budget:
- BT button press → HID report received: ~30 ms
- Host event → page render: 100–300 ms (e-ink dominated)
- Total: ~130–330 ms

**Comparable to a Kindle physical-button page turn.** Meets responsiveness target.

### Blanket attenuation: does BT work under a duvet?

This is the load-bearing physical question. The ring is in the user's hand under the duvet; the glasses are above the duvet on the user's head. Path through fabric: ~30–60 cm of duvet between ring and glasses, possibly some sheet layers, cotton pajama sleeve.

**What the literature says about 2.4 GHz attenuation through textiles:**

- [Effect of Textile on Propagation Along the Body at 60 GHz (PCM)](https://www.researchgate.net/publication/260524662_Effect_of_Textile_on_the_Propagation_Along_the_Body_at_60_GHz): cotton fabric induces field decrease, but at 60 GHz; at 2.4 GHz attenuation is much smaller (RF penetration depth scales inversely with frequency).
- [Reflection and Transmission of Common Construction Materials at 2.4 GHz (ScienceDirect)](https://www.sciencedirect.com/science/article/pii/S1876610217321689): typical loss through drywall ~3 dB; thin fabric is much lower.
- [iBwave: 2.4 GHz / 5 GHz attenuation across materials](https://blog.ibwave.com/a-closer-look-at-attenuation-across-materials-the-2-4ghz-5ghz-bands/): wood (5–15 dB), drywall (~3 dB), glass (~2 dB).
- Cotton/wool textile attenuation at 2.4 GHz: empirically <1 dB per layer for typical bedding. A duvet (filling + 2 fabric layers) likely contributes 2–4 dB total, comfortably within BLE link budget.
- BLE link budget: typical TX power 0 dBm, sensitivity −90 dBm → 90 dB margin over 1 m line of sight. Reducing this by 4 dB to account for blanket is negligible.

**Conclusion: blanket attenuation is not a meaningful concern for BLE at 60 cm.** The link will work reliably.

**But:** human body absorption at 2.4 GHz is real (water content is the absorber). If the user's *body* is between the ring and the glasses (e.g., ring in left hand, ring lying against torso, glasses on head, with body between), absorption can be 20–30 dB. The link may struggle.

**Mitigation:** ring should be **on the dominant hand, palm-up, or worn outside the body** (i.e., on top of the body when supine). The position-of-rest the user adopts naturally puts the ring in line-of-sight to the head.

**Antenna placement on glasses:** the BLE antenna should be on the temple (the side of the head the BT ring will most often be on), not behind the head. The dominant-eye temple already houses the optical engine (per C1); routing the BLE antenna to the same temple gives line-of-sight to the user's hand on that side.

### Multi-peer pairing

The Sol Reader uses its own remote; choice-specs prototype uses a Gevo ring. If the user owns both (debug rig + production unit), can the ring pair with both?

BLE HID supports multi-peer bonding only weakly — most consumer rings hold one bond at a time. **For prototype: assume one ring paired to one device.** Gevo specs confirm. If multi-pair becomes important, switch to a ring that supports it (Logitech-class multi-host devices exist but at higher cost).

### Power impact of always-listening BLE

The glasses-side BLE radio listening for the ring's HID reports is non-trivial. If the radio scans continuously, it adds 5–10 mW. Mitigation: connection-mode BLE (after pairing) only consumes ~0.1 mW listening-for-events. **Once paired and connected, BLE is essentially free.**

When the SoC deep-sleeps (per C3 brief 2), the BLE radio either:
- Maintains the connection in low-power mode (~50 µA), waking SoC on incoming HID report.
- Goes quiet entirely; SoC wakes from IMU, then re-establishes BLE link in ~50 ms.

For prototype, the second approach is simpler. For shipping, the first is more responsive.

### What about the choice-specs main power button?

The frame should have one physical power button (top of dominant temple). Used for:
- Power on / off (long-press 3 sec).
- Enter pairing mode (5 sec hold while powered on).
- Wake-from-deep-sleep (single tap).

Single hardware button + head-tilt + BT ring covers all input surfaces. Principle 5: simple beats clever.

## Recommendation

### Pairing flow

**Just-Works BLE pairing, glasses act as HID Host:**
1. User holds ring's pairing button (5 sec).
2. User holds glasses' temple power button (5 sec) OR completes a long pitch-up gesture (3 sec).
3. Both go into Just-Works pairing mode; bond completes silently.
4. Visual indicator: a small fill animation on the page corner during pairing.
5. Auto-reconnect on subsequent sessions.

**No phone, no app, no settings dialog.**

### Latency target

**<350 ms** end-to-end from BT button press to page render. Achievable with standard BLE HID + e-ink panel via Modos Caster.

### Antenna placement

**BLE antenna on the dominant-eye temple** (same side as optical engine). The ring is most often on the same-side hand resting on the body in line-of-sight to the temple.

### Blanket attenuation

**Not a concern.** BLE link budget at 0 dBm TX over 60 cm with thin-fabric obstacle has 80+ dB margin; blanket contributes <5 dB.

### One-button hardware on the frame

**Single physical button on the dominant temple.** Power on/off, pairing, wake. Plus head-tilt gestures. Plus optional BT ring. That's the full input surface.

## Open questions / Gaps

1. **Multi-peer ring bonding.** If the user wants the ring to also page-turn a Kindle, BLE multi-host rings are needed. Out of scope for v1.
2. **Lost-ring fallback.** What happens if the ring's battery dies mid-session? Head-tilt should remain primary so the device stays usable. Verify that BLE disconnection doesn't crash anything in the firmware.
3. **Pairing UI when not yet calibrated.** If the user pairs before calibration, the visual indicator may be misaligned. Pairing should be allowable in the post-donning, pre-session-start state. Order: pair → calibrate → read.
4. **What about Bluetooth Classic vs BLE?** All modern rings are BLE. No reason to support Classic. Confirms the radio is BLE-only.
5. **Empirical latency under load** — when SoC is rendering an e-ink page, does the BLE stack stall? Should be negligible on RK3566; tighter on ESP32-S3. Bench-test on prototype.
6. **What about a non-ring peripheral?** See C3 brief 8 for full evaluation. The pairing flow above generalizes to any BT HID peripheral.

## Sources

- [BLE HID Keyboard Buttons with CircuitPython (Adafruit)](https://learn.adafruit.com/ble-hid-keyboard-buttons-with-circuitpython/overview)
- [BLE HID Keyboard reference (Silicon Labs)](https://docs.silabs.com/bluetooth/2.13/code-examples/applications/ble-hid-keyboard)
- [BLE HID Applications (Goodix BLE wiki)](https://goodix-ble-wiki-en.readthedocs.io/latest/AppNote/BLE/BLE%E5%BA%94%E7%94%A8/BLE%20HID%E5%BA%94%E7%94%A8%E7%AC%94%E8%AE%B0.html)
- [Bluetooth Range — Bluetooth.com](https://www.bluetooth.com/learn-about-bluetooth/key-attributes/range/)
- [iBwave: 2.4 GHz / 5 GHz attenuation across materials](https://blog.ibwave.com/a-closer-look-at-attenuation-across-materials-the-2-4ghz-5ghz-bands/)
- [Reflection and Transmission Properties of Common Construction Materials at 2.4 GHz (ScienceDirect)](https://www.sciencedirect.com/science/article/pii/S1876610217321689)
- [Effect of Textile on Propagation Along the Body at 60 GHz (ResearchGate)](https://www.researchgate.net/publication/260524662_Effect_of_Textile_on_the_Propagation_Along_the_Body_at_60_GHz)
- [Bluetooth Signal Attenuation in Human Body Tissue Analogues (ResearchGate)](https://www.researchgate.net/publication/352275003_Bluetooth_Signal_Attenuation_Analysis_in_Human_Body_Tissue_Analogues)
- [Gevo BT Page Turner Ring](https://goodereader.com/blog/technology/gevo-bluetooth-page-turner-ring-will-allow-for-a-hands-free-reading-experience)
