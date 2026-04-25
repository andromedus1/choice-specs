---
description: "Scout landscape for choice-specs — prior art, adjacent approaches, lessons learned for AR-glasses bedtime reading device"
type: landscape
updated: 2026-04-25
---

# Scout Landscape: choice-specs

*Scouted: 2026-04-25*

## Context

choice-specs is a personal feasibility prototype: a standalone wearable for reading books in AR-glasses form factor. Standalone (no tether), single-purpose, e-ink display preferred (no blue light, eye comfort, low power), prism/birdbath optics, head-locked single-page UX, EPUB-only, head-tilt input (BT clicker/ring as fallback). Designed for bedtime reading with hands under the blankets — propped-up posture in bed or chair/sofa.

This landscape maps prior art across the project's risk surfaces so that downstream research, architecture, and design can be grounded in what exists rather than what's assumed. Read alongside the north star (when written) and the research-program plan that will follow.

## Search Vectors

**Direct:**
- E-ink microdisplays for AR/wearables
- Open AR glasses platforms / DIY HMDs
- Consumer AR glasses (tethered + standalone)
- EPUB renderers for embedded / lightweight systems

**Adjacent:**
- Head-tilt / IMU gesture input as primary UX
- Standalone wearable compute platforms
- Single-purpose e-reader hardware/software philosophy
- Open / DIY HMD optical designs

**Analogous:**
- Audiobooks as the dominant in-bed reading mode
- VR/AR reading apps (Quest, Vision Pro)
- HUDs in other domains (cycling, motorcycle, military, surgical)
- In-bed / sleep tech form factors
- Single-purpose / focus / no-notifications devices

## Landscape

### Theme A — The only shipped predecessor

#### [Sol Reader](https://solreader.com) — assessed, **discontinued**

The only commercially-shipped e-ink HMD ever found. Wearable e-reader goggles using E Ink panels viewed through pancake lenses. Less than 100g, ~$350 at launch (2023), discontinued.

**Approach:** Pancake-lens optical path (not birdbath, not waveguide), sub-1.3" or larger E Ink panels, monocular or near-monocular. Reportedly worked with E Ink Corp on higher-resolution panels but did not get there.

**Lessons:**
- Existence proof that e-ink + near-eye optics is physically buildable for reading.
- Pancake lens was the chosen optical path — simpler than birdbath when see-through is not required.
- Resolution complaints from users even with E Ink cooperation suggests the display tech is the binding constraint.
- Discontinuation is the single most important data point in this scout. A specific post-mortem deserves its own research dig (commercial failure vs. UX failure vs. supply chain failure are very different lessons).

**Relevance:** This is the prior art that most directly defines what we're attempting. Every architecture decision should ask "did Sol Reader try this, and how did it go?"

---

### Theme B — Slim HUD glasses (the closest currently shipping form factor)

#### [Even Realities G1](https://www.evenrealities.com/g1) — assessed

43g, microLED + waveguide, monochrome 640×200, 1000 nits, 12-hour battery, BLE-paired to phone, $599, prescription-compatible.

**Approach:** Lightweight HUD-on-real-glasses; waveguide optics for see-through; monochrome low-resolution display sized for text/notification class content; BLE-tethered to phone for content.

**Lessons:**
- 43g + waveguide + 12hr battery is shipping today in a frame that genuinely looks like glasses.
- 640×200 monochrome is enough bandwidth for text but raises a real EPUB typography question — what does a book at this resolution actually look like? (Worth its own research.)
- Prescription support is a solved manufacturing problem, not a research gap.

**Relevance:** Closest commercial product to the target form factor. If software-on-existing-hardware is the build path, this is one of two candidates.

#### [Vuzix Z100](https://www.vuzix.com/products/z100-smart-glasses) + [MentraOS](https://github.com/Mentra-Community/MentraOS) — assessed

38g, monochrome transparent display, 48-hour battery, looks like standard glasses. BLE to phone. Partners with MentraOS (TypeScript SDK, $8M Series A, app store).

**Approach:** Same class as Even G1 but with a real open software platform. MentraOS is the closest thing to "Android for glasses" currently shipping; runs on Z100 and G1; supports more hardware in 2025.

**Lessons:**
- 38g and 48-hour battery is exceptional — likely measured in mixed use, not single-purpose static text. Could plausibly extend significantly.
- Open software platform with TypeScript SDK lowers the build cost dramatically.
- An IMU is in every shipping AR glasses product; head-tilt as input is a software/UX choice, not a hardware gap.

**Relevance:** The fastest "build app, skip hardware" path. If the prototype goal is to validate UX rather than display tech, MentraOS on Z100 is arguably the right starting point.

#### [Halliday Glasses](https://hallidayglobal.com/) — brief mention
28.5g monocular direct-to-eye projection, 12-hour battery. Lightest display glasses found. Different optics path (closer to retinal projection than waveguide); unclear how comfortable for extended reading.

#### [Meta Ray-Ban Display (2025)](https://www.meta.com/ai-glasses/meta-ray-ban-display/) — brief mention
Wayfarer form factor, monocular waveguide, 600×600 full-color, $799. Paired with Neural Band (sEMG wristband). 6hr battery, 24hr with case. Indicative of where mainstream is going.

#### [ActiveLook / ENGO 2](https://engoeyewear.com/) — brief mention
Sport-frame microOLED HUD, 36g, BLE-only, very low power. Their power management approach is worth studying as a low-power always-on HUD reference.

---

### Theme C — Tethered display glasses (the mature mass market)

The XReal / Rokid / VITURE cluster has converged on ~75g, 50° FOV, micro-OLED, USB-C-tethered, 1080p. This class is not standalone (rules it out for choice-specs as-is) but defines the optics envelope and cost structure for the form factor.

#### [XReal One Pro](https://us.shop.xreal.com/products/xreal-one-pro) — brief mention
57° FOV, micro-OLED, 87g, has the X1 chip onboard for 3DoF stabilization (instructive — onboard compute for UX, not full standalone).

#### [Rokid Max](https://global.rokid.com/pages/rokid-max-specs) — brief mention
50° FOV, micro-OLED, 75g, $259. Built-in -6D myopia adjustment is worth studying for prescription compatibility.

#### [VITURE One](https://www.viture.com) — brief mention
50° FOV, micro-OLED, 78g, electrochromic dimming. Confirms the design envelope of the tethered class.

---

### Theme D — Open hardware AR platforms

#### [Project North Star](https://github.com/leapmotion/ProjectNorthStar) — assessed

Fully open-source wide-FOV AR headset using reflective half-silvered ellipsoidal mirror combiners and side-mounted 3.5" LCDs at 1600×1400/eye, 120 FPS, >100° combined FOV. ~$100–350 to build.

**Approach:** Reflective combiner optics — different class from birdbath/waveguide. Bulky development headset, not wearable as a final product.

**Health:** 1.8k stars, 272 forks, last pushed October 2025; Discord active.

**Lessons:**
- Canonical open-source AR optics reference. The optical design is overkill (wide FOV, binocular, large mirrors) for reading-only but the combiner geometry, calibration tooling, and STEP CAD files are reusable.
- Community is alive though the official repo is minimally maintained — the action is in forks and Discord.

**Relevance:** Knowledge source, not a build target. Useful for understanding reflective combiner tradeoffs vs. our likely pancake-lens or birdbath path.

#### [Mentra OpenSourceSmartGlasses](https://github.com/Mentra-Community/OpenSourceSmartGlasses) — brief mention
Hardware designs from a YC company that pivoted to MentraOS. Hardware repo is somewhat stale; the value moved to the software layer.

#### [Modos Caster (E Ink FPGA driver)](https://github.com/Modos-Labs) — assessed

Open-hardware FPGA controller (Xilinx Spartan-6) for driving E Ink panels at up to 75 Hz video rates. Modos Paper Monitor is a 13.3" e-ink monitor; the Dev Kit exposes the FPGA controller.

**Approach:** Open-source firmware for fast e-ink drive. Modos doesn't make microdisplays — their smallest panel is 13.3" — but the controller stack is the most accessible fast e-ink driver available.

**Health:** ~1,700 stars, last commit 2025, actively funded on Crowd Supply.

**Lessons:**
- If a small E Ink panel is sourced (direct from E Ink Corp partnership likely), Modos's Caster is the firmware to start from.
- Validates that fast-refresh e-ink at video rates is achievable open-source — historically e-ink was assumed slow because of stock controllers, not the panels.

**Relevance:** Direct building block for the e-ink path.

---

### Theme E — Display tech alternatives

#### [Daylight Computer DC-1](https://daylightcomputer.com) — brief mention
10.5" reflective IGZO-LCD tablet, no backlight, no blue light, 120Hz, $729. Demonstrates that reflective LCD at video rates is commercially viable — but no microdisplay-scale supplier surfaced.

#### [Kopin / ForthDD LCoS](https://www.kopin.com/technologies-products/commercially-available-products/microdisplays/) — assessed

Reflective LCoS microdisplays in sizes 0.2"–1" at resolutions up to 2048×2048. Commercially available. Used in the Divesoft dive mask (their 0.35" WQVGA part).

**Approach:** LCoS is reflective — needs an external illuminator, but with a warm/amber LED becomes a soft-light display closer to e-ink in feel than emissive OLED.

**Lessons:**
- Genuine sub-1" microdisplay path that's buyable today.
- Power cost of the illuminator is real but the illuminator can be dim and warm.
- Most mature non-OLED, non-microLED option for AR glasses scale.

**Relevance:** The most buildable non-e-ink path that preserves the eye-comfort intent. Likely the realistic display choice if the e-ink path proves too slow or unsourceable.

#### [JBD MicroLED](https://www.jb-display.com) — brief mention
Leading microLED microdisplay vendor. Tiny dies (0.13–0.26"), aggressively emissive. Eye-comfort hostile for bedtime; included as the dominant alternative if e-ink/LCoS paths fail.

#### [Retinal E-Paper (Uppsala / Nature 2025)](https://www.nature.com/articles/s41586-025-09642-3) — brief mention
Research-stage WO₃ nanodisk metapixel display, >25,000 PPI, ~50% optical contrast, >25Hz refresh, electrochemically driven. Not buildable — defines the long-term ceiling.

#### [Qualcomm Mirasol / IMOD (dead)](https://en.wikipedia.org/wiki/Interferometric_modulator_display) — brief mention
MEMS-based reflective interferometric display. Ideal on paper (zero hold power, ambient-light reflective, fast switching). Qualcomm shut it down by 2015. Entire MEMS-reflective category is a graveyard.

---

### Theme F — Compute platforms for the head

#### [Snapdragon AR1 Gen 1](https://www.qualcomm.com/xr-vr-ar/products/ar-series/snapdragon-ar1-gen-1-platform) — brief mention
Purpose-built smart-glasses SoC; powers Ray-Ban Meta. Closed proprietary stack, Meta-gated SDK. Reference architecture for <50g standalone glasses but not a personal-prototype target.

#### [Rockchip RK3566 / RK3588](https://eleclifeoem.com/product/ar-vr-smart-glasses-and-smart-helmets-motherboard-solution-based-on-rockchip-rk3588/) — assessed

RK3566: quad Cortex-A55 @ 1.8GHz, Mali-G52, runs Buildroot/Debian/Android. RK3588: quad A76 + quad A55 @ 2.4GHz. Used in many open-hardware boards (Radxa Zero 3, Pine64). XReal earlier glasses cited as using RK3568 (mid-point variant).

**Approach:** General-purpose Linux SBCs, well-documented. KOReader/MuPDF run natively.

**Lessons:** RK3566 is the practical sweet spot for a DIY head unit running Linux. Active power 1–3W requires 1000–2000 mAh battery for multi-hour use.

**Relevance:** Likely the realistic compute platform if we build custom hardware around an existing reader stack.

#### [ESP32-S3 + DIY EPUB reader (atomic14)](https://github.com/atomic14/diy-esp32-epub-reader) — assessed

Working DIY EPUB reader on ESP32-S3 + e-paper. Requires PSRAM (8MB recommended). Basic HTML tag subset, no CSS reflow engine. Active power ~150–250 mW.

**Health:** 762 stars, actively maintained.

**Lessons:**
- Proof that EPUB reading is feasible on a $5 MCU — but typography is primitive.
- Lowest-power option; matches sleep-context power budget cleanly.
- Acceptable if "readable" is enough; not enough if "beautiful typography" matters.

**Relevance:** The minimum-viable compute path. Worth validating as a fallback even if the eventual platform is more capable.

#### [Kobo Clara HD + KOReader](https://github.com/koreader/koreader) — assessed

Used Kobo Clara HD (~$30–50): NXP i.MX6 Cortex-A9 @ ~800MHz, 512MB LPDDR3, 8GB eMMC, Linux + Qt or framebuffer + KOReader. Active power ~300–500 mW.

**Approach:** Buy a hacked Kobo, install KOReader, validate the entire EPUB software stack on representative hardware before designing custom anything.

**Lessons:**
- Fastest path to validating typography, page-turn UX, and library management.
- i.MX6 + 512MB LPDDR3 is the proven minimum for "feels like a Kindle."
- KOReader is 26.5k stars, last release March 2026 — battle-tested on exactly this class of hardware.

**Relevance:** **The single most efficient prototype investment in this scout.** Validates the software stack without any hardware design. Buy it, run it, learn from it.

#### [Ambiq Apollo4](https://ambiq.com/apollo4/), [Allwinner V3s/T113](https://www.cnx-software.com/2022/02/13/allwinner-t113-s3-dual-core-cortex-a7-soc-features-128mb-ddr3-targets-automotive-and-industrial-applications/), [Raspberry Pi Zero 2W](https://www.cnx-software.com/2021/12/09/raspberry-pi-zero-2-w-power-consumption/) — brief mentions
Apollo4: ultra-low-power Cortex-M4F (~5µA/MHz); 1.8MB SRAM is tight for full EPUB reflow but works for pre-rendered bitmap pages. Allwinner V3s/T113: cheap Linux ARM, ~200–400mW. Pi Zero 2W: full Linux but ~300–600mW even optimized — marginal for head-worn.

---

### Theme G — Reader software stack

#### [KOReader](https://github.com/koreader/koreader) — assessed (gold standard)

Full-featured multi-format reader. Lua over C/C++ (MuPDF + crengine fork for EPUB reflow). Runs on Kindle, Kobo, PocketBook, reMarkable, Linux framebuffer. Confirmed minimum: NXP i.MX6 @ 1GHz with 512MB RAM.

**Lessons:** Battle-tested embedded EPUB stack; framebuffer-native; supports hyphenation, CSS, custom fonts. If our target Linux runs a framebuffer device, KOReader drops in.

**Relevance:** Default software target. Even if we eventually replace it, validating against KOReader first is the right test.

#### [Plato](https://github.com/baskerville/plato) — assessed

Rust-native document reader for Kobo. Custom EPUB reflow engine (not crengine). 98.5% Rust. Lighter memory footprint than KOReader.

**Lessons:** If you're building a custom Linux image and want auditable, portable code, Plato is closer to metal than KOReader.

**Relevance:** Strong second choice; better if the project values Rust portability over feature breadth.

#### [MuPDF Core](https://mupdf.com/core) — brief mention
C library for PDF/XPS/EPUB. ~2MB binary. AGPL. Pair with framebuffer write for a minimal EPUB viewer in ~3MB. Real runtime usage 10–30MB per rendered page.

#### [InkBox / Quill OS](https://github.com/Quill-OS/quill) — brief mention
Qt5-based open-source OS + reader for Kobo. Cleaner architecture than KOReader for those comfortable with Qt/C++.

---

### Theme H — Single-purpose e-reader prior art

#### [Kindle Paperwhite](https://www.ifixit.com/Device/Kindle_Paperwhite_11th_Generation), [Kobo](https://github.com/kobolabs/Kobo-Reader), [Onyx Boox](https://en.wikipedia.org/wiki/Onyx_Boox), [PocketBook](https://gbatemp.net/review/pocketbook-era-e-reader.2258/) — collective lessons

| Device | SoC | RAM | OS | UX philosophy |
|---|---|---|---|---|
| Kindle Paperwhite 11 | NXP i.MX6 | 512MB | Amazon Linux + custom UI | Library grid, tap-zone nav, ~3 screens total |
| Kobo Clara HD | NXP i.MX6 Solo | 512MB | Linux + Qt | More open, KFMon/InkBox extensible |
| Boox Mid/High | Snapdragon 750G | 4–8GB | Android | Full Android, any APK, $$$ |
| PocketBook Era | Dual ARM 2.1GHz | 512MB | Linux + custom SDK | Less community, native EPUB |

**Cross-cutting lessons:**
- The "library + reading + minimal settings" UX is a 3-screen pattern. Don't over-design.
- Multi-week standby is achievable via deep-sleep between page turns.
- Power profile for the UX class is 200–500mW active — the target for the head-worn version.
- Boox shows what happens when you go full Android (4–8GB RAM, much heavier silicon) — that's the wrong direction for choice-specs.

---

### Theme I — Head-tilt input research and accessibility prior art

#### [HeadGesture (ACM IMWUT 2018)](https://dl.acm.org/doi/10.1145/3287076) — assessed

Defines a hands-free head-gesture vocabulary for HMDs (nod, shake, tilt-left, tilt-right, composites) with on-device ML recognition. Performance comparable to mid-air hand gestures, less arm fatigue.

**Relevance:** Foundational academic reference for exactly this use case. Provides a starting gesture vocabulary.

#### [Head Gesture Recognition with 6DOF IMU (MDPI Sensors 2020)](https://www.researchgate.net/publication/340821722) — assessed
DTW / SVM / LSTM classifiers; single-IMU 4-class accuracy ~95%+ in upright posture. **Lying-down posture is documented as a known degradation case** — gravity vector shifts from Y to Z, de-anchoring standard reference frames.

#### [Design of Low-False-Positive Gesture (SciTePress 2016)](https://www.scitepress.org/papers/2016/57019/57019.pdf) — assessed
Compound gestures (tilt + 400ms hold, double-tilt) reduce false positives dramatically vs. single-axis tilts. Pillow repositioning is kinematically similar to intentional tilts — without compound design, false-positive rates run 10–25%.

#### [IMU Reliability over 1hr Sessions (Nature Sci Reports 2021)](https://www.nature.com/articles/s41598-021-99459-7) — assessed
Drift over 45–90min sessions is real and documented. Complementary or Kalman filter is required, not optional. "Stillness re-anchor" (frame reset during 1-second pause) is the practical mitigation used in production GlassOuse-class devices.

#### [GlassOuse V1.4](https://glassouse.com/) — assessed
Glasses-worn 9-axis gyroscope head mouse for accessibility users. 150+hr battery, BLE. Most mature consumer head-tracking input device. **Core lesson: the click is always the hard part.** For our use case, "tilt to page-turn" sidesteps this — the turn IS the gesture.

---

### Theme J — HUDs in other domains

The cross-domain consensus: **no active input on the glasses.** Successful deployments (cycling, motorcycle, military, surgical, industrial) all use either a peripheral (Snow2 wrist remote, HOTAS) or voice. Bedtime reading rules out voice — which validates the BT clicker fallback as the conservative-but-correct path.

#### [ENGO 2](https://engoeyewear.com/products/engo-2) — brief mention
Sport HUD, 36g, BLE-only, glance-only design — zero active input on device. All configuration pre-ride on phone.

#### [Recon Snow2 + wrist remote](https://appleinsider.com/articles/14/01/11/hands-on-recon-instruments-snow2-iphone-connected-heads-up-display-for-sports) — brief mention (discontinued)
Direct analog for our BT ring fallback. Snow2's wrist remote was a purpose-built single peripheral for HUD navigation while hands were occupied (gloves). D-pad: forward/back paging, home, select. **Discontinued because the market was too small in 2017, not because the UX failed.**

#### [BMW ConnectedRide Smartglasses](https://hiconsumption.com/motors/bmw-motorrad-connectedride-smartglasses/), [Augmedics xvision Spine](https://augmedics.com/xvision/) — brief mentions
Both are pure display, zero input. Reinforces the cleanliness of display-only + delegated input.

---

### Theme K — Bluetooth peripherals (the fallback input)

#### [Gevo BT Page Turner Ring](https://goodereader.com/blog/technology/gevo-bluetooth-page-turner-ring-will-allow-for-a-hands-free-reading-experience) — assessed

3-button ring (top/bottom = page forward/back, middle = pair). 14-hour battery, USB-C charging case. Pairs as BT keyboard (arrow keys).

**Lessons:** **The fallback input is fully de-risked.** Ring page turner for ebook reading is a solved $20–30 problem. Pairs as BT HID keyboard — arrow-right = next page, trivial integration.

#### [Kobo Remote](https://us.kobobooks.com/products/kobo-remote), [generic BT clickers](https://www.amazon.com/Bluetooth-Control-Clicker-Reader-Scrolling/dp/B0DJR1TJNR) — brief mentions
First-party validation of the form factor at hardware level. Commodity BT 5.0 clickers cost ~$15.

#### [Genki Wave Ring](https://genkiinstruments.com/) — brief mention
Over-featured ring (5 buttons + LED + haptic, BT MIDI, 4hr battery). Demonstrates ring-form-factor BT input with haptic feedback is shippable. **Haptic confirmation on page turn is a meaningful UX refinement** — feel the turn in the dark, no need to check visually.

---

### Theme L — Analogous: in-bed / sleep-context tech

#### [Bose Sleepbuds (discontinued)](https://ozlosleep.com/blogs/news/why-were-bose-sleepbuds-discontinued) — assessed cautionary tale

Two failure modes back-to-back: Gen I died from a battery defect (2019 recall + full refunds); Gen II died from "low demand" because users were locked into a preloaded ambient sound library with no streaming.

**Lessons applicable directly:**
- **Sleep-context hardware reliability is existential** — a device that fails at 2am is unforgivable.
- **Open content is a design gate** — locked content libraries kill adoption. Our open-EPUB-only choice already aligns.

#### [Bed Prism Glasses](https://www.magnifier.com/bed-prism-lazy-spectacles-supine-glasses-bpg-lw.htm) — assessed

90-degree optical prism glasses, $15–30, widely available. Let you read a book on your chest while looking at the ceiling.

**Lessons:** Validates the posture demand. Solves the neck problem but not the arm-holding problem. Heavy on the face, falls off when you turn your head. Our device must solve the same posture at near-zero weight (~50g vs. ~40–60g of these).

#### [Aura Smart Sleep Mask](https://aura-circle.com/products/aura-sleep-mask) — brief mention
Premium sleep mask + light bar + speakers + ambient sound. Proves users accept face-worn electronics for sleep if comfortable. No one has put a *visual display* in a sleep mask form factor at consumer price.

#### [Meta Quest Lying Down Mode](https://developers.meta.com/horizon/documentation/unity/ps-lying-down-mode/) + [Biblio VR Reader](https://www.meta.com/experiences/biblio-reading-app-for-books-and-comics-in-vr/7448147755222436/) — brief mentions
Meta acknowledges in-bed VR posture exists; experimental feature. **Vision Pro reviewers hit pain at 15–30 minutes** — the 50g weight cliff is real.

---

### Theme M — Single-purpose device philosophy

The category — Kindle, [Daylight Computer DC-1](https://www.tomsguide.com/tablets/android-tablets/daylight-dc1-is-a-new-tablet-with-a-live-paper-display-that-might-be-better-than-e-ink-heres-how), [Light Phone](https://insidetechworld.com/featured/light-phone-3-quest-for-minimal-phones/), [Onyx Boox Palma](https://news.ycombinator.com/item?id=45789405), [Freewrite](https://getfreewrite.com/), [Whispersync](https://www.amazon.com/Whispersync-for-Voice/b?ie=UTF8&node=12527156011) — points at a real cohort. Cross-cutting lessons:

- "Does less, but does it intentionally" is a viable premium brand position.
- E-ink's latency *is* a feature for focus — the medium itself discourages doomscrolling.
- The failure mode for focus devices is "novelty fades" when the constraint is more limiting than empowering. Reading at bedtime is a use case where the constraint (no notifications, no apps) maps cleanly to the activity (drift-off is incompatible with interruption).
- [Audible owns the bedtime market](https://publishdrive.com/the-audiobook-surge-in-2025-why-authors-cant-ignore-it.html), but Whispersync's existence proves users *want* to switch between modes. The visual-bedtime cohort exists.

---

## Cross-Cutting Design Signals

1. **No sub-inch e-ink microdisplay exists.** E Ink Corp's smallest panel is ~1.3"; Modos doesn't go below 13.3"; no other vendor surfaced. Sol Reader hit this and worked around it with pancake lenses + a 1.3"+ panel. **Realistic display options for the e-ink-aesthetic intent: (a) larger E Ink + pancake lens, (b) warm-illuminated LCoS, (c) low-refresh micro-OLED with software dimming. Each has a real research dig behind it.**

2. **For dark-room bedtime reading, see-through is unnecessary** — and that reframes the optics question. **Pancake lens (Sol Reader's choice) is simpler and more efficient than birdbath when see-through isn't required.** Birdbath's typical "ambient-light-blocking" con becomes neutral-to-positive in this use case.

3. **Cross-domain HUD consensus: no active input on the glasses.** Cycling, motorcycle, military, surgical, industrial — all use voice or peripheral. Voice is ruled out (partner sleeping). **The BT clicker fallback is the conservative-correct path.** Head-tilt as primary input is the *novel* path worth research, not a foregone conclusion.

4. **Head-tilt is feasible but lying-down posture is the hardest case.** Single-axis 4-class accuracy is ~95% in upright posture; supine drops because the gravity vector shifts and pillow repositioning is kinematically similar to intentional tilts. Mitigations: **compound gestures** (tilt + 400ms hold), **per-session calibration**, **velocity-profile filter**, **stillness re-anchor**. None are research-stage; all are well-understood from the literature. Implementation is the work, not invention.

5. **The 50g weight cliff is real.** Vision Pro users hit pain at 15–30 minutes. Glasses below ~50g cross from "noticeable" to "imperceptible" for long sessions. Our target is below the cliff. This is a category break, not a feature upgrade.

6. **The arm-fatigue problem is the dominant pain point** for bedtime visual reading — more cited than blue light, brightness, or weight in user forums. Hands-free is not a nice-to-have; it's the core job-to-be-done. **A wearable display directly solves this** in a way no other device class does.

7. **Sleep-context hardware has two existential gates** (per Bose Sleepbuds): reliability and open content. Choice-specs already aligns on both — open EPUB only, simple stack.

8. **The wedge cohort is real and underserved.** People who prefer visual reading at night, are dissatisfied with e-readers (arm fatigue) and audiobooks (can't re-read, lower retention) and VR (weight). Signals: prism-glasses market, Kindle arm-pain forums, DIY ceiling projector communities, Whispersync power users.

## Research Recommendations

Given the breadth and interconnectedness of unknowns, the recommendation is to **escalate to `/research-program`** rather than running individual `/research` tasks. Four campaigns, ~28 briefs, with cross-campaign synthesis at the end.

### Campaign 1: Display & Optics (highest-leverage; gates everything)
- E-ink microdisplay vendor landscape + direct E Ink Corp engagement (sample availability, MOQs, sub-inch roadmap)
- LCoS + warm-illuminator path (Kopin parts, illumination architecture, power)
- Micro-OLED at low refresh + dimming (eye comfort tradeoffs, warm-tinting)
- Reflective LCD options (Daylight-class panels at smaller form factor)
- Pancake lens design (Sol Reader path, sourcing, design files, optical efficiency)
- Birdbath optics for monocular non-see-through reading
- Reflective combiner / freeform prism alternatives
- Vergence-accommodation conflict + eye comfort literature for long reading sessions
- Sol Reader specific post-mortem (commercial vs. UX vs. supply chain failure analysis)
- Component sourcing as a hobbyist (vendor channels, MOQs, lead times)

### Campaign 2: Compute & Software Stack
- SoC selection: RK3566 / ESP32-S3 / i.MX6 / Pi Zero 2W tradeoff matrix
- EPUB renderer comparison: KOReader / Plato / MuPDF / custom
- OS choice: Linux / FreeRTOS / RTOS for a single-purpose head unit
- MentraOS deep evaluation as alternate "skip hardware" path (SDK capabilities, latency, IMU access, head-tilt feasibility)
- Library management UX for head-locked single-page interface
- Typography at low resolution / monochrome (640×200 case — what does a book actually look like?)
- Page rendering pipeline architecture (memory budget, cache strategy, refresh management)

### Campaign 3: Input & Interaction
- Head-tilt reliability in supine / reclined posture (full literature synthesis with mitigation strategies)
- IMU sensor fusion approaches (complementary / Kalman / ML — tradeoffs at this compute level)
- Compound gesture design and segmentation (false-positive minimization)
- Calibration / drift handling over 1+ hour sessions
- BT HID integration patterns (pairing UX, latency, reliability under blanket)
- Library navigation UX without a keyboard (head-tilt only; head-tilt + ring; voice was ruled out)
- Accessibility input prior art deep dive (GlassOuse, dwell click, breath / jaw / eye-blink alternatives)
- Peripheral fallback form factor (ring vs. clicker vs. wrist remote vs. pendant — hands-under-blanket optimization)

### Campaign 4: Industrial Design, Power, & Safety
- Weight distribution / center-of-mass for long-session glasses comfort
- Heat dissipation in glasses form factor (active power 200–500mW against the temple)
- Frame materials (TR-90 / titanium / magnesium / carbon-fiber / 3D-printed plastic — mass, thermal, cost)
- Battery cell selection + BMS + USB-C charging IC
- Power budget for chosen display+compute combo (is 2hr/night realistic? Multi-night standby?)
- Sleep-context safety (falling asleep with glasses on, breakage modes, materials safety, choking)
- 3D printing / fabrication for glasses frames (resin / SLS / FDM tradeoffs)
- Prescription lens compatibility (light pass; deferred to product phase but design must not preclude it)

**Sequencing:** Campaign 1 first (display gates everything — its outputs constrain compute, optics, and frame). Campaigns 2 and 3 in parallel after Campaign 1 lands. Campaign 4 last (depends on choices made in 1–3).

**Cross-campaign synthesis:** A program-level synthesis ties together design tradeoffs that span campaigns — e.g., display tech ↔ power budget ↔ frame thermal design, or input modality ↔ EPUB library navigation UX.

## Gaps

These are absences worth noting — either technology that should exist but doesn't, or search areas that came back thin and may warrant a second pass.

1. **No sub-1-inch e-ink microdisplay vendor.** Genuine technology gap. Confirms the e-ink path requires either direct E Ink Corp partnership or working around with a 1.3"+ panel and lens magnification.
2. **No open-hardware "Even G1 / Vuzix Z100 class"** — slim HUD glasses with waveguide. Project North Star is much heavier and lower-fi. There's no "North Star for slim HUD glasses."
3. **No shipping product uses head-tilt as a primary page-turn input.** IMU hardware is in everything. This interaction pattern itself is novel.
4. **No published battery-life numbers for static text + low refresh** in glasses. Everyone benchmarks active use. Choice-specs's actual use case may run dramatically longer than published specs — but no one has measured it.
5. **Pancake lens design files / suppliers for small-format near-eye use** were thin. Sol Reader's path, but no open designs surfaced. Worth a deeper dig in Campaign 1.
6. **IMU reliability literature for supine posture specifically** is thin. Most head-gesture papers are seated/upright. This is a real gap that Campaign 3 should fill empirically if needed (collect calibration data).
7. **Sleep-mask + display product** doesn't exist commercially. Face-worn electronics for sleep is a real category (Aura, Bose Sleepbuds) but no one has shipped a visual display in the form factor.
8. **Component sourcing knowledge** for hobbyists trying to build at this scale is fragmented across vendor sites, distributor catalogs, and forums. Campaign 1's sourcing brief is partly meta-research.

## Sources

### Display & optics
- [E Ink Corporation](https://www.eink.com)
- [Modos Paper Monitor — Crowd Supply](https://www.crowdsupply.com/modos-tech/modos-paper-monitor)
- [Modos Caster on GitHub](https://github.com/Modos-Labs)
- [Sol Reader (discontinued product page)](https://solreader.com/products/sol-reader)
- [Gizmodo: Sol Reader e-reader goggles](https://gizmodo.com/e-reader-goggles-hmd-kindle-e-ink-e-paper-vr-glasses-1849965229)
- [Retinal E-Paper (Nature 2025)](https://www.nature.com/articles/s41586-025-09642-3)
- [Daylight Computer DC-1](https://daylightcomputer.com)
- [Kopin / ForthDD LCoS](https://www.kopin.com/technologies-products/commercially-available-products/microdisplays/)
- [Qualcomm Mirasol — Wikipedia](https://en.wikipedia.org/wiki/Interferometric_modulator_display)
- [JBD MicroLED](https://www.jb-display.com)
- [KGOnTech: Nreal birdbath teardown](https://kguttag.com/2021/06/01/nreal-teardown-part-1-clones-and-birdbath-basics/)
- [KGOnTech: Birdbath pros/cons (2017)](https://kguttag.com/2017/03/03/near-eye-bird-bath-optics-pros-and-cons-and-immys-different-approach/)
- [DisplayModule Birdbath Dev Kit](https://www.displaymodule.com/products/binocular-birdbath-ar-headset-development-kit)
- [Frontiers: AR Near-Eye Displays Review](https://www.frontiersin.org/journals/virtual-reality/articles/10.3389/frvir.2022.838237/full)
- [ScienceDirect: Compact off-axis lightguide prism](https://www.sciencedirect.com/science/article/abs/pii/S0030401817301426)

### AR glasses platforms
- [XREAL One Pro](https://us.shop.xreal.com/products/xreal-one-pro)
- [XREAL Air 2 Pro](https://eu.shop.xreal.com/products/xreal-air-2-pro)
- [Rokid Max specs](https://global.rokid.com/pages/rokid-max-specs)
- [VITURE XR Glasses](https://www.viture.com)
- [Vuzix Z100](https://www.vuzix.com/products/z100-smart-glasses)
- [Vuzix M400](https://www.vuzix.com/products/m400-smart-glasses)
- [Magic Leap 2](https://www.magicleap.com/magic-leap-2)
- [Apple Vision Pro specs](https://www.apple.com/apple-vision-pro/specs/)
- [Meta Ray-Ban Display](https://www.meta.com/ai-glasses/meta-ray-ban-display/)
- [Even Realities G1](https://www.evenrealities.com/g1)
- [Halliday Glasses](https://hallidayglobal.com/)
- [ActiveLook](https://www.activelook.net/)
- [Lynx R1](https://vr-compare.com/headset/lynxr1)
- [Project North Star — GitHub](https://github.com/leapmotion/ProjectNorthStar)
- [Project North Star Docs](https://docs.projectnorthstar.org/)
- [MentraOS — GitHub](https://github.com/Mentra-Community/MentraOS)
- [BasedHardware OpenGlass](https://github.com/BasedHardware/OpenGlass)

### Compute & software
- [Snapdragon AR1 Gen 1](https://www.qualcomm.com/xr-vr-ar/products/ar-series/snapdragon-ar1-gen-1-platform)
- [Ambiq Apollo4](https://ambiq.com/apollo4/)
- [Rockchip RK3588 motherboard](https://eleclifeoem.com/product/ar-vr-smart-glasses-and-smart-helmets-motherboard-solution-based-on-rockchip-rk3588/)
- [KOReader — GitHub](https://github.com/koreader/koreader)
- [Plato — GitHub](https://github.com/baskerville/plato)
- [MuPDF Core](https://mupdf.com/core)
- [DIY ESP32 EPUB Reader (atomic14)](https://github.com/atomic14/diy-esp32-epub-reader)
- [Allwinner T113-S3](https://www.cnx-software.com/2022/02/13/allwinner-t113-s3-dual-core-cortex-a7-soc-features-128mb-ddr3-targets-automotive-and-industrial-applications/)
- [Pi Zero 2W power](https://www.cnx-software.com/2021/12/09/raspberry-pi-zero-2-w-power-consumption/)
- [Kobo Clara HD postmarketOS](https://www.cnx-software.com/2021/07/13/diy-linux-e-reader-postmarketos-now-runs-on-kobo-clara-hd/)
- [Kobo SDK](https://github.com/kobolabs/Kobo-Reader)
- [KFMon](https://github.com/NiLuJe/kfmon)
- [InkBox / Quill OS](https://github.com/Quill-OS/quill)
- [Kindle Paperwhite iFixit](https://www.ifixit.com/Device/Kindle_Paperwhite_11th_Generation)
- [MobileRead Wiki — KOReader](https://wiki.mobileread.com/wiki/KOReader)

### Input & UX
- [HeadGesture — ACM IMWUT 2018](https://dl.acm.org/doi/10.1145/3287076)
- [Headar — ACM IMWUT 2023](https://dl.acm.org/doi/10.1145/3610900)
- [Head Gesture Recognition with 6DOF IMU](https://www.researchgate.net/publication/340821722)
- [VR Helmet Pattern Recognition — MDPI Sensors 2019](https://www.mdpi.com/1424-8220/19/24/5408)
- [Low-False-Positive Gesture Design — SciTePress 2016](https://www.scitepress.org/papers/2016/57019/57019.pdf)
- [Real-Time Head Gesture Recognition — arXiv 2017](https://arxiv.org/pdf/1707.06691)
- [IMU Reliability Study — Nature Sci Reports 2021](https://www.nature.com/articles/s41598-021-99459-7)
- [GlassOuse Head Mouse](https://glassouse.com/)
- [BlyncSync — CHI 2020](https://dl.acm.org/doi/10.1145/3313831.3376132)
- [ENGO 2 Cycling HUD](https://engoeyewear.com/products/engo-2)
- [Solos AirGo 3 Review](https://www.gearbrain.com/solos-airgo3-smartglasses-review-ai-2672130005.html)
- [Recon Snow2 Hands-On](https://appleinsider.com/articles/14/01/11/hands-on-recon-instruments-snow2-iphone-connected-heads-up-display-for-sports)
- [BMW ConnectedRide Smartglasses](https://hiconsumption.com/motors/bmw-motorrad-connectedride-smartglasses/)
- [HOTAS — Wikipedia](https://en.wikipedia.org/wiki/HOTAS)
- [Augmedics xvision Spine](https://augmedics.com/xvision/)
- [Gevo BT Page Turner Ring](https://goodereader.com/blog/technology/gevo-bluetooth-page-turner-ring-will-allow-for-a-hands-free-reading-experience)
- [Kobo Remote](https://us.kobobooks.com/products/kobo-remote)
- [Genki Wave Ring](https://genkiinstruments.com/)

### Analogues
- [Audiobook Surge in 2025 — PublishDrive](https://publishdrive.com/the-audiobook-surge-in-2025-why-authors-cant-ignore-it.html)
- [Whispersync for Voice](https://www.amazon.com/Whispersync-for-Voice/b?ie=UTF8&node=12527156011)
- [Apple Vision Pro Reading — Publishers Weekly](https://www.publishersweekly.com/pw/by-topic/digital/devices/article/95543-can-apple-s-vision-pro-usher-in-a-new-era-for-reading.html)
- [Bed Prism Glasses](https://www.magnifier.com/bed-prism-lazy-spectacles-supine-glasses-bpg-lw.htm)
- [Bose Sleepbuds Discontinued — Ozlo](https://ozlosleep.com/blogs/news/why-were-bose-sleepbuds-discontinued)
- [Aura Smart Sleep Mask](https://aura-circle.com/products/aura-sleep-mask)
- [Daylight DC-1 — Tom's Guide](https://www.tomsguide.com/tablets/android-tablets/daylight-dc1-is-a-new-tablet-with-a-live-paper-display-that-might-be-better-than-e-ink-heres-how)
- [Boox Palma — Hacker News](https://news.ycombinator.com/item?id=45789405)
- [Light Phone 3](https://insidetechworld.com/featured/light-phone-3-quest-for-minimal-phones/)
- [Quest Lying Down Mode](https://developers.meta.com/horizon/documentation/unity/ps-lying-down-mode/)
- [Biblio VR Reading App](https://www.meta.com/experiences/biblio-reading-app-for-books-and-comics-in-vr/7448147755222436/)
- [Read to Sleep Survey — Penguin Random House](https://www.penguinrandomhouse.com/articles/read-to-sleep-survey-results/)
- [Kindle Holders for Reading in Bed](https://rafalreyzer.com/kindle-holders-for-reading-in-bed/)
- [Freewrite](https://getfreewrite.com/)
