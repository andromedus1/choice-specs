# North Star: choice-specs

*Last updated: 2026-04-25*

## Vision

A standalone wearable that lets you read books in bed without holding anything. Glasses form factor, single purpose, designed for hands-under-blankets bedtime reading.

## Problem

Bedtime reading is the canonical use case for the visual reading market — and it's the worst-served. Each existing option fails in a specific way:

- **Kindles and physical books** — arm and wrist fatigue from holding a device for an hour. The dominant pain point in user forums; multiple product categories (gooseneck mounts, pillow stands, prism glasses) exist solely to address it.
- **Audiobooks** — won the bedtime market by removing physical cost, but readers consistently cite lower retention, no re-reading, no scanning, no page-spatial memory. Audible owns the comfort tier; visual readers feel underserved.
- **Phones and tablets** — bright, blue-light-rich, notification-rich. Not made for sleep.
- **VR headsets** — Vision Pro reviewers hit pain at 15–30 minutes; 500g+ on your face supine is a non-starter.
- **Bed-prism reading glasses** — solve the neck-angle problem but not the arm-holding problem; heavy on the face; fall off.

The underserved cohort is real: people who prefer visual reading at night, find audiobooks insufficient for retention or re-reading, and would pay for a device that solves the form factor. The closest active product — **Sol Reader** — is a binocular goggle e-reader at $249 with 256×256/eye and a handheld remote. It validates that the category exists but leaves the **glasses form factor**, **monocular**, **lighter (<50g)**, and **hands-free input** axes still open. (See [Campaign 1 brief 9](../programs/choice-specs-feasibility/campaigns/01-display-and-optics/briefs/09-sol-reader-postmortem.md) for full status and direct A/B; scout-landscape's earlier "discontinued" claim was incorrect and has been corrected.)

## Principles

### 1. Single purpose, no compromise
Every feature is reading. If it's not reading, it doesn't exist on this device. No notifications, no apps, no shortcuts to other content.

### 2. Hands stay under the blankets
Input must work without removing arms from beneath the covers. The peripheral fallback is OK if it can be held in a closed palm or worn on a finger; head-tilt is preferred. This rules out voice (partner sleeping), eye tracking (dark room), held devices, and frame-side touchpads.

### 3. Eye-comfort beats eye-candy
Display tech, brightness, refresh, focal distance — all optimized for "read 90 minutes, sleep well." Beats any tradeoff prioritizing vivid colors, refresh rate, or animation.

### 4. Open formats only
The device renders EPUB. No DRM. Users own their library. Importing content from other ecosystems (Kindle, Adobe) is a separate off-device pipeline — not a device concern.

### 5. Sleep-grade reliability
A device that fails at 2am is unforgivable. Boring and predictable is the goal. No flaky firmware, no spinning wheels, no "syncing now."

### 6. Below the weight cliff
Target ≤ 50g — the threshold below which glasses become imperceptible for long sessions. Every gram of feature creep is a vote against the core experience.

### 7. Personal first, product later
Optimize for "works for one user (me)" before "works for everyone." Prescription support, multi-user adjustability, and accessibility breadth are valid product concerns, but they slow the prototype.

## Out of Scope

Explicit exclusions to keep the project tight:
- Multi-purpose AR (no apps, no notifications, no assistant)
- Phone tethering
- SLAM / spatially-anchored content
- Native Kindle DRM rendering
- Side-sleeping or flat-on-back postures
- Bookmarks, highlights, notes, dictionary lookup
- Voice interaction
- Camera, microphone, recording
- Audio output (deferred)
- Multi-user (deferred to product phase)
- Prescription-adaptable hardware (deferred to product phase, but design must not preclude it)

## Success Criteria

**Personal feasibility prototype is successful when:**
- The user can wear it in bed for 60+ minutes without arm, neck, or eye discomfort.
- Page-turn intent succeeds > 95% of the time.
- Library browsing and book-switching work without removing hands from covers.
- The display is comfortable enough that the user prefers it to a Kindle for bedtime reading.
- Battery lasts a full evening session (~ 2 hours) without charging.
- It doesn't fall off during normal head movement.
- It doesn't break if the user falls asleep wearing it.

**Product-gate criterion** (when to consider productizing):
- All personal-feasibility criteria met.
- Core experience replicable across face shapes and prescriptions.
- BOM realistic for a $400–700 product price point.

## Domain Model

### Reader
The user wearing the device. For v1, a single reader is assumed.

### Library
The reader's collection of books, stored on the device.

### Book
An EPUB file with content and metadata (title, author, etc.). Pre-converted before transfer; the device renders, it does not import.

### Session
A continuous reading period. Begins when the device is donned and a book is opened; ends when removed, when the user falls asleep, or after a configurable idle.

### Page
What the reader currently sees. Reflowed at the chosen typography settings, head-locked in their visual field.

### Position
The current location in a book. Persists across sessions per book.

### Reading config
Typography settings — font size today; later potentially line height, font family, hyphenation. Per-book overrides global default.

### Input gesture
A head-tilt or peripheral button-press, mapped to an action (next page, previous page, library, settings). Specific implementation depends on Campaign 3 research.

### Display state
The current top-level mode: library (browse books), book (reading), or settings (typography, device).

### Import pipeline (off-device)
The tooling — Calibre, DeDRM, ad-hoc converters — that produces EPUBs from user content (Kindle KFX, Adobe-DRM PDFs, raw EPUBs, etc.). Lives off-device. Not a choice-specs component, but documented as a workflow for v1; potentially wrapped in a desktop helper in a later version.

## Research Plan

A coordinated `/research-program` of 4 campaigns, ~28 briefs total. **Sequencing:** Campaign 1 first (display gates everything); Campaigns 2 and 3 in parallel after; Campaign 4 last. Cross-campaign synthesis at the end ties tradeoffs that span campaigns (e.g., display tech ↔ power budget ↔ frame thermal design).

### Campaign 1 — Display & Optics *(highest-leverage; gates everything)* — ✅ complete (2026-04-25)
**Output:** [campaigns/01-display-and-optics/parent.md](../programs/choice-specs-feasibility/campaigns/01-display-and-optics/parent.md) — synthesis recommends monocular pancake (salvaged Quest 3) + Pervasive Displays e-ink + virtual page at 50cm + dominant eye + occluded other eye. Display fallback order: e-ink → LCoS+amber → micro-OLED. ~$2k prototype budget.

- [x] **E-ink microdisplay vendor landscape + direct E Ink Corp engagement** — confirms sub-1" gap; Pervasive Displays via DigiKey is the path. *([brief](../programs/choice-specs-feasibility/campaigns/01-display-and-optics/briefs/01-eink-microdisplay-landscape.md))*
- [x] **LCoS + warm-illuminator path** — Kopin 0.20"–0.88" with amber LED is the credible non-e-ink fallback. *([brief](../programs/choice-specs-feasibility/campaigns/01-display-and-optics/briefs/02-lcos-warm-illuminator.md))*
- [x] **Micro-OLED at low refresh + dimming** — third-priority fallback; never reaches still-as-paper feel. *([brief](../programs/choice-specs-feasibility/campaigns/01-display-and-optics/briefs/03-microoled-low-refresh.md))*
- [x] **Reflective LCD options** — rejected; resolution insufficient + reflective premise fails in dark room. *([brief](../programs/choice-specs-feasibility/campaigns/01-display-and-optics/briefs/04-reflective-lcd.md))*
- [x] **Pancake lens design** — recommended optical path; salvaged Quest 3 lens at $200–400. *([brief](../programs/choice-specs-feasibility/campaigns/01-display-and-optics/briefs/05-pancake-lens.md))*
- [x] **Birdbath optics for monocular non-see-through reading** — secondary path; DisplayModule kit at $400–1.2k. *([brief](../programs/choice-specs-feasibility/campaigns/01-display-and-optics/briefs/06-birdbath-optics.md))*
- [x] **Reflective combiner / freeform prism alternatives** — all rejected (overkill / unbuildable / patent-locked). *([brief](../programs/choice-specs-feasibility/campaigns/01-display-and-optics/briefs/07-reflective-combiner-freeform-prism.md))*
- [x] **Vergence-accommodation conflict + eye-comfort literature** — virtual page at 50cm (2.0 D), monocular, occluded unviewing eye. *([brief](../programs/choice-specs-feasibility/campaigns/01-display-and-optics/briefs/08-vergence-accommodation.md))*
- [x] **Sol Reader status, lessons, direct comparison** — **correction:** Sol is shipping at $249, not discontinued; treat as production reference + buy for empirical baseline. *([brief](../programs/choice-specs-feasibility/campaigns/01-display-and-optics/briefs/09-sol-reader-postmortem.md))*
- [x] **Component sourcing as a hobbyist** — ~$2k buildable via DigiKey/Mouser/Adafruit + DisplayModule + eBay salvage. *([brief](../programs/choice-specs-feasibility/campaigns/01-display-and-optics/briefs/10-component-sourcing.md))*

### Campaign 2 — Compute & Software Stack
- [ ] **SoC selection tradeoff matrix** — RK3566 / ESP32-S3 / i.MX6 / Pi Zero 2W — power, RAM, dev experience, EPUB compatibility *(scout)*
- [ ] **EPUB renderer comparison** — KOReader / Plato / MuPDF / custom *(scout)*
- [ ] **OS choice** — Linux / FreeRTOS / RTOS for a single-purpose head unit *(scout)*
- [ ] **MentraOS deep evaluation as alternate "skip hardware" path** — SDK capabilities, latency, IMU access, head-tilt feasibility on Vuzix Z100 / Even G1 *(scout)*
- [ ] **Library management UX for head-locked single-page interface** — how do you browse without a keyboard? *(domain analysis)*
- [ ] **Typography at low resolution / monochrome** — Even G1's 640×200 case; what does a book actually look like? *(scout + domain analysis)*
- [ ] **Page rendering pipeline architecture** — memory budget, cache strategy, refresh management *(domain analysis)*
- [ ] *(Library navigation UX moved to Campaign 3 — single owner for screens design + input modality.)*

### Campaign 3 — Input & Interaction
- [ ] **Head-tilt reliability in supine / reclined posture** — full literature synthesis with mitigation strategies *(scout)*
- [ ] **IMU sensor fusion approaches** — complementary / Kalman / ML — tradeoffs at this compute level *(scout)*
- [ ] **Compound gesture design and segmentation** — false-positive minimization *(scout)*
- [ ] **Calibration and drift handling over 1+ hour sessions** — stillness re-anchor, per-session calibration *(scout)*
- [ ] **BT HID integration patterns** — pairing UX, latency, reliability under a blanket *(scout + domain analysis)*
- [ ] **Library navigation UX (screens + input together)** — how does the library view render *and* how do you traverse it? Head-tilt only; head-tilt + ring; voice ruled out. Single owner for the whole nav experience. *(domain analysis)*
- [ ] **Accessibility input prior art deep dive** — GlassOuse, dwell click, breath / jaw / eye-blink alternatives *(scout)*
- [ ] **Peripheral fallback form factor** — ring vs. clicker vs. wrist remote vs. pendant; hands-under-blanket optimization *(scout)*

### Campaign 4 — Industrial Design, Power, & Safety
- [ ] **Weight distribution / center-of-mass for long-session glasses comfort** *(domain analysis)*
- [ ] **Heat dissipation in glasses form factor** — active power 200–500 mW against the temple *(domain analysis)*
- [ ] **Frame materials** — TR-90 / titanium / magnesium / carbon-fiber / 3D-printed plastic *(domain analysis)*
- [ ] **Battery cell selection + BMS + USB-C charging IC** — sleep-context safety; Bose Sleepbuds lesson *(scout + domain analysis)*
- [ ] **Power budget for chosen display + compute combo** — is 2 hr/night realistic? Multi-night standby? *(domain analysis)*
- [ ] **Sleep-context safety** — falling asleep with glasses on, breakage modes, materials safety, choking *(domain analysis)*
- [ ] **3D printing / fabrication for glasses frames** — resin / SLS / FDM tradeoffs *(domain analysis)*
- [ ] **Prescription lens compatibility** — light pass; deferred to product phase but design must not preclude it *(domain analysis)*

## Related Documents

| Document | Purpose |
|----------|---------|
| [Scout Landscape](scout-landscape.md) | Prior art, adjacent approaches, lessons learned |
| Research-program plan *(TBD — `docs/programs/`)* | 4-campaign plan, produced by `/research-program` |
| Domain briefs *(TBD — `docs/briefs/`)* | Output of the research program |
| Architecture *(TBD)* | Technical design — modules, data flow, conventions (after research) |
| Roadmap *(TBD)* | Phased build plan (after architecture) |
