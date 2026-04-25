---
description: "Reflective and transflective LCD options at microdisplay scale — Sharp Memory LCD, JDI MIP, IGZO transflective"
type: brief
campaign: 01-display-and-optics
program: choice-specs-feasibility
research_method: deep-research
confidence: speculative
status: draft
updated: 2026-04-25
---

# Reflective LCD Options

## Question

Daylight Computer's DC-1 demonstrates that a reflective IGZO-LCD panel at 10.5" can be a credible "paper-like" display alternative to e-ink with the bonus of video-rate refresh. Hisense Q5 ships a 10.5" RLCD tablet using the same approach. Both products are e-reader-class but no one has shipped this technology at glasses scale. This brief asks: does any vendor make a microdisplay-scale reflective LCD? Sharp Memory LCD (Pebble) and JDI MIP transflective panels exist at small sizes — are any usable for choice-specs at acceptable resolution and contrast?

## Findings

### Reflective LCD: the technology

A reflective LCD has the standard polarizer-LC-polarizer-color-filter stack but replaces the backlight with a reflector behind the back polarizer. Ambient light enters the panel, passes through the LC layer (which modulates polarization per pixel), bounces off the rear reflector, passes back through the LC layer, and exits through the front polarizer to the eye. **No emission, no backlight, paper-like response to ambient light.** Power can be very low because the only active element is the LC drive, not a backlight.

The Daylight Computer / Hisense path uses an IGZO (indium-gallium-zinc-oxide) TFT backplane instead of standard a-Si silicon — IGZO supports much lower-frequency drive (down to ~1 Hz for static content) without flicker, dramatically reducing power. [Daylight DC-1 vs e-ink (Tom's Guide)](https://www.tomsguide.com/tablets/android-tablets/daylight-dc1-is-a-new-tablet-with-a-live-paper-display-that-might-be-better-than-e-ink-heres-how)

### Sharp Memory LCD (Pebble's panel)

Sharp's "Memory LCD" line is a reflective monochrome panel with **1-bit pixel memory** (each pixel stores its own state, no continuous refresh required). Power is quoted as 1/40 to 1/80 of a standard STN-LCD and 1/1000 of an AM-TFT LCD. [Sharp Memory LCD](https://sharpdevices.com/memory-lcd/) Pebble used the 1.26" 144×168 part for a static-content smartwatch with multi-day battery life.

Available sizes today (per [Panox Display catalog](https://www.panoxdisplay.com/transflective/) and [Sharp's own catalog](https://sharpdevices.com/memory-lcd/)):
- **1.26" 144×168** — the Pebble part. Still in production. ~$40–60 retail.
- **1.28" 176×176** — Memory in Pixel (MIP) variant.
- **2.7" 400×240**, **3.16" 432×240**, **4.4" 480×400** — larger but still small.

**Resolution is the killer.** 144×168 across a panel magnified by a pancake lens to fill the FoV is ~50 ppi virtual — far below the ~300 ppi Sol Reader virtual or ~150 ppi DC-1. **Memory LCD is too low-resolution for reading.** It's a smartwatch / industrial-instrument display.

### Japan Display Inc (JDI) MIP transflective TFT-LCD

JDI's MIP (Memory in Pixel) line is the closer-to-reading-grade reflective option. Active matrix TFT with embedded per-pixel memory. [JDI MIP Displays at Youritech](https://www.youritech.com/products/mip-displays/)

Available sizes:
- **1.28" 176×176** (LPM013M126A/C) — small but watch-class resolution. [JDI 1.28" MIP at Youritech](https://www.youritech.com/products/lpm013m126a-jdi-1-28-inch-reflective-display-176x176-sunlight-readable-3-wire-spi-memory-in-pixel-display.html)
- **1.28" 205×148 transflective** (LPM012A220A) — slightly higher resolution at the same size. [LPM012A220A datasheet at Toppoplcd](https://www.toppoplcd.com/productdetails_5258051.html)
- **1.34" 320×300 round** (LPM013M091A) — for round smartwatches.
- **3.2" 336×536** — larger; reasonable PPI.

These panels are sourceable in single units from Youritech, Mouser, Panox Display, and Alibaba at $20–80. **Resolution at 1.28" tops out around 320 PPI native (LPM013M091A) but only at sub-1.34" sizes.** Magnified through a pancake lens, that's still below Sol Reader's effective virtual PPI.

### IGZO microdisplay: the gap

IGZO transflective at glasses scale **does not exist as a commercial product.** Sharp ships Reflective IGZO LCDs at industrial / business sizes (15"+ datasheet panels). [Sharp Reflective IGZO](https://sharpdevices.com/reflective-lcd/) Daylight Computer uses a 10.5" panel; Hisense Q5 the same. **There is no commercial sub-2" IGZO reflective panel.** This is a real gap in the supply chain — IGZO + small form factor + reflective is unbuilt.

Why? IGZO displays the manufacturing complexity of TFT-LCD plus the complexity of a reflective stack, and the volume markets (e-readers, signage) prefer e-ink. The IoT/wearable market that *would* use small reflective panels has been fully captured by Memory LCD (Sharp) and MIP (JDI) at lower resolution.

### Hisense Q5 / Daylight DC-1 panel sourcing

Both ship 10.5" panels, almost certainly same supplier (BOE or Truly, sourced via the Hisense relationship). The panel itself is not sold separately as a development part. eBay occasionally lists Sharp LS315M7JX01 (Reflective IGZO 31.5" — too large). [Sharp LS315M7JX01 on eBay](https://www.ebay.com/itm/116648565214) The ~10.5" panel doesn't appear in bulk distributor channels.

**No sub-3" IGZO reflective panel surfaced.** This brief confirms a real category gap.

### Color reflective LCD

Color reflective LCD has the additional problem of color-filter loss (~⅓ light penalty per color subpixel) on top of reflective inefficiency. JDI's MIP color line tops at 1.34" 320×300 (LPM013M091A). Color contrast is poor (typical 5:1 vs. 12:1 for monochrome) and ambient-light dependent. **Not a credible bedtime path** because in a dark room there is no ambient to reflect — the entire premise of a reflective display fails.

This is the bedtime-specific objection that matters most: **a reflective display is fundamentally wrong for a dark-room use case** because there's no light to reflect. We'd need a frontlight (like Kindle Paperwhite's edge-lit waveguide) to make any reflective panel work in bed. At that point we're combining "reflective panel" + "active illumination" — which is roughly the same architecture as LCoS (Brief 2) but with worse contrast, worse PPI, and the same illumination complexity. **The bedtime case kills the reflective LCD path the same way it kills any pure-reflective tech.** The *only* reflective tech that wins is e-ink, because e-ink is the only one with a mature integrated front-light supply chain (Kindle, Kobo, Boox).

### Comparison summary

| Tech | Smallest commercial size | Native PPI | Bedtime (dark room) | Refresh | Hobbyist sourcing |
|---|---|---|---|---|---|
| Sharp Memory LCD | 1.26" 144×168 | ~135 | Needs frontlight | Slow / static | Easy |
| JDI MIP TFT | 1.28" 320×300 | ~330 | Needs frontlight | Static / 30 Hz w/ external drive | Easy |
| IGZO RLCD (Daylight class) | 10.5" | ~150 | Needs frontlight | 60–120 Hz | Hard (panel not separable) |
| E-ink (for comparison) | 1.43"+ Carta | ~300 | Front-lit panels available | 1–10 Hz typical | Easy via Pervasive Displays |

## Recommendation

**Drop reflective LCD as a candidate display path. It does not solve a problem e-ink doesn't already solve, and the bedtime use case fails the reflective premise.**

Specifically:

1. **Sharp Memory LCD and JDI MIP are too low resolution for reading.** Even the highest-PPI 1.28"–1.34" MIP parts after pancake-lens magnification fall below Sol Reader's virtual PPI. Not viable.
2. **IGZO at glasses scale doesn't exist.** Sharp/Daylight/Hisense panels are 10.5"+ and not separable from the host devices. Not procurable.
3. **The dark-room use case structurally undermines reflective.** A reflective LCD in bed needs a frontlight — same as e-ink — but with worse contrast and worse PPI than e-ink offers. There is no scenario where reflective LCD wins.
4. **If choice-specs ever pivots to a daylight-reading product** (out of scope per north-star: "single purpose" reading at bedtime), revisit. Reflective makes sense outdoors. It does not make sense for our use case.
5. **Useful side benefit of writing this brief:** confirms that the e-ink path is privileged not just by the eye-comfort intuition but by the supply chain — there is literally no other paper-like microdisplay class to build with.

## Open questions / Gaps

- **Custom small IGZO panel.** Sharp could fab one as a custom — same path as E Ink Corp engagement. Out of scope for prototype budget; revisit only at product phase, and only if an IGZO advantage over e-ink emerges (it likely doesn't).
- **Frontlight for Memory LCD.** If we revisit, JDI/Sharp datasheets describe optional frontlight assemblies. None surfaced as standalone parts; would need custom integration.
- **JDI MIP 320×300 1.34" round panel** *might* be usable for a circular page UI experiment. Niche; not pursued.

## Sources

- [Sharp Memory LCD product page](https://sharpdevices.com/memory-lcd/)
- [Sharp Reflective LCD page](https://sharpdevices.com/reflective-lcd/)
- [Sharp SECD low-power LCD displays](https://sharpsecd.com/)
- [Hacker News on Pebble using Sharp Memory LCD](https://news.ycombinator.com/item?id=26861671)
- [Wikipedia — Transflective LCD](https://en.wikipedia.org/wiki/Transflective_liquid-crystal_display)
- [Panox Display 1.26" 144×168 Memory LCD](https://www.panoxdisplay.com/transflective/1-26-memory-lcd-144x168.html)
- [Panox Display 3.2" 336×536 Memory LCD](https://www.panoxdisplay.com/transflective/3-2-memory-lcd-336x536.html)
- [Panox Display transflective catalog](https://www.panoxdisplay.com/transflective/)
- [JDI 1.28" 176×176 Reflective Display LPM013M126A](https://www.youritech.com/products/lpm013m126a-jdi-1-28-inch-reflective-display-176x176-sunlight-readable-3-wire-spi-memory-in-pixel-display.html)
- [JDI 1.28" 176×176 LPM013M126C](https://www.youritech.com/products/lpm013m126c-jdi-1-28-inch-tft-lcd-display-176x176-reflective-sunlight-readable-lcd-panel.html)
- [JDI 1.34" round 320×300 LPM013M091A](https://www.youritech.com/products/lpm013m091a-1-34-inch-round-lcd-transflective-sunlight-readable-320x300-panel-mipi-interface-circle-lcd-display-for-smart-watch-application.html)
- [JDI MIP catalog at Youritech](https://www.youritech.com/products/mip-displays/)
- [JDI 1.28" 205×148 transflective at Toppoplcd](https://www.toppoplcd.com/productdetails_5258051.html)
- [Mbed wiki — JDI MIP reflective color display](https://os.mbed.com/teams/JapanDisplayInc/wiki/MIP-reflective-color-display)
- [Daylight DC-1 vs e-ink (Tom's Guide)](https://www.tomsguide.com/tablets/android-tablets/daylight-dc1-is-a-new-tablet-with-a-live-paper-display-that-might-be-better-than-e-ink-heres-how)
- [Hisense Q5 at e-Ink-Info](https://www.e-ink-info.com/hisense-q5)
- [Hisense Q5 RLCD tablet (Good e-Reader)](https://goodereader.com/blog/electronic-readers/the-hisense-q5-rlcd-tablet-is-back)
- [Sharp LS315M7JX01 Reflective IGZO (eBay)](https://www.ebay.com/itm/116648565214)
- [Android Central — sun-ready reflective tablets (Q5/DC-1/Harbor)](https://www.androidcentral.com/tablets/harbor-paper-7-daylight-computer-hisense-q5-rlcd-tablets-e-readers)
