---
description: "Micro-OLED at low refresh + dimming — eye-comfort tradeoffs and burn-in risk for static page reading at bedtime brightness"
type: brief
campaign: 01-display-and-optics
program: choice-specs-feasibility
research_method: deep-research
confidence: speculative
status: draft
updated: 2026-04-25
---

# Micro-OLED at Low Refresh + Dimming

## Question

If both e-ink (Brief 1) and LCoS (Brief 2) fail, micro-OLED is the dominant remaining microdisplay class — every shipping AR product (XReal, Rokid, VITURE, Apple Vision Pro) uses it. The eye-comfort concern is the spectral profile (high blue), the persistence (always emissive), and the burn-in risk (static page content). This brief asks: at the choice-specs use case (1–5 Hz refresh, ~10–30 nits, monochrome amber tint via software, static page reading 90 min/session), does micro-OLED actually fail the eye-comfort principle, or does software dimming + amber tinting + low refresh make it acceptable?

## Findings

### The micro-OLED supply chain in 2026

The big four:
- **Sony Semiconductor (Japan)** — the ECX series (ECX335S 0.71" 1920×1080 is the canonical part). Sells via OEM channels; sample acquisition is gated by NDA. Used in Apple Vision Pro, XReal Air series, VITURE.
- **BOE / Yunnan OLightek (China)** — 0.5"–1.03" panels at 1920×1080 to 4K. Aggressive pricing; available on Alibaba/Made-in-China at ~$300–800 for single 0.71" panels with driver boards.
- **SeeYa Technology (China)** — listed on STAR Stock Exchange, mass-production focused. 0.71" panels common. [SeeYa, BOE, Sony 0.7"–0.72" microdisplays on Made-in-China](https://www.made-in-china.com/showroom/0e8010ae45e28857/product-detailCOMtNyoulGfc/China-Seeya-Boe-Sony-0-7-0-71-0-72inch-Amoled-Microdisplays-Si-OLED-1920X1200-1080P-FHD-1920X1080-Ecx335s-Vr-Ar-Microoled-Display.html)
- **eMagin (US, acquired by Samsung 2023)** — historically THE high-end military/medical micro-OLED supplier. Now part of Samsung; future channel uncertain.

For an individual builder, **the China-sourced 0.71" 1920×1080 panel + driver board on Alibaba is the canonical procurement route**: $300–800, single-unit MOQ, 2–4 week lead time, no NDA. Sony parts are not realistically buyable as a hobbyist except as Sony-purchased complete devices to be torn down.

### Spectral profile and bedtime / melatonin concern

OLED's blue-light hazard reputation is more nuanced than the popular framing. Several findings:

- **OLED is markedly LOWER blue-emission than LCD** because OLEDs are emissive per-subpixel and don't rely on a blue-LED-pumped white backlight. LG Display's clinical study (Kookmin University, 40 adults, 2 hours of TV viewing) measured **melatonin secretion fell 2.7% over two hours for an LCD group and rose 8.1% for the OLED group.** [SID Information Display — LG and Eyesafe on OLED for sleep](https://sid.onlinelibrary.wiley.com/doi/full/10.1002/msid.1553)
- **Blue-light hazard is brightness-dependent, not just spectrum-dependent.** At <30 nits the absolute blue photon flux is low even on a non-color-managed OLED. [OLED-Info on blue-light hazard](https://www.oled-info.com/oled-blue-light-hazard)
- **The micro-OLED in choice-specs would be aggressive software-dimmed** to ~5–20 nits perceived (vs. ~150 nits typical desktop). Combined with a software amber/sepia tint (drive only the red subpixel and a fraction of green), the blue subpixel can be effectively zero. This is what Apple's "Night Shift," f.lux, and Kindle Paperwhite "warm" front-light all approximate; on micro-OLED it's literally driving the blue subpixel at zero current.

**Conclusion on spectrum:** with software dimming + software amber tint, micro-OLED's spectral profile is acceptable for bedtime reading. It is NOT inherently disqualifying. The "OLED blue light suppresses melatonin" framing is correct in absolute terms but misses the 100× brightness reduction we're applying.

### Persistence / "always emitting" concern

E-ink holds a static image with **zero hold power** — the display literally stops drawing current between refreshes. Micro-OLED is an emissive display: every visible pixel is drawing current continuously. For our use case this matters in three ways:

1. **Power.** A 0.71" 1080p OLED at 30 nits draws ~50–150 mW (manufacturer figures, cross-checked against XReal/VITURE published numbers scaled down). Compared to e-ink's <1 mW hold power, micro-OLED is a 50–150× active power penalty for static reading. Over a 90-minute session, that's an extra ~150–300 mWh — small in absolute terms but meaningful when the battery is also constrained by ≤50g weight.
2. **Eye-comfort feel.** A "still" OLED page at 30 nits is not literally still — sub-pixel PWM at low brightness introduces flicker at frequencies that sensitive users can perceive as eye strain. **DC dimming** (vs. PWM) avoids this and is supported on most modern micro-OLED drivers but must be explicitly enabled. [Panox on OLED dimming and eye fatigue](https://www.panoxdisplay.com/knowledge/oled-bad-for-your-eyes.html)
3. **Burn-in.** Static page content over 90-minute sessions, then static "library" UI between sessions — this is exactly the burn-in failure mode. **DisplayMate 2023 reported 4K OLEDs at 100 nits develop permanent burn-in after >72 hours of static logo display.** At 30 nits the threshold pushes out (less current per subpixel), but it does not vanish. Over a 5-year product lifetime at ~2 hours/night, that's ~3,650 hours of static content — well into burn-in territory. Mitigations: **pixel shift** (sub-pixel jitter every frame), **page text repositioning** every 30–60 seconds, **lower brightness still**.

### Low-refresh behavior

Micro-OLED panels driven below their nominal refresh rate (typical 60/90/120 Hz) is unusual but mechanically straightforward. The MIPI-DSI interface can run at 1–5 Hz commanded refresh; the panel just holds the last-written frame in its memory cells. This is the same trick used on the Apple Watch always-on display.

**Empirically, low-refresh (1–5 Hz) micro-OLED at low brightness should look "calm" to the user** — no scan-line flicker, no motion artifact, just a still page. The absence of any data on what 1–5 Hz refresh + 10–30 nits + amber tint actually FEELS like for a 90-minute reading session is a real gap; this is the most important thing to prototype on a Quest or VITURE before committing to micro-OLED.

### Sourcing for prototyping

The hobbyist channel for micro-OLED is **Alibaba/AliExpress for the BOE/SeeYa 0.71" + driver board**, ~$300–800. [Made-in-China 0.71" microdisplays](https://www.made-in-china.com/showroom/0e8010ae45e28857/product-detailCOMtNyoulGfc/China-Seeya-Boe-Sony-0-7-0-71-0-72inch-Amoled-Microdisplays-Si-OLED-1920X1200-1080P-FHD-1920X1080-Ecx335s-Vr-Ar-Microoled-Display.html) Driver boards typically take HDMI in and output MIPI-DSI to the panel.

DisplayModule (US-based reseller of Chinese OLED + birdbath modules) sells complete 1920×1080 birdbath optical modules for $500–1,200 with built-in micro-OLED. [DisplayModule birdbath products](https://www.displaymodule.com/) — useful as a prebuilt path if optics integration is too much engineering for the prototype.

### The "third path" reality

Micro-OLED is the path the rest of the AR industry took. We're swimming against the current by preferring e-ink. Brief observation: **every AR-glasses product the user has access to today (XReal, Rokid, VITURE, Halliday, Even G1, Vuzix Z100) uses micro-OLED or microLED** — both emissive. There is no shipping non-emissive AR display product except Sol Reader. This is a massive supply-chain advantage for micro-OLED. If we go micro-OLED we inherit the ecosystem; if we go e-ink we build the ecosystem.

## Recommendation

**Micro-OLED is an acceptable third-priority fallback. Prototype it specifically as an A/B comparison against e-ink to empirically test whether software dimming + amber tint + low refresh can match e-ink's bedtime feel — but do not commit to micro-OLED as the primary path.**

Specifically:

1. **Acquire a Sony ECX335-class 0.71" 1080p panel + HDMI driver board from Alibaba ($400–700) for A/B testing.** Drive at 30 nits, amber-tinted (red+green subpixels only), 1 Hz refresh in static state. Wear for 60 minutes in a dark room. Compare subjectively against a Kindle Paperwhite at warm-front-light = max-warm.
2. **If A/B test shows micro-OLED is acceptably e-ink-like at low brightness/amber tint, hold it as a "ship it now" alternative** — the supply chain is mature, the optics modules are off-the-shelf, the development time would shrink by months.
3. **If micro-OLED in A/B test produces eye fatigue or visible flicker, drop it from the candidate set.** Don't try to engineer around it; the e-ink path is closer to the principle.
4. **Burn-in mitigation must be designed in if micro-OLED is chosen**: pixel-shift, full-screen inversion every N pages, max 30 nits effective brightness. Acceptable but adds firmware complexity.
5. **Use DC dimming, not PWM dimming, in firmware.** This is a hard requirement for any OLED choice; the difference between flicker and no-flicker at low brightness.
6. **Power penalty (~50–150 mW continuous vs. e-ink's ~1 mW) must be carried into Campaign 4's battery sizing.** A 90-minute session adds ~75–200 mWh — within budget for a 1000mAh cell but marginal for any aggressive ≤30g battery.

**Headline:** micro-OLED is the realist's fallback. Eye-comfort is acceptable with the right tuning; the supply chain is the easy one. But it is not first-best for choice-specs because it never gets to "still as paper."

## Open questions / Gaps

- **Empirical eye-comfort over 90 minutes at 1–5 Hz / 30 nits / amber-tinted micro-OLED.** No published data. The single most important thing to prototype before committing to or rejecting this path. Recommend Quest 3 or VITURE One sideload as the empirical test bed before custom hardware.
- **Long-term burn-in trajectory** at 2 hours/night, 5 years, 30 nits, with page-text content. Manufacturer data only covers higher brightness; extrapolation is rough.
- **DC-dimming support specifics** vary by panel and driver. ECX335 supports it; cheaper Chinese parts are inconsistent. Datasheet check required at sourcing time.
- **Sony's actual hobbyist channel.** Sony Semiconductor sells through Japanese OEM partners; getting an ECX-series panel as a US individual is uncertain. The Alibaba parts marked "ECX335s" are sometimes B-stock or fakes — empirical caveat for procurement.
- **Polarization compatibility with pancake lens.** Micro-OLED is unpolarized; pancake-lens stacks typically include their own polarizers, so this is benign, but the loss budget needs verification at design time.

## Sources

- [SID Information Display — LG/Eyesafe study on OLED and sleep](https://sid.onlinelibrary.wiley.com/doi/full/10.1002/msid.1553)
- [Sleepopolis — OLED TV may help sleep](https://sleepopolis.com/news/oled-tv-might-improve-sleep/)
- [OLED-Info — blue-light hazard and OLED](https://www.oled-info.com/oled-blue-light-hazard)
- [Effects of OLEDs on Circadian Rhythm and Sleep (PMC)](https://pmc.ncbi.nlm.nih.gov/articles/PMC8169338/)
- [Sleep Foundation on blue light](https://www.sleepfoundation.org/bedroom-environment/blue-light)
- [Panox — Is OLED Bad for Your Eyes? Dimming and flicker](https://www.panoxdisplay.com/knowledge/oled-bad-for-your-eyes.html)
- [Made-in-China BOE/SeeYa/Sony 0.71" microdisplays](https://www.made-in-china.com/showroom/0e8010ae45e28857/product-detailCOMtNyoulGfc/China-Seeya-Boe-Sony-0-7-0-71-0-72inch-Amoled-Microdisplays-Si-OLED-1920X1200-1080P-FHD-1920X1080-Ecx335s-Vr-Ar-Microoled-Display.html)
- [DisplayModule — full color vs monochrome micro-OLED, luminance and power](https://www.displaymodule.com/blogs/knowledge/full-color-vs-monochrome-micro-oled-use-cases-luminance-power-draw)
- [DisplayModule on extending OLED lifespan / preventing burn-in](https://www.displaymodule.com/blogs/knowledge/how-to-extend-oled-lifespan-preventing-burn-in-power-management)
- [OLED-Info — OLED microdisplays](https://www.oled-info.com/oled-microdisplays)
