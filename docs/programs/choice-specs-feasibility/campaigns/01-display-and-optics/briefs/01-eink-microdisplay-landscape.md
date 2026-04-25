---
description: "E-ink microdisplay vendor landscape — verifying the sub-1\" gap and mapping the realistic procurement path for sub-2\" electrophoretic panels"
type: brief
campaign: 01-display-and-optics
program: choice-specs-feasibility
research_method: deep-research
confidence: speculative
status: draft
updated: 2026-04-25
---

# E-Ink Microdisplay Vendor Landscape

## Question

choice-specs's principle 3 (eye-comfort beats eye-candy) and the bedtime use case both point hard at e-ink as the preferred display medium. The scout found "no sub-1" e-ink microdisplay exists" and Sol Reader confirmed this experimentally by using two 1.3" panels behind pancake lenses. This brief verifies that finding, maps every electrophoretic vendor down to the smallest panel they sell, and identifies the realistic procurement path — distributor, MOQ, lead time — to put a panel in our hands. If the e-ink path is buildable, what does the smallest, highest-resolution panel we can actually buy look like?

## Findings

### E Ink Corporation (Taiwan): the dominant vendor

E Ink Holdings is effectively a monopoly on commercial-grade electrophoretic displays. Their product lineup, organized by film generation:

- **Carta 1200 / Carta 1300 (monochrome)** — the current high-contrast monochrome films. Carta 1300 was introduced in 2024 and is the film used in the latest Kindle Paperwhite and Boox devices. Standard sizes range from ~6" up to 13.3"; the production catalog on eink.com highlights 6" (ED060KHE), 7" (ED070KH4), 10.3" (ED103KC2). [E Ink Carta 1200/1300 product pages](https://www.eink.com/brand/detail/carta1200)
- **Kaleido 3 (color filter overlay)** — color e-paper at lower effective monochrome resolution; 6"–10.3" sizes.
- **Gallery 3 / ACeP (advanced color)** — slow refresh (~1.5–2.5s for full color), only useful for static art / signage.
- **Smaller sizes** appear in the e-shelf-label segment: 1.43", 1.54", 2.13", 2.66", 2.9", 3.7", 4.2". These are sold via OEM channels (SES, Solum, Pricer) for digital price tags, not directly through E Ink for development. They are real Carta or older film products and are physically tiny.

E Ink's smallest **front-lit, integrated** panels (which the bedtime use case wants for low-light reading) start at **6"** in the consumer/Kindle class. The smallest **bare panel** they ship is around **1.43"** (depot price tag class). There is **no commercial sub-1" Carta panel** as of 2026-04. The ~1.3" panel Sol Reader ships almost certainly came from a co-development arrangement with E Ink Corp — confirmed indirectly by Sol Reader's CEO interviews referencing direct E Ink engineering work and "you have to do really neat tricks with the light." [Publishers Weekly Q&A with Ben Chelf](https://www.publishersweekly.com/pw/by-topic/digital/devices/article/98085-can-e-readers-still-innovate-pw-talks-with-sol-reader-ceo-ben-chelf.html)

### Pervasive Displays (Taiwan, E Ink licensee)

Pervasive Displays is the most hobbyist-accessible E Ink licensee. They distribute through DigiKey, Adafruit, Mouser, and Pi Hut. They ship development kits with the EPD Extension Kit (EXT3/EXT4). [Pervasive Displays](https://www.pervasivedisplays.com/) Their smallest current panel is **1.54"** (152×152 or 200×200) at $15–30 retail, single-unit MOQ. They make panels up to ~13.3". They are the canonical channel for buying a small E Ink panel as an individual.

### Good Display / GooDisplay (Dalian Good Display, China)

Good Display is the second-tier hobbyist channel and also an E Ink licensee. They are the most prolific maker of small/odd-size panels:
- **0.97" GDEW0097T50** — smallest e-paper panel surfaced in this scout (184×88, monochrome). [Good Display 0.97" listing](https://www.good-display.com/product/434.html) **NOTE: end-of-life.** Replacement is GDEM0097T61.
- **1.02" flexible GDEM0102T61** (128×80, FPC connection) — interesting because it's sub-1" but the flexible substrate may have different optical behavior under magnification. [e-paper-display.com 1.02"](https://www.e-paper-display.com/products_detail/productId=412.html)
- 1.54", 2.13", 2.66", 2.9" sizes are stocked.

These panels **technically exist sub-1"**, but they are tiny-resolution (≤200×88), single-bit monochrome, with slow refresh. They were designed for IoT badges and shelf labels, not reading. **Resolution is the binding constraint, not size.** A 0.97" panel at 184×88 magnified ~2.5× through a pancake lens to fill the visual field gives an effective 70 ppi — Sol Reader's 256×256 at 1.3" magnified is roughly 300 ppi virtual. Going smaller than 1.3" sacrifices resolution non-linearly.

### Waveshare, BuyDisplay, SeeKink, Crystalfontz (resellers)

These are second-source resellers of the Good Display / Pervasive Displays / E Ink Holdings panels. Useful for fast hobbyist procurement but not panel manufacturers themselves. Waveshare's smallest in-stock panel for individual order is 1.54" (200×200) at ~$20.

### Modos Labs (open-source FPGA driver)

Modos doesn't make panels; they make the **Caster** — an open-source FPGA controller that drives E Ink panels at video rates (up to 75 Hz). [Modos Labs GitHub](https://github.com/Modos-Labs) Their reference monitor uses a 13.3" panel. **Caster is the firmware path if/when we source a small E Ink panel** — without a custom controller, stock E Ink controllers limit refresh to ~1–2 Hz which is fine for page-turn reading but precludes any animation.

### Confirming the scout: there is no sub-1" e-ink reader-class panel

The scout's claim — "E Ink Corp's smallest is ~1.3", nothing smaller [for reading-grade panels]" — is **confirmed**. Smaller panels exist (Good Display 0.97", 1.02") but their resolution disqualifies them for reading. Sol Reader's 1.3" 256×256 panels are at the floor of usability and even those drew "Apple II" / "microfiche" complaints in reviews. [Sol Reader review on Good e-Reader](https://goodereader.com/blog/tablet-slates/sol-reader-your-own-private-reading-aid-a-review)

### The procurement path

For a prototype, three paths exist, in order of buildability:

1. **Pervasive Displays via DigiKey** — 1.54"–4.2" panels, single-unit MOQ, $15–60, evaluation kits with EXT3 driver board, lead time same-day shipping. **Easy.** This is the right path for a first physical prototype.
2. **Good Display direct** — 0.97" through 13.3" panels. Slightly cheaper, slightly slower (1–3 week shipping from China), wider panel selection. Single-unit MOQ on most lines.
3. **E Ink Corp directly** — for sub-1.3" reading-grade panels, requires direct sales engagement. Expect NDA, MOQ in the 1k–10k range, lead time 4–6 months, and likely co-development costs in the $50k–250k range based on Sol Reader's experience. **Out of scope for a personal prototype budget.**

### E Ink's roadmap for smaller / higher-PPI panels

No public roadmap surfaced for sub-1" Carta panels. E Ink's announced 2025–2026 directions are:
- Carta 1300 generally available across existing sizes.
- Gallery 3 productization for art frames.
- Larger panels (25"+) for digital signage.
- "E Ink Folio" partnerships with PC OEMs (e.g. Lenovo).

There is no signal that E Ink is targeting near-eye / microdisplay. The economics don't favor it: shelf-label / e-reader / signage are the volume markets, and microdisplays are the contested territory of OLED/microLED/LCoS — categories E Ink doesn't compete in.

## Recommendation

**Build with the largest e-ink panel that fits the form factor and use a pancake lens to magnify, mirroring Sol Reader's approach — do not chase a sub-1" e-ink panel.**

Specifically:

1. **Prototype with a 1.54" Pervasive Displays panel from DigiKey.** Single-unit MOQ, $15, ships overnight. 200×200 resolution is below Sol Reader's 256×256 — acceptable for a first build to validate the optical and software stack. If text legibility is unacceptable, escalate to Step 2.
2. **Iterate to 2.13"–2.9" if temple length allows.** 2.13" at 250×122 (Pervasive Displays) or 2.9" at 296×128 give materially better PPI. Frame depth becomes an engineering question for Campaign 4 — pancake-lens optical TTL (track length) of ~10–15mm plus panel thickness needs ~20–25mm temple depth, which is fat but not impossible.
3. **Do not pursue direct E Ink Corp engagement at the prototype stage.** Cost, MOQ, and lead time are wrong for a personal-feasibility build. Revisit only if the project goes to product phase and we can write a $250k development check.
4. **Plan to use the Modos Caster as the reference firmware.** Stock E Ink controllers cap refresh too low for a smooth UI; Caster's open FPGA stack gives us the headroom.
5. **Hold the e-ink path as primary**, not as fallback. The eye-comfort principle is the single most heavily-weighted constraint in the project; e-ink is the only display class that fully honors it.

The scout's prior is confirmed: this is a real, structural gap in the supply chain. The workaround (larger panel + magnification) is also confirmed by Sol Reader's existence proof. The risk is panel resolution, which we mitigate by using a panel as large as the frame allows.

## Open questions / Gaps

- **Sub-1" Carta panel source for production.** If choice-specs ever goes to product, a custom E Ink panel is the path Sol Reader took. Cost, MOQ, and lead time are unknown without direct E Ink Corp engagement. Defer.
- **Front-light integration for small panels.** Pervasive Displays' small panels are not sold with integrated front-lights. For dim-room reading we need a custom edge-lit waveguide. Sourceable as a custom optic — Campaign 4 territory.
- **Refresh-rate behavior of small panels under Modos Caster.** Caster targets larger panels; whether the FPGA driver works unchanged on a 1.54" Pervasive Displays panel needs hands-on validation.
- **Greyscale support on small panels.** The 1.54" 200×200 panels are typically 2-bit (4 levels) at fast refresh, 4-bit (16 levels) at slow refresh. Sol Reader claims 16 grayscale at 300ppi virtual. The smallest panel that supports 16-level greyscale at fast refresh is unclear — needs datasheet verification.
- **Long-term supply.** Good Display's 0.97" panel is already EOL. Small e-paper panels turn over faster than larger ones because the IoT badge market churns. We should buy ≥3 spare panels at any given build phase.

## Sources

- [E Ink Carta 1100/1200 product page](https://www.eink.com/brand/detail/carta1200)
- [E Ink 7" Carta 1300 ED070KH4](https://www.eink.com/product/detail/ED070KH4)
- [E Ink 10.3" Carta 1200 ED103KC2](https://www.eink.com/product/detail/ED103KC2)
- [E Ink — main product directory](https://www.eink.com/)
- [MobileRead Wiki — E Ink Carta](https://wiki.mobileread.com/wiki/E_Ink_Carta)
- [Pervasive Displays main site](https://www.pervasivedisplays.com/)
- [Pervasive Displays at DigiKey](https://www.digikey.com/en/supplier-centers/pervasive-displays)
- [E Ink at DigiKey](https://www.digikey.com/en/supplier-centers/e-ink)
- [Good Display 0.97" GDEW0097T50 (EOL)](https://www.good-display.com/product/434.html)
- [1.02" flexible e-paper at e-paper-display.com](https://www.e-paper-display.com/products_detail/productId=412.html)
- [Waveshare 1.54" 200×200 panel](https://www.waveshare.com/1.54inch-e-paper-module.htm)
- [BuyDisplay 1.54" 152×152 e-paper panel](https://www.buydisplay.com/1-54-inch-epaper-152x152-smallest-e-ink-display-panel-white-black)
- [Adafruit 1.54" Monochrome e-Ink Display](https://www.adafruit.com/product/4196)
- [Modos Labs (Caster FPGA driver) on GitHub](https://github.com/Modos-Labs)
- [SEEKINK e-paper display catalog](https://www.seekink.com/eink-displays/)
- [Publishers Weekly Q&A with Sol Reader CEO Ben Chelf](https://www.publishersweekly.com/pw/by-topic/digital/devices/article/98085-can-e-readers-still-innovate-pw-talks-with-sol-reader-ceo-ben-chelf.html)
- [Sol Reader review — Good e-Reader](https://goodereader.com/blog/tablet-slates/sol-reader-your-own-private-reading-aid-a-review)
