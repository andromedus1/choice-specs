---
description: "Campaign 1 (Display & Optics) synthesis — display tech and optical path recommendations for choice-specs"
type: brief
campaign: 01-display-and-optics
program: choice-specs-feasibility
research_method: deep-research
content_type: campaign-synthesis
confidence: speculative
status: draft
updated: 2026-04-25
---

# Campaign 1 — Display & Optics: Synthesis

## Campaign question

What display technology and optical path lets choice-specs deliver e-ink-class bedtime reading comfort in a glasses-class form factor (≤50g), buildable as a personal prototype on a $2–5k budget, with a procurement path that doesn't depend on enterprise-scale vendor engagement?

This campaign covered ten sub-briefs spanning the four display technology classes (e-ink, LCoS, micro-OLED, reflective LCD), the four optical-path classes (pancake, birdbath, reflective combiner, freeform prism), the eye-comfort literature on focal distance for long sessions, the only shipped predecessor (Sol Reader), and the end-to-end hobbyist sourcing path. Display/optics is the gating campaign — every constraint here propagates to compute (Campaign 2), input/IMU (Campaign 3), and frame/power/safety (Campaign 4).

## Decision: Display tech

### Recommended path: **E-ink (Pervasive Displays 1.54"–2.13" via DigiKey)**

Rationale: Honors the "eye-comfort beats eye-candy" principle (#3) most fully. Aligns with the bedtime / dark-room / no-melatonin-suppression use case. Validated by Sol Reader as a working architecture. Sourcing is friction-light: single-unit MOQ, $15–60 per panel, same-day shipping from a US distributor. Front-light integration is achievable via Kindle-class edge-lit waveguide.

Specifically:
- **Prototype with 1.54" 200×200 Pervasive Displays panel.** $15, single-unit. If text legibility falls short, escalate to 2.13" 250×122 or 2.9" 296×128 — frame depth permitting.
- **Use Modos Caster FPGA for fast refresh.** Stock E Ink controllers cap at ~1–2 Hz; Caster gets us 30+ Hz when needed for UI animation.
- **Scout's claim "no sub-1" e-ink microdisplay" is confirmed.** Sub-1" panels (Good Display 0.97", 1.02") exist but at prohibitively low resolution. The 1.3"+ panel + magnification path Sol took is the right model.
- **E Ink Corp direct engagement is out of scope for prototype.** $50k–250k NRE, 1k–10k MOQ, multi-month lead. Revisit only at product phase.

### Fallbacks, in order

1. **LCoS + warm-amber illuminator (Kopin 0.20" or 0.88" via direct sales).** The most credible non-e-ink path that preserves bedtime feel. $1,000–2,500 eval kit cost; 4–8 week lead time; vendor engagement required. Resolution after pancake-magnification is comparable to or slightly worse than 1.3" e-ink. **Use only if e-ink path proves unworkable in our pancake/frame budget.**
2. **Micro-OLED at low refresh + amber tint (BOE/SeeYa 0.71" via Alibaba).** Mature supply chain, $300–800, single-unit. Eye-comfort is acceptable with software dimming + amber tint + DC dimming + burn-in mitigation, but it never reaches the "still as paper" feel of e-ink. **Use only if both e-ink and LCoS fail.**

### Rejected paths

- **Reflective LCD (Memory LCD, JDI MIP, IGZO).** Too low-PPI at glasses scale; reflective premise fails in dark room without front-light, at which point e-ink is structurally better.
- **microLED (JBD).** Eye-comfort hostile per scout; emissive at high brightness with green-only spectrum.
- **Retinal e-paper (Uppsala research).** Years from buildable.

## Decision: Optical path

### Recommended path: **Monocular pancake, salvaged from used Quest 3**

Rationale: Honors principle 6 (≤50g) — pancake is the slimmest optical class. Honors principle 7 (personal first) — monocular cuts weight + complexity in half. Validated by Sol Reader. Custom pancake designs are unbuildable for hobbyists (Meta's "supply chain from scratch over years"); salvaging Quest 3 lenses solves the procurement problem with $250 of used hardware.

Specifically:
- **Salvage one pancake lens from a used Quest 3 ($200–400 on eBay).** Discard or retain the second lens.
- **Drive the panel at sufficient brightness to overcome 87.5% pancake loss** (unpolarized e-ink → 12.5% transmission). Front-lit e-ink at ~250 panel-nits → ~30 nits to eye = correct bedtime brightness.
- **Build a Hackaday-class "PsuedoPancake" as $30 backup** if Quest 3 fit fails.
- **Plan for a ~25mm front-of-frame depth.** This is fatter than Even G1 / Vuzix Z100 but acceptable for a bedside reader.

### Fallback: **Monocular birdbath via DisplayModule kit (only if we accept micro-OLED)**

Rationale: DisplayModule birdbath kits ship with integrated micro-OLED at $400–1,200, complete optical engine, no custom optics work required. Best path if we want a wearable prototype in a month. **Forfeits the e-ink commitment** because the kits are tuned for OLED.

### Rejected paths

- **Reflective combiner (Project North Star)** — wrong tool: wide-FOV is anti-feature for reading; bulkier than birdbath.
- **Dual-curved-mirror (IMMY)** — no open designs, no buyable components.
- **Freeform prism (Olympus, Epson Moverio)** — patent-encumbered or product-locked.
- **Diffractive waveguide (Even G1 / Vuzix Z100 class)** — requires JBD MicroLED partnership; closed channel; HUD-class FoV not reading-class.

## Decision: Virtual page focal distance

### Recommended: **~50cm (2.0 D), monocular, dominant eye, occlude the unviewing eye**

Rationale: Brief 8's deep dive on the VAC literature indicates monocular HMDs sidestep the canonical stereoscopic VAC entirely (no stereo cue, no conflict to resolve). The dominant remaining fatigue source is prolonged near accommodation — exactly the same ciliary fatigue we already accept from Kindle / paperback at 50cm for 90 minutes. Matching natural reading distance also preserves the user's mental model.

Alternative considered: ~2m (0.5 D) for relaxed accommodation. Common in shipping AR glasses (XReal, Even G1, Halliday). Lower accommodation effort but breaks the "reading" mental model and has not been validated for extended-session reading specifically. **Empirical A/B test required at prototype stage** — design the optical engine to allow swappable focal-distance lenses for the test.

Monocular-with-occluded-other-eye specifics:
- **Use the dominant eye** (sighting test confirms, single user, "personal first").
- **Occlude the unviewing eye** with opaque inner frame — reduces interocular conflict; benign in dark bedroom.
- **Reading speed penalty for monocular vs binocular = ~2%** (Banks et al, normal-vision subjects). Negligible.

## Cross-cutting findings

### 1. The supply chain favors pancake-with-e-ink — but only with salvage and via Pervasive Displays

Sol Reader's stack (e-ink + pancake) is the only existing implementation, and it's reproducible at hobbyist scale only with creative procurement (eBay-salvaged Quest 3 + Pervasive Displays panel via DigiKey). The "no off-the-shelf pancake assembly for hobbyists" gap is real and is the single biggest procurement constraint. Salvage closes the gap.

### 2. "Eye-comfort" is a four-axis evaluation, not one

The eye-comfort principle decomposes into:
- **Spectral profile** (blue-light / melatonin) — e-ink wins, LCoS-amber wins, OLED-with-amber-tint passes, OLED untreated fails.
- **Persistence** (still-paper feel vs continuous emission) — e-ink wins, LCoS is hold-type but emitter-illuminated, OLED is continuous.
- **Brightness controllability at low end** — e-ink is essentially unflickerable, LCoS is software-dimmable cleanly, OLED requires DC-dimming explicitly to avoid PWM flicker.
- **Vergence-accommodation** — monocular dodges most of it; focal-distance choice determines residual accommodation effort.

Only e-ink wins all four axes. LCoS wins three. Micro-OLED wins two with software work. This ordering is the basis for the display recommendation.

### 3. The dark-room / non-see-through reframe simplifies optics meaningfully

Several optic classes (birdbath, waveguide) are conventionally optimized for see-through use. Choice-specs's bedtime-only framing makes see-through irrelevant. Specific simplifications:
- **Birdbath becomes a simpler "closed" engine** — full-reflective rear coating instead of partial-reflective doubles efficiency.
- **Pancake becomes more efficient** — no need to balance see-through with display path.
- **Ambient-blocking is a feature, not a flaw.** Light from outside the frame is unwanted noise.

This reframe was scout's insight; this campaign confirms and operationalizes it.

### 4. Resolution is the binding display constraint, not size

Sol Reader's 256×256 per panel produces ~300 PPI virtual after magnification — which still draws "Apple II" / "microfiche" complaints. Going below this in choice-specs is not viable. **Choice-specs needs ≥256×256 per panel at 1.5"+ to match Sol baseline; targeting 320×320 or 400×400 to exceed it.** Pervasive Displays 2.13" 250×122 is sub-Sol resolution; 2.9" 296×128 is comparable; 4.2" 400×300 exceeds but pushes frame depth.

### 5. Monocular is structurally better for choice-specs than binocular

Monocular wins on:
- Weight (half the optic mass)
- BOM cost
- Mechanical complexity (no IPD adjustment)
- VAC (no stereo cue, no conflict)
- Compute (no stereo rendering)
- Sourcing (one Quest 3 lens, not two)

Monocular loses on:
- Reading speed (~2%, negligible)
- Interocular conflict (mitigated by frame occlusion)
- Asymmetric ambient adaptation (benign in dark room)

The math favors monocular by a wide margin.

### 6. The empirical-baseline-before-build path is high-leverage

Two purchases unlock most of the empirical questions cheaply:
- **Sol Reader at $249** — validates the entire device concept end-to-end. Wear it for an hour; if it works, the architecture works.
- **Kobo Clara HD at $30–50** — validates the software stack (KOReader on i.MX6 + e-ink). The Campaign 2 bedrock.

Both should be ordered Day 1 of any build budget. They short-circuit a month of design risk for $300.

## Lessons from Sol Reader

**Correction to scout-landscape:** Sol Reader is NOT discontinued. As of 2026-04, the company is actively shipping at $249, recently added Kindle integration (June 2025), updated the iOS app (November 2025), and current production runs sell out. The scout's framing of "shipped briefly in 2023 and was discontinued" is wrong and the original "single most important data point in this scout" framing should be revised. Scout-landscape Theme A should be updated to remove the "discontinued" claim. **This correction propagates upstream** — north-star.md's problem statement says "the closest attempt, Sol Reader, shipped briefly in 2023 and was discontinued" should also be revised.

What we will preserve from Sol's architecture:
1. **Single-purpose device** (no apps, no notifications)
2. **Open EPUB** as the primary content format
3. **Pancake optics + e-ink** as the optical-display pairing
4. **Sub-100g target** (we go more aggressive: ≤50g)
5. **WiFi sync, no phone tether** for content
6. **BT clicker / ring** as the input fallback (matched to scout-landscape Theme K)
7. **Bedside posture optimization** (no head strap, designed for reclined use)

What we will diverge on:
1. **Monocular instead of binocular** (lighter, simpler, lower VAC risk)
2. **Higher native resolution** even at the cost of slightly larger panel / frame depth
3. **Head-tilt as primary input**, BT ring as fallback (Sol uses only handheld)
4. **Glasses form factor target** instead of goggle (more aspirational, harder)
5. **Open / hackable software stack** (KOReader / Plato instead of Sol-branded)
6. **Haptic confirmation on page turn** (Sol has none)

## Sourcing path

End-to-end procurement plan, sequenced for unblocking flow:

**Day 1 ($300):** Sol Reader + Kobo Clara HD. Empirical baselines.

**Day 14 ($150):** Pervasive Displays 1.54" e-ink × 3 + Adafruit BNO055 + RK3566 SBC. Software stack starts.

**Day 30 ($300–500):** Used Quest 3 (eBay) for pancake salvage; Modos Caster FPGA dev board; LiPo + BMS + USB-C charging IC (DigiKey). Hardware integration starts.

**Day 60 ($200):** SLA-printed frame iteration; Edmund Optics polarizer + small mirrors for assembly bench; Gevo BT ring. First wearable prototype.

**Reserve $1,000** for downstream Campaign 2/3/4 components (custom PCB fab, SoC change, optics iteration).

**Total prototype budget: ~$1,950–2,150.** Well within $2–5k. Adding LCoS fallback path adds ~$1,500.

**Tier strategy:**
- **Tier 1 (DigiKey, Mouser, Adafruit) for everything you can name** — single-unit MOQ, fast shipping, no risk.
- **Tier 2 (DisplayModule, Alibaba, Made-in-China) only for AR-specific optics + microdisplays** — small-batch first; vet supplier history.
- **Tier 3 (direct E Ink, Kopin) only at product phase**.
- **Tier 4 (eBay Quest 3 / Sol Reader / Kobo) is the load-bearing channel for "high-quality optics + working baselines."**

**Community channels to join Day 1:**
- Project North Star Discord
- MentraOS Discord
- MobileRead Forums (Sol Reader / Kobo / KOReader threads)
- Subscribe to KGOnTech (Karl Guttag) for AR-optics signal

## Open questions for downstream campaigns

### To Campaign 2 (Compute & Software Stack)

1. **Display refresh rate** — Modos Caster supports up to 75 Hz on E Ink. What does the page-rendering pipeline (Campaign 2 territory) actually need? 1 Hz for static text, 5–10 Hz for UI animation, 30 Hz for scrolling? Determines compute and memory.
2. **Panel resolution implication for typography** — at 1.54" 200×200 we have a constrained typography budget; KOReader's font rendering must be tuned. (Campaign 2 will have a "typography at low resolution / monochrome" brief.)
3. **Power profile of e-ink + Caster + RK3566** — total system active power is ~300–500 mW. Battery sizing must accommodate.
4. **Front-light power and dimming** — controlled by SoC GPIO + PWM + driver IC. Software dimming behavior.

### To Campaign 3 (Input & Interaction)

1. **IMU placement** — pancake optical engine + e-ink panel + driver in the temple is bulky. Where does the IMU sit for cleanest gravity-axis sensing? Likely above the bridge, not in the temple.
2. **Monocular optical path means asymmetric weight on dominant-eye side.** Affects center-of-mass, head-tilt sensing baseline. Calibrate accordingly.
3. **BT ring pairing** — single ring or could be paired with multiple devices (Sol Reader, choice-specs prototype, debug rig)? Likely just one paired peer at a time.

### To Campaign 4 (Industrial Design, Power, Safety)

1. **Frame depth budget = ~25mm at front of eye** for pancake + 1.54" e-ink + driver. Larger than fashion-glasses; smaller than VR goggles. Reclined-use ergonomics tolerate it.
2. **Weight budget** — pancake ~10g + e-ink panel ~5g + driver ~3g + frame ~10g + battery ~10g + electronics ~5g + IMU/BT ~2g = **~45g.** Below 50g cliff but tight.
3. **Heat from LED front-light** is in the milliwatt range; not a concern.
4. **Battery cell — 500–800 mAh single LiPo** sized for 2-hour active session + 2-week standby. Cell weight ~10g. Sleep-context safety per Bose Sleepbuds lesson — use certified cells from Adafruit/Mouser, not Alibaba surplus.
5. **Prescription-corrector path** — defer per principle 7. Sol's insert approach is the conservative choice if revisited.

### Unresolved at Campaign 1 (revisit later)

1. **Empirical 60-min comfort at monocular pancake e-ink at 50cm.** Most important next test. Requires either Sol Reader baseline (likely "yes, comfort works") or first prototype.
2. **Front-light coupling efficiency** with pancake polarization geometry. Bench test required.
3. **Actual resolution feeling at 1.54" 200×200 magnified.** Should be test-driven on a paper mock-up at the right magnification before committing the panel buy.
4. **LCoS pricing from Kopin direct.** Send the inquiry email but keep the e-ink path moving in parallel.

## Brief index

- [01 — E-Ink Microdisplay Vendor Landscape](briefs/01-eink-microdisplay-landscape.md) — confirms sub-1" gap; Pervasive Displays via DigiKey is the procurement path
- [02 — LCoS + Warm Illuminator Path](briefs/02-lcos-warm-illuminator.md) — Kopin direct sales, $1k–2.5k eval kit; the credible non-e-ink fallback
- [03 — Micro-OLED at Low Refresh + Dimming](briefs/03-microoled-low-refresh.md) — acceptable with DC-dimming + amber tint + burn-in mitigation; not first-best
- [04 — Reflective LCD Options](briefs/04-reflective-lcd.md) — rejected (low PPI; dark room undermines reflective premise)
- [05 — Pancake Lens Design](briefs/05-pancake-lens.md) — recommended; salvage from Quest 3
- [06 — Birdbath Optics for Monocular Non-See-Through Reading](briefs/06-birdbath-optics.md) — secondary; only if we accept micro-OLED via DisplayModule kit
- [07 — Reflective Combiner / Freeform Prism Alternatives](briefs/07-reflective-combiner-freeform-prism.md) — rejected (overkill, unbuildable, or product-locked)
- [08 — Vergence-Accommodation Conflict + Eye Comfort](briefs/08-vergence-accommodation.md) — virtual page at 50cm, monocular dominant eye, occluded other eye
- [09 — Sol Reader Status, Lessons, and Direct Comparison](briefs/09-sol-reader-postmortem.md) — correction: Sol is NOT discontinued; it's a working production reference
- [10 — Component Sourcing as a Hobbyist](briefs/10-component-sourcing.md) — $2k buildable prototype with mainstream distributors + Sol/Kobo + Quest 3 salvage
