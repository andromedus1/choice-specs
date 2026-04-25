---
description: "LCoS microdisplays with warm/amber illumination — the most buildable non-e-ink path that preserves eye comfort"
type: brief
campaign: 01-display-and-optics
program: choice-specs-feasibility
research_method: deep-research
confidence: speculative
status: draft
updated: 2026-04-25
---

# LCoS + Warm Illuminator Path

## Question

If the e-ink path proves unsourceable at usable resolution or unworkable inside a glasses frame, LCoS is scout's leading fallback because (a) sub-1" panels are commercially available today, (b) the reflective architecture allows a soft, warm, amber-tinted illuminator that preserves the bedtime-friendly aesthetic, and (c) Kopin and ForthDD support hobbyist-tier engagement. This brief asks: what's actually buyable as an individual; what does the illumination architecture cost in power, weight, and complexity; and how close does a warm-LED-illuminated LCoS get to the e-ink "feel"?

## Findings

### What LCoS is, briefly

Liquid Crystal on Silicon panels are reflective: a CMOS backplane drives a liquid-crystal layer that modulates polarized light reflecting off the silicon. They emit no light themselves — they require an external illuminator (typically RGB LEDs sequentially flashed) and a polarizing beam splitter to inject light and extract the reflected image. They are inherently small (the silicon backplane caps die size) and inherently high-resolution (CMOS lithography). Color is typically time-sequential RGB rather than spatial subpixels, which gives them the highest effective PPI of any color microdisplay. [Wikipedia — LCoS](https://en.wikipedia.org/wiki/Liquid_crystal_on_silicon)

For choice-specs we don't need color. A **single-color amber or warm-white LED** behind a monochrome (or color-with-amber-only-driven) LCoS gives a soft sepia-page aesthetic at a fraction of the power of full-RGB illumination.

### Kopin's commercial LCoS catalog

Kopin is the dominant Western LCoS supplier and runs a public commercial product page. [Kopin Microdisplays](https://www.kopin.com/technologies-products/commercially-available-products/microdisplays/) Notable parts:

- **0.20" WQVGA color LCD** — 428×240 pixels, RGB, used in Divesoft dive masks and rifle scopes. **Smallest commercial Kopin part.** Reflective, ferroelectric LCoS with integrated illumination assembly options.
- **0.24" CyberDisplay** — VGA / WVGA family, the workhorse Kopin part used in many older industrial AR glasses. Discontinued but still findable on the secondary market.
- **0.49" 720p AMOLED** — emissive, not reflective. Listed for completeness; this is Kopin's micro-OLED play, not LCoS.
- **0.88" SXGA M249 FLCoS** — 1280×1024 reflective ferroelectric LCoS. Listed under their Spatial Light Modulator category, also targeted at near-eye applications. [Kopin SXGA 0.88" M249 FLCoS](https://www.kopin.com/technologies-products/commercially-available-products/spatial-light-modulators/sxga-0-88-m249-flcos/)
- **1.3" 2.6K OLED microdisplay** — high-PPI emissive, listed for completeness; not LCoS but Kopin makes this for the AR market. [OLED-Info: Kopin 1.3" 2.6K](https://www.oled-info.com/kopin-developed-new-13-26k-oled-microdisplay)

Kopin **does sell to individual developers**, but distribution is gated. They are NOT carried by DigiKey or Mouser as standard catalog parts. Sample orders go through `sales@kopin.com` directly. Their stated channel is "demo boards" and "high-efficiency backlights" sold as part of an evaluation engagement, but pricing is not public. From hobbyist-forum hearsay (KGOnTech, AR/VR Discord): expect $500–2,000 for a single panel + driver board eval kit, ~4–8 week lead time, no NDA required for off-the-shelf parts.

### Forth Dimension Displays (ForthDD, Scotland)

ForthDD is the British LCoS specialist, focused on extremely high-resolution panels (QXGA 2048×1536, even 2K×2K). [ForthDD products](https://www.forthdd.com/products/) These are aimed at the spatial-light-modulator market (holography, lithography, scientific imaging) but the smaller QXGA-3DM SLM is also marketed for near-to-eye. [ForthDD QXGA-3DM SLM](https://www.displaydaily.com/paid-news/mdm-market-news/forthdd-launches-qxga-3dm-slm)

Sample availability: contact sales@forthdd.com directly. ForthDD is more expensive than Kopin (panels routinely list $3k–8k each as a single unit) and aimed at scientific buyers. **Not a hobbyist channel.**

### Other LCoS players

- **HiMax (Taiwan)** — ships LCoS to most Tier-2 AR glasses brands. Sample orders only via OEM relationship.
- **Compound Photonics (UK/US)** — high-resolution near-eye LCoS, acquired by Snap (Spectacles). Effectively closed.
- **Syndiant (US)** — smaller player; near-eye LCoS panels around 0.31"; very limited sample availability.

For an individual builder, **Kopin is the only realistic LCoS sourcing channel.**

### Illumination architecture

A reflective LCoS in a glasses-form-factor optical engine looks roughly like:

```
[LED illuminator] → [collimator/condenser] → [polarizer] → [PBS cube] → [LCoS panel]
                                                                ↓ (reflects polarized light)
                                                       [PBS recombines] → [eyepiece optic] → eye
```

The polarizing beam splitter (PBS) is the load-bearing optic. Light from the LED enters one face polarized; the PBS reflects it onto the LCoS; the LCoS rotates polarization pixel-by-pixel; the modulated light passes back through the PBS and out the other face into the eyepiece (waveguide / birdbath / pancake / freeform prism — see briefs 5, 6, 7).

For an amber/warm path: a **single 590–605 nm amber LED** at 5–20 mW gives a sepia-on-paper aesthetic. Total wall power with driver inefficiency: ~50–100 mW, well within the choice-specs power budget. **A warm white LED (2700K) with green/blue emission filtered out** is the closer-to-paper visual. Power roughly the same.

LED + driver components are off-the-shelf from DigiKey/Mouser. PBS cubes (5–10mm) are stock items at Edmund Optics ($50–200) or Thorlabs ($100–400). Total optical engine BOM excluding the LCoS panel: ~$200–500.

### Eye-comfort tradeoff vs. e-ink

LCoS-with-amber-LED is **closer to e-ink than micro-OLED**, but is **not equivalent to e-ink**:

- ✅ Spectral profile: a 590nm amber LED has near-zero blue emission, sidestepping the melatonin-suppression concern.
- ✅ Brightness: software-dimmable to <10 nits without flicker (LCoS modulates by reflection, not by emission; it doesn't suffer from low-PWM flicker like OLED at low brightness).
- ✅ Persistence: LCoS in the typical 60Hz time-sequential mode looks "hold-type." With static page content this means no flicker. At 1Hz refresh (page-turn cadence) LCoS would just hold the last frame at low LED current.
- ❌ Reflectivity vs. emissivity: an LCoS panel is itself reflecting illuminator light, but the panel is in the optical box, not facing the room. It does not "look like paper" the way e-ink does; the page is generated by an illuminator that is always on. This is a meaningful UX difference for a reader who wants the e-ink "stillness."
- ❌ Resolution at our scale: Kopin 0.20" at 428×240 is a 250 ppi panel at the panel level, which after pancake-lens magnification gives less effective PPI than Sol Reader's 1.3" e-ink. Going up to the Kopin 0.24" or 0.88" gets back the PPI but increases power and frame depth.

### Reference designs / eval kits

Kopin publishes "demo boards" but they are not pre-engineered for warm-LED illumination — that's a custom integration. Two open references for understanding the optical architecture:

- **Project North Star** (covered in brief 7) uses LCDs not LCoS but the geometry is similar.
- **Plessey / Vuzix / Kopin trade-show demonstrators** — most cited at AWE, SID Display Week. Open source designs are scarce.

[Fraunhofer IPMS](https://www.ipms.fraunhofer.de/en/applications/Customer-Evaluation-Kits/Microdisplays-and-Sensors-Evaluation-Kits/Ultra-bright-SXGA-OLED-microdisplays.html) sells customer evaluation kits for high-resolution OLED microdisplays — same vendor pattern would apply to LCoS but Fraunhofer's catalog is OLED-focused.

## Recommendation

**LCoS-with-amber-LED is the credible fallback if e-ink fails, but it is not the primary path. Hold it as a Plan B and do the engagement with Kopin as a parallel research thread once a build budget exists.**

Specifically:

1. **Primary path remains e-ink** (Brief 1). LCoS does not match e-ink's "still-paper" UX feel even with a warm illuminator. The eye-comfort principle is satisfied better by e-ink.
2. **If e-ink resolution proves unworkable in our pancake-lens budget, switch to Kopin 0.20" WQVGA + amber LED.** This is the buildable LCoS path: smallest panel, simplest illumination, lowest power. Resolution after magnification is comparable to or slightly worse than Sol Reader's 1.3" e-ink.
3. **For initial Kopin engagement, send a sales@kopin.com inquiry with project description, target volume (1 unit prototype, 50–500 if productized), and request datasheet + eval-kit pricing on the 0.20" WQVGA part.** Frame this as a research engagement; expect 2–4 week response cycle.
4. **Budget $1,000–2,500 for an LCoS eval kit including the Kopin panel, driver board, and a custom amber-LED illumination assembly.** Edmund Optics PBS + Thorlabs LED + custom 3D-printed mount.
5. **Do not pursue ForthDD.** Cost and channel both wrong for a personal prototype.
6. **Run an eye-comfort A/B test** on an existing LCoS-based product (Vuzix M400, Divesoft mask) to validate the "warm illuminator preserves bedtime feel" hypothesis empirically before committing.

The headline conclusion: **LCoS is buildable, but it requires a vendor engagement that e-ink does not.** For a personal-feasibility prototype on a $2–5k budget, e-ink wins on procurement friction even if LCoS wins on resolution. Hold LCoS as the second-priority path.

## Open questions / Gaps

- **Kopin pricing** for individual eval kits is not public. Hobbyist-forum hearsay suggests $500–2,000 but only a sales engagement confirms.
- **Custom illumination integration cost.** Building the PBS + LED + driver board sub-assembly is non-trivial for a hobbyist; Campaign 4 (mechanical) will need to scope this.
- **Power draw at low LED current.** Total system power at <10 nits perceived brightness (bedtime case) is the meaningful number, and there's no published characterization. Empirical test required.
- **Polarization-preserving optical path.** LCoS demands polarized light all the way through to the eye; many waveguides depolarize. Pancake-lens uses polarization tricks already, so it's compatible. Birdbath would need polarization-preserving coatings — material engineering question.
- **JBD MicroLED as a competing fallback.** Halliday glasses use a JBD green MicroLED at 0.13"; brightness is high but spectrum is green not amber. Worth a separate investigation if both e-ink and LCoS fail; not in this brief.

## Sources

- [Kopin Microdisplays — commercially available products](https://www.kopin.com/technologies-products/commercially-available-products/microdisplays/)
- [Kopin SXGA 0.88" M249 FLCoS](https://www.kopin.com/technologies-products/commercially-available-products/spatial-light-modulators/sxga-0-88-m249-flcos/)
- [Kopin 720p 0.49" AMOLED color display](https://www.kopin.com/technologies-products/commercially-available-products/microdisplays/oled-720p-49-color/)
- [Kopin custom solutions](https://www.kopin.com/technologies-products/custom-solutions/microdisplays/)
- [Kopin 1.3" 2.6K OLED microdisplay (OLED-Info)](https://www.oled-info.com/kopin-developed-new-13-26k-oled-microdisplay)
- [Kopin launches 2K×2K Reflective LCoS Microdisplay](https://www.kopin.com/press-releases/kopin-launches-2k-x-2k-reflective-lcos-microdisplay/)
- [Forth Dimension Displays — Products](https://www.forthdd.com/products/)
- [ForthDD launches QXGA-3DM SLM (DisplayDaily)](https://www.displaydaily.com/paid-news/mdm/mdm-market-news/forthdd-launches-qxga-3dm-slm)
- [Wikipedia — Liquid crystal on silicon](https://en.wikipedia.org/wiki/Liquid_crystal_on_silicon)
- [Forth Dimension Displays — Wikipedia](https://en.wikipedia.org/wiki/Forth_Dimension_Displays)
- [Fraunhofer IPMS OLED microdisplay eval kit (analogous LCoS pattern)](https://www.ipms.fraunhofer.de/en/applications/Customer-Evaluation-Kits/Microdisplays-and-Sensors-Evaluation-Kits/Ultra-bright-SXGA-OLED-microdisplays.html)
- [VAN-LCoS microdisplay design paper (ScienceDirect)](https://www.sciencedirect.com/science/article/abs/pii/S0141938202000148)
- [KGOnTech: Halliday AR/AI glasses (LCoS-adjacent context)](https://kguttag.com/2025/01/25/halliday-arnot-ai-glasses/)
