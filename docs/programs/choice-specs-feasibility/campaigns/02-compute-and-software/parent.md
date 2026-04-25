---
description: "Campaign 2 (Compute & Software Stack) synthesis — i.MX6 Solo + Buildroot Linux + KOReader on a custom carrier, validated first via Kobo Clara HD. MentraOS path rejected as primary."
type: brief
campaign: 02-compute-and-software
program: choice-specs-feasibility
research_method: deep-research
content_type: campaign-synthesis
confidence: speculative
status: draft
updated: 2026-04-25
---

# Campaign 2 — Compute & Software Stack: Synthesis

## Campaign question

Given Campaign 1's display + optical inheritance (monocular pancake + Pervasive Displays e-ink + 50 cm focal distance + Modos Caster as optional fast-refresh path; ~300–500 mW system active typical; ≤50g weight target; sleep-grade reliability principle), what runs in the glasses? Specifically:

- Which SoC?
- Which OS?
- Which EPUB renderer?
- Custom-build vs MentraOS-on-existing-hardware (the meta-decision that reframes the entire program if MentraOS works)?
- How does typography work at our display class?
- What's the page-rendering pipeline?

This campaign covered six sub-briefs. Three of them have direct decision outputs (SoC, OS, renderer); three are evaluations or architecture sketches (MentraOS, typography, pipeline). One brief — the MentraOS evaluation — is the highest-leverage decision in the program; if MentraOS had been a viable path, choice-specs becomes a software project. It isn't, so choice-specs stays a hardware project.

## Decision: SoC — **NXP i.MX6 Solo**

Validated first via off-the-shelf Kobo Clara HD ($30–50 used); migrated to a custom i.MX6 Solo SoM on a small carrier PCB for the worn prototype.

Rationale:
- KOReader runs natively on i.MX6 Solo today — proven, gold-standard EPUB stack on this exact silicon.
- Suspend-to-RAM is mature on i.MX6 with NXP's BSP — ~10 mW DSM achieved in Kobo production firmware.
- Active power profile is the second-best of realistic Linux SoCs (only Allwinner T113-S3 ties); ~30 mWh per 90-min session inside our budget.
- **Day-1 empirical validation path:** buy a Kobo Clara HD, install KOReader. Whole software stack live in an afternoon. **No other candidate has this property.**
- Migration path to custom carrier is bounded (i.MX6 SoMs from Toradex, Variscite available; verify EPDC variant).

Fallback: **ESP32-S3 + 8 MB PSRAM + atomic14-style EPUB reader.** Selected if i.MX6 SoM sourcing collapses or if power budget tightens further. Cost is typography quality — accept "readable" not "Kindle-grade" as a v1 ceiling.

Rejected: RK3566 (500 mW idle floor swallows the entire power budget); Pi Zero 2W (no documented sleep state — fatal for deep-sleep architecture); Allwinner T113-S3 (smaller ecosystem, would need fresh BSP work); Apollo4 (2.75 MB SRAM ceiling can't fit a CSS-reflow EPUB engine); AR1 Gen 1 (closed Meta-gated SDK).

See [brief 1](briefs/01-soc-selection.md) for full matrix and power math.

## Decision: OS — **Buildroot Linux on i.MX6 Solo**

NXP's i.MX6 BSP for kernel; Buildroot for userspace; simple shell `/init` script (no systemd) that execs `koreader.lua`; read-only root + small `/var` overlay; U-Boot dual-partition update via USB-C OTG mass-storage.

Rationale:
- Buildroot fits a single-developer single-purpose project (small image, fast iteration, comprehensible mental model). Yocto is overengineered for this scope; Debian/Ubuntu is power-hungry by default.
- Linux is mandatory for KOReader/Plato (both assume `/dev/fb0`).
- Suspend-to-RAM (`mem`) on i.MX6 → ~10 mW DSM with mature BSP support.
- Wake-from-suspend latency ~150 ms; total page-turn UX latency ~350 ms inside the 500 ms budget.
- Boot tuning: 3-second cold boot achievable with minimal init scripts.

Fallback: **FreeRTOS via ESP-IDF** (tied to ESP32-S3 SoC fallback). Native, well-supported, 250 ms wake from deep sleep, OTA support.

Rejected: Yocto (overengineering); Debian/Ubuntu (power overhead); Zephyr (would have been right for Apollo4, which was rejected upstream); bare-metal (concurrency surface needs scheduler); custom microkernel (not warranted by reliability bar).

See [brief 3](briefs/03-os-choice.md) for full reasoning.

## Decision: Renderer — **KOReader (crengine), lightly skinned**

Rationale:
- crengine is the most mature open-source EPUB engine; ~50 hyphenation languages, full CSS support, language-tuned typography ([poire-z's PR series in crengine](https://github.com/koreader/crengine/pulls?q=author%3Apoire-z)).
- Memory footprint (80–150 MB RSS on i.MX6 Solo + 512 MB) fits comfortably.
- Battle-tested e-ink refresh integration (partial / full / flashing modes; ghost-load tracking; periodic flash refresh).
- Lua plugin architecture means choice-specs's UX customizations (head-locked single-page view, custom input event routing, hand-off to Campaign 3 library nav) are small Lua patches, not C forks.
- AGPL inheritance via MuPDF is a non-issue at personal-prototype scope; revisit at product phase.

Fallback: **Plato** (Rust, smaller memory footprint ~40-80 MB) if RSS contention forces it. Less likely given 512 MB headroom.

Renderer-of-last-resort if ESP32-S3 SoC fallback is taken: atomic14's [diy-esp32-epub-reader](https://github.com/atomic14/diy-esp32-epub-reader) reference codebase as the starting point. Accepts typography downgrade in exchange for not needing Linux.

Custom Rust/C from scratch on Linux: rejected; Plato already exists at MIT and covers that role.

See [brief 2](briefs/02-epub-renderer.md) for the matrix.

## Decision: Custom-build vs MentraOS — **Custom-build is primary. MentraOS rejected as build target.**

The most consequential decision in this campaign. Three converging reasons:

1. **The display capability on supported MentraOS hardware is below Sol Reader's already-criticized baseline.** Vuzix Z100 doesn't support bitmap rendering through MentraOS at all (per [glasses-compatibility.md](https://github.com/Mentra-Community/MentraOS/blob/main/glasses-compatibility.md)) — only text strings via the LayoutManager API. Even Realities G1 supports bitmaps but at 640×200 monochrome green, the panel produces only ~30–40 words per page at readable font sizes — worse than Sol's already-criticized resolution. We cannot deliver a Kindle-class typography experience on either device.

2. **MentraOS's architecture requires a phone tether** (glasses ↔ BLE ↔ phone ↔ HTTP ↔ cloud ↔ HTTPS ↔ app server). This violates principle 1 (no phone tethering) and the explicit Out-of-Scope item in [north-star.md](../../../architecture/north-star.md). The principle was made for exactly this kind of decision.

3. **IMU access and BT clicker pairing are uncertain** — MentraOS's marketing claims "sensor access" but the public docs don't surface a documented IMU API, and event round-trip latency through the cloud-mediated architecture is plausibly 200–500 ms — at the edge of "feels responsive" for page-turn input.

The custom-build path on i.MX6 + KOReader provides Sol-baseline display quality (with the right Pervasive Displays panel), standalone operation, direct IMU access at <5 ms, and direct BT HID pairing.

**Useful subsidiary recommendation:** **buy an Even Realities G1 for $599 as a UX-validation toy** — write a 200-line MentraOS app that pages text through small string snippets with a paired BT ring, validate the *interaction model* before committing months of custom-hardware work. **Treat as usability prototype, not as deliverable.** With Sol Reader discontinued (per [Campaign 1 brief 9](../01-display-and-optics/briefs/09-sol-reader-postmortem.md)), G1 is the closest available shipping head-worn device for empirical UX validation, despite its display being below Sol's. Conditionally; if budget is tight, skip and rely on Kobo Clara HD + bench-mocked panels as the empirical baseline.

When MentraOS becomes preferable: if the platform adds a standalone runtime (apps run on glasses without phone+cloud), or if a vendor ships >1 megapixel monochrome displays in glasses form factor at <50g with bitmap support and MentraOS exposes it. None of these conditions are met today. **Re-evaluate in 12 months.**

See [brief 4](briefs/04-mentraos-evaluation.md) for the full evaluation.

## Decision: Typography strategy

**Minimum acceptable native panel resolution: ~40,000 pixels (≈ Pervasive Displays 2.9" 296×128 floor).**

This rules out 1.54" 200×200 (40k pixels — at floor) and 2.13" 250×122 (30k — below Sol baseline, below floor) for the worn prototype. Brings into target: 2.9" 296×128, 4.2" 400×300, or a custom higher-PPI panel co-engineered with E Ink (deferred to product phase).

**Target default: Pervasive Displays 2.9" 296×128.** Have a 4.2" 400×300 in the parts pool for second-build iteration if frame depth allows. Use the 1.54" 200×200 panel from [Campaign 1's sourcing brief](../01-display-and-optics/briefs/10-component-sourcing.md) for **bench bringup only** (validate driver, FBuf write loop, refresh policy) — not as the worn-prototype display.

**Body font default:** Literata 14–16 px, 1.4× line-height, left-aligned (ragged-right), no hyphenation by default at narrow column widths. Justification + hyphenation available in settings.

**Refresh policy default:** partial refresh on page turn, full flashing every 6th turn (KOReader `EpubFlashingPages = 6`).

**Fallback typography** if forced onto a 200×200 panel: 12 px body, 1.3× leading, 2 px margins, ~70 words per page, accept 5× page-turn frequency. Treat as v1; plan v2 around larger panel.

The "300 PPI virtual" Sol-style magnification metric is misleading: pancake aberration + contrast loss + total-pixel-count drives perceived sharpness more than angular pixel density. Words-per-page is the binding metric; physical panel size matters more than panel DPI.

See [brief 5](briefs/05-typography-low-res.md) for the math + ranking.

## Decision: Page rendering pipeline architecture

Eight modules, top to bottom of the stack:

1. **InputAdapter** (Campaign 3 owned, contract specified here): emits `{type, name, confidence, ts}` events to AppController. Source-agnostic — BT HID, IMU classifier, future eye-blink — all upstream.
2. **AppController** (Lua): top-level state machine: library | reading | settings.
3. **ReadingView** (Lua, KOReader-skinned): single-page head-locked, no chrome.
4. **BookLoader**: EPUB unzip + manifest parse + spine.
5. **ReflowEngine** (crengine): HTML/CSS → box tree → flat page index.
6. **Pager**: maps page_id → DOM range; manages 4-page rasterized bitmap cache (current ±1, +2 prefetch); background prefetch coroutine.
7. **Rasterizer** (FreeType + crengine glyph cache): glyphs → 296×128 8bpp bitmap.
8. **FrameBufferDriver + EinkRefreshPolicy**: writes EPDC framebuffer; selects refresh mode; tracks ghost-load.

**Cold boot to first page: ~3.7–4.0 s.** Wake-from-suspend to next page: **~600 ms.** Memory peak: **~180 MB on 512 MB platform** (comfortable). Energy per 90-min session: **~21 mWh** (a 500 mWh battery supports ~24 sessions).

**Suspend on 30 s idle.** Wake on BT IRQ or IMU IRQ. Hardware watchdog (i.MX6 integrated WDOG) as failsafe against resume hangs.

**Modos Caster: optional Phase 2 upgrade.** Phase 1 uses i.MX6's native EPDC peripheral (the path Kobo ships in production). Caster's value (75 Hz refresh) is unnecessary for static EPUB text.

See [brief 6](briefs/06-rendering-pipeline.md) for the full sketch + memory budget per stage.

## Cross-cutting findings

### 1. The Day-1 empirical validation path is even higher leverage than Campaign 1 suggested

[Campaign 1 named Kobo Clara HD as a $30–50 software-stack baseline.](../01-display-and-optics/parent.md) Campaign 2 confirms and reinforces: **Kobo Clara HD is not a baseline; it's the literal SoC + RAM + KOReader build we're committing to.** The migration from Kobo to a custom i.MX6 carrier preserves the entire software stack. Skipping Kobo is lighting money on fire.

With Sol Reader [discontinued](../01-display-and-optics/briefs/09-sol-reader-postmortem.md), the broader Day-1 empirical bundle is narrower: ~$50 Kobo Clara HD validates the entire compute + software stack. The display + optics empirical baseline shifts to (a) Pervasive Displays bench panels viewed through salvaged Quest 3 pancake (per Campaign 1's sourcing brief), (b) Kindle Paperwhite for general typography reference, and optionally (c) Even Realities G1 for shipped-glasses UX hands-on per [brief 4](briefs/04-mentraos-evaluation.md).

### 2. The custom-vs-MentraOS choice is forced, not a tradeoff

We hoped MentraOS-on-G1 might let us skip hardware. Empirically it doesn't:
- Z100 doesn't render bitmaps through MentraOS — typography out of our control.
- G1 renders bitmaps but at 640×200 = ~30 wpp — below Sol.
- Both architectures require phone tether, violating principle 1.

This is not a "depends on use case" tradeoff. It's a "MentraOS hardware doesn't currently meet the bar." **The decision is forced by the capability matrix.** Re-evaluate when Even Realities ships G2's higher-res panel + MentraOS exposes the bitmap API at G2 resolution + a standalone runtime path appears.

### 3. Typography quality is the binding constraint, not raw display PPI

Words-per-page > pixel-density-virtual. Sol's 256×256/eye magnified to "300 PPI virtual" still draws "Apple II" complaints because 250 wpp×2 (binocular) = ~125 wpp on screen, magnified to *feel* like ~125 wpp. Choice-specs at 2.9" 296×128 produces ~40 wpp before magnification, magnified to feel like ~40 wpp. **We need to either (a) accept the words-per-page penalty, (b) source a larger panel, or (c) co-engineer a higher-PPI panel.** All other typography work (font choice, leading, hyphenation) is rounding error against this metric.

### 4. Power architecture is dominated by sleep-state quality, not active-mode efficiency

Per [brief 1's power math](briefs/01-soc-selection.md): the difference between a 500 mWh / 16-session battery profile and a 500 mWh / 1-session profile is the SoC's deep-sleep behavior. **Pi Zero 2W is fast and cheap and has no real sleep state — making it disqualified despite excellent dev experience.** i.MX6 Solo with mature DSM is the "boring works" choice, exactly aligning with principle 5 (sleep-grade reliability).

### 5. The renderer can be commodity; the integration is bespoke

KOReader is "the renderer" for choice-specs. We don't write our own — we configure existing software. The choice-specs-specific work is:
- Frame design (Campaign 4)
- Power management tuning
- Input event routing (Campaign 3 contract)
- Light Lua skinning (head-locked single page, hide library nav, etc.)
- Boot tuning

Most of these are bounded weeks-of-work tasks, not months. **The project's center of gravity is hardware integration, not software development.** This is an unusual property for a personal project; lean into it.

### 6. The Caster question turned out to be smaller than expected

Campaign 1 inherited Modos Caster as the fast-refresh FPGA controller for E Ink panels. [Brief 1](briefs/01-soc-selection.md) and [brief 6](briefs/06-rendering-pipeline.md) both conclude: **i.MX6 Solo's native EPDC peripheral is sufficient for static text; Caster is a Phase 2 upgrade, not a Phase 1 requirement.** This simplifies prototype Phase 1 considerably.

## Open questions for downstream campaigns

### To Campaign 3 (Input & Interaction)

1. **Input event contract.** Events: `{type: "key" | "tilt", name: "next_page" | "prev_page" | "open_library" | ..., confidence: 0..1, ts: int}`. Delivered to KOReader's Lua event loop via Linux input device or unix-domain socket. Source-agnostic.
2. **Library navigation owner.** Campaign 3 owns the library screen (UI + input traversal). KOReader's `file_manager` Lua module is a starting point; replace or hide as Campaign 3 sees fit. Contract: Campaign 3 picks a book, calls `AppController.openBook(path)`.
3. **IMU sensor location and asymmetry.** Pancake optical engine is on dominant-eye side; weight is asymmetric. Campaign 3 should know that the IMU sees an asymmetric inertial reference frame; calibration must compensate.
4. **BLE ring pairing UX.** Campaign 3 owns. KOReader has built-in support for arrow-key BT HID; pairing is OS-level (`bluetoothctl`).

### To Campaign 4 (Industrial Design, Power, Safety)

1. **Compute board target weight: ~5–8 g for a custom i.MX6 Solo carrier with BLE/WiFi combo + USB-C + IMU + battery management.**
2. **Frame depth budget for compute: ~5 mm in temple plane.** Most of the depth budget is the optical engine (Campaign 1).
3. **Battery target: 500 mWh (140 mAh × 3.7 V) — supports ~24 sessions of 90 min.** This is sized against the brief 6 power model (~21 mWh / session) plus margin.
4. **Heat: passive cool sufficient.** SoC + display + radio total dissipates ≤200 mW in active reading; against the temple, passive aluminum spreader on the SoC is enough.
5. **Charge / data: USB-C, single port, in the temple.** Both charging and book-loading use the same connector. OTG mass-storage mode for USB drops in firmware update + book sync.
6. **No active cooling, no fan, no air gap. Passive monolith.**

### Unresolved at Campaign 2 (revisit later)

1. **Empirical 90-min KOReader power on Kobo Clara HD.** The 21 mWh/session estimate is back-of-envelope. **High-leverage measurement** — bench it Day 1.
2. **Custom i.MX6 SoM availability at hobbyist quantities.** Toradex Colibri / Variscite VAR-SOM-MX6 are the realistic candidates. Verify availability + EPDC variant before committing the custom-carrier path.
3. **MentraOS revisit gate.** When Even Realities G2 documentation is published with concrete resolution + bitmap API + IMU API specs, re-read the capability matrix. Set a reminder.
4. **crengine on tiny resolutions (200×200, 296×128).** Defaults assume 1024×768+ panels; bench-test layout quality on our small panels before committing typography defaults.
5. **MuPDF AGPL trajectory.** Personal prototype: fine. Product: pay Artifex's commercial license or replace MuPDF with PDF-removed crengine-only build. Punt to product phase.

## Brief index

- [01 — SoC Selection Tradeoff Matrix](briefs/01-soc-selection.md) — i.MX6 Solo recommended primary; ESP32-S3 fallback. RK3566 / Pi Zero 2W / Apollo4 / AR1 ruled out.
- [02 — EPUB Renderer Comparison](briefs/02-epub-renderer.md) — KOReader recommended primary; Plato fallback; custom Rust/C rejected on Linux.
- [03 — OS Choice](briefs/03-os-choice.md) — Buildroot Linux on i.MX6 Solo; FreeRTOS on ESP32-S3 fallback. Yocto / Debian / Zephyr / bare-metal / microkernel rejected.
- [04 — MentraOS Deep Evaluation](briefs/04-mentraos-evaluation.md) — Rejected as primary build target. Display capability below Sol baseline; phone-tether violates principle 1; IMU latency uncertain. Useful only as $599 UX-validation toy.
- [05 — Typography at Low Resolution / Monochrome](briefs/05-typography-low-res.md) — Min panel: 2.9" 296×128. Literata body font; left-aligned default; full flash every 6th turn.
- [06 — Page Rendering Pipeline Architecture](briefs/06-rendering-pipeline.md) — 8-module pipeline, 4-page bitmap cache, ~21 mWh/session, ~600 ms wake-to-page.
