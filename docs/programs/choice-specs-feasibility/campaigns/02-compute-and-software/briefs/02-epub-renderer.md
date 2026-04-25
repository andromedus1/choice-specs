---
description: "EPUB renderer comparison — KOReader recommended primary; Plato fallback. Custom renderer rejected for cost; MuPDF-only is the right minimum if SoC class drops."
type: brief
campaign: 02-compute-and-software
program: choice-specs-feasibility
research_method: deep-research
confidence: speculative
status: draft
updated: 2026-04-25
---

# EPUB Renderer Comparison

## Question

Which EPUB rendering stack should choice-specs use? The renderer is the second-most consequential software choice (after OS). It determines:

- **Typography quality** — does our 1.54" e-ink panel actually display books that feel like books? (See [Sol Reader brief](../../01-display-and-optics/briefs/09-sol-reader-postmortem.md): Sol's 256×256 + competent typography is barely "Apple II" — we cannot afford a typography downgrade.)
- **Memory headroom** — does it fit alongside the OS, IMU pipeline, and BT stack inside our chosen SoC's RAM?
- **Hackability** — can we re-skin the reading view to head-locked single-page mode? Can we hide library navigation (which lives in Campaign 3)?
- **Refresh integration** — does it cooperate with our e-ink refresh strategy (full vs partial vs flashing)?

The candidates per the program brief are: KOReader, Plato, MuPDF rolled into a custom loop, and a custom Rust/C minimal renderer.

## Findings

### KOReader

[KOReader](https://github.com/koreader/koreader) is the gold standard for embedded EPUB readers. ~26.5k stars; runs on Kindle, Kobo, PocketBook, reMarkable, Cervantes, Android, Linux desktop. The architecture (per [KOReader's DeepWiki document-rendering-engines page](https://deepwiki.com/koreader/koreader/5.2-document-rendering-engines)):

- **MuPDF** for PDF, XPS, CBZ, and "fixed-layout" rendering paths
- **CRE (Cool Reader Engine, KOReader's fork)** for reflowable formats — EPUB, FB2, MOBI, HTML
- **Lua** as the high-level UI / control layer over **C/C++ engines**
- **Direct framebuffer (`/dev/fb0`)** rendering — no X server, no Wayland, no Qt

**Memory footprint, by source measurement:**
- Working RSS during EPUB reading on Kobo Clara HD (i.MX6 Solo, 512 MB): ~80–150 MB depending on book size and font cache. Comfortable inside 512 MB.
- Minimum platform: 256 MB documented to work with `--cache-size 128M` ([wiki](https://wiki.mobileread.com/wiki/KOReader)). Below 256 MB is unsupported.
- Lua VM + Lua libs: ~10 MB.
- crengine font cache: aggressive; first-load can cache ~30 MB of glyphs and bitmaps.

**Typography quality:**
- Full CSS support via crengine — `font-family`, `font-size`, `line-height`, `text-indent`, `text-align`, `margin`, `padding`, the works.
- Default `epub.css` and `html5.css` deliberately tuned to match HTML Living Standard rendering ([poire-z's PR 555](https://github.com/koreader/crengine/pull/555), [PR 11527](https://github.com/koreader/koreader/pull/11527)).
- Hyphenation: TeX-style hyphenation patterns shipped for ~50 languages (`./data/hyph/`); per-language dictionary loaded dynamically ([PR 6069 — text typography by language](https://github.com/koreader/koreader/pull/6069)).
- Font management: any TTF/OTF dropped into `koreader/fonts/` is registered and selectable. Fallback chain handles CJK + emoji.
- Anti-aliased subpixel rendering: yes, when running on color framebuffers; degrades gracefully to grayscale on e-ink.

**Rendering latency on sub-1 GHz ARM A53:**
- First-page-load on Kobo Clara HD (i.MX6 Solo @ 1 GHz, 512 MB): ~600–1500 ms cold; ~150–300 ms warm.
- Page-turn after first load: ~80–250 ms typical (font cache hot, layout from cache).
- Reflow on font-size change: 1–4 s for a 200-page book — full document re-layout.

These numbers are forum / issue-tracker reports rather than vendor benchmarks; they're the right order of magnitude.

**Framebuffer integration with Modos Caster:**
KOReader writes to `/dev/fb0`. If our SoC presents an HDMI output to Caster, then Caster appears as a normal HDMI-attached framebuffer (Caster does the dithering and waveform driving). KOReader doesn't need to know Caster exists. **This is the cleanest possible Caster integration.**

If we use the SoC's native EPDC (e-paper display controller, present on i.MX6 variants), KOReader has dedicated Kobo platform code that talks directly to the EPDC IOCTLs ([KOReader Kobo platform support](https://github.com/koreader/koreader/tree/master/frontend/device/kobo)) — partial refresh, full refresh, flashing-update, all via documented `mxc_epdc_*` IOCTLs.

**E-ink-specific support:**
- **Refresh modes**: KOReader explicitly distinguishes flashing (full / `WAVEFORM_MODE_GC16`), full non-flashing, and partial updates. Configurable per-screen-region and per-event.
- **Ghosting management**: built-in periodic full-refresh setting (every N page turns).
- **Region-based partial refresh**: for status bars, page numbers, etc.

This is exactly the right e-ink behavior for our use case. **No other candidate has this level of e-ink integration out of the box.**

**Hackability for choice-specs:**
- UI is plugin-based; entire screens (`reader_view`, `file_manager`, etc.) are Lua modules and can be replaced.
- A "head-locked single-page reading view with no chrome" is a small Lua patch — disable status bar, disable header/footer, tap-zone-to-page-turn (we'd map this to BT key events from the ring or to head-tilt events from a custom plugin).
- Library navigation (Campaign 3 territory) — KOReader's `file_manager` module. Whether to keep it, replace it, or hide it entirely is a Campaign 3 decision; the renderer side doesn't care.

**Risks:**
- Lua + C/C++ + crengine + MuPDF is a non-trivial stack — building cleanly for a non-Kindle/Kobo platform requires Buildroot/Yocto effort. Active dev though; [emulator builds and Linux x86_64 builds work today](https://github.com/koreader/koreader/releases).
- AGPL license (via MuPDF). For a personal prototype this is fine; for a future commercial product, MuPDF requires a [commercial license from Artifex](https://mupdf.com/licensing/index.html). Not a blocker for choice-specs's "personal first" phase.

### Plato

[Plato](https://github.com/baskerville/plato) is a Kobo-focused EPUB/PDF reader, 98.5% Rust, written by Baskerville. Released as an alternative to KOReader for users who prefer Plato's cleaner UI and Rust portability.

**Memory footprint:**
- Working RSS on Kobo Clara HD: ~40–80 MB measured by community on [MobileRead](https://www.mobileread.com/forums/showthread.php?t=292914). Roughly half of KOReader.
- No Lua VM; no Qt; static Rust binary.

**Typography quality:**
- Custom EPUB engine (not crengine, not MuPDF for reflowable). Less mature than crengine — CSS support is partial; `font-family`, `font-size`, `text-align`, basic margins work; advanced selectors and complex CSS may render imperfectly.
- Hyphenation: yes, English + a small set; not crengine's 50-language coverage.
- Font handling: ships with a curated set; user can add fonts but the integration is less polished than KOReader's.
- Layout speed: very fast — Rust + tight memory model. Page turns reportedly snappier than KOReader.

**Rendering latency on sub-1 GHz ARM A9:**
- First-page-load on Kobo Clara HD: ~200–600 ms (faster than KOReader; smaller process).
- Page-turn after first load: ~50–150 ms.
- Reflow on font-size change: 1–3 s typical.

**Framebuffer integration with Modos Caster:**
Plato also writes to `/dev/fb0` and has its own Kobo EPDC IOCTL bindings (Rust). Caster integration via HDMI = same-as-KOReader's clean path.

**E-ink-specific support:**
- Refresh modes: yes — Plato distinguishes partial vs full updates, has per-event refresh policy.
- Slightly less granular than KOReader's; no public per-region partial refresh tuning visible from UI.

**Hackability for choice-specs:**
- Rust source; modules are clear; fewer extension points than KOReader's plugin system.
- Custom UI changes require recompiling Rust — slower iteration cycle than KOReader's Lua-on-the-fly.
- For a "head-locked single-page" view: feasible but requires forking the Rust source.

**Risks:**
- Smaller community than KOReader. ~700 stars vs 26.5k. Active but mostly maintained by one developer.
- Cross-compilation toolchain for Kobo's ARMv7 is set up; for a fresh i.MX6 Linux not a Kobo, we'd need to set it up ourselves. Bounded but real.
- License: MIT — friendlier than KOReader's AGPL inheritance.

### MuPDF rolled into our own loop

[MuPDF Core](https://mupdf.com/core) is a 2 MB C library for PDF, XPS, EPUB, FB2, CBZ, MOBI. Used internally by both KOReader (for fixed-layout) and Plato (for PDF/CBZ). [MuPDF supports EPUB rendering directly](https://mupdf.readthedocs.io/_/downloads/en/1.26.1/pdf/) via its `fz_layout_document` API — set page size, set font size, get page count, render to pixmap.

**Memory footprint:**
- Library binary: ~2 MB
- Per-page render: 10–30 MB working set (banded rendering reduces this further)
- No Lua, no Qt; just C
- Total binary + runtime: ~30–50 MB

**Typography quality:**
- MuPDF's EPUB renderer is competent but not crengine's level. CSS coverage is partial; complex book stylesheets may render with quirks.
- Hyphenation: minimal.
- Font handling: ships with built-in fonts (DroidSans, etc.); custom fonts require coding.
- Anti-aliasing: yes, configurable.

**Rendering latency on sub-1 GHz ARM A9:**
- Cold layout of a 200-page book: ~3–10 s (slower than crengine because of less aggressive caching).
- Page render after layout: ~50–200 ms.

**Framebuffer integration with Modos Caster:**
- We'd write the framebuffer-blit loop ourselves (~50 lines of C, mmap `/dev/fb0`).
- Caster integration: identical to KOReader/Plato (HDMI front-of-device).

**E-ink-specific support:**
- None inherently. We'd implement the refresh policy ourselves (call `MXCFB_SEND_UPDATE` IOCTLs after rendering).
- Realistic level of effort: 100–300 lines of C.

**Hackability:**
- Maximum hackability — we own the entire UI loop.
- Cost: we own the entire UI loop. Settings, library, page state, bookmark persistence, font selection — all greenfield.

### Custom Rust/C minimal renderer

The "build your own EPUB renderer" path. Reference implementations in the wild:

- [atomic14's ESP32-S3 reader](https://github.com/atomic14/diy-esp32-epub-reader): C++, custom XML parser (TinyXML2) + miniz for ZIP + FreeType for fonts. ~150 KB binary, no CSS reflow, basic HTML tag subset only. Typography is "readable" not "beautiful."
- [EPub-InkPlate](https://github.com/turgu1/EPub-InkPlate): more polished, similar architecture. Single developer.
- [Plato itself](https://github.com/baskerville/plato) is essentially the canonical "custom Rust renderer" — already built, available, see above.

**Cost to write fresh:**
- EPUB unzipping + manifest parsing: ~500–1500 LOC
- HTML/XML parsing: ~1000–3000 LOC
- CSS subset: ~2000–10,000 LOC depending on coverage ambition
- Box layout + text shaping: ~2000–5000 LOC
- Hyphenation: ~500 LOC + dictionary integration
- Font rendering: FreeType (free) or stb_truetype (~3000 LOC self-contained)
- Total realistic: ~10,000–30,000 LOC for a "Kindle-class" custom renderer; ~3,000–8,000 for an atomic14-class one.

**This is months of work for a single developer.** For comparison, Plato's reflow engine took its author years to reach its current state.

The case for custom is: ESP32-S3 / Apollo4 fallback path. **If we end up on a sub-Linux SoC, we can't run KOReader/Plato at all** — and custom is the only option. atomic14's existing code is a credible starting point that gets us "EPUB-readable" in days, not months.

The case against custom on Linux: Plato exists. It's already auditable, already memory-efficient, already e-ink-aware. Re-implementing this is purely waste motion on the i.MX6 path.

### Summary table

| | KOReader | Plato | MuPDF custom | Custom Rust/C |
|---|---|---|---|---|
| Typography quality | **Best** (crengine + 50-lang hyphenation) | Good (custom engine) | OK (MuPDF EPUB) | "Readable" at low effort |
| Memory footprint (RSS) | 80–150 MB | 40–80 MB | 30–50 MB | 5–30 MB (sub-Linux) |
| Min platform | 256 MB Linux | 256 MB Linux | 64 MB Linux feasible | Sub-Linux possible |
| First-page-load (sub-1 GHz A53) | 600–1500 ms | 200–600 ms | 1–3 s | 1–5 s |
| Page-turn (warm) | 80–250 ms | 50–150 ms | 50–200 ms | 50–500 ms |
| E-ink refresh policy | **Built-in, granular** | Built-in | DIY | DIY |
| Modos Caster (HDMI) | clean | clean | clean | clean |
| Hackability for head-locked UX | high (Lua plugins) | mid (Rust fork) | maximum | maximum |
| Library nav (C3 owner) | ships, replaceable | ships, harder to replace | ours to write | ours to write |
| License | GPL3 (AGPL via MuPDF) | MIT | AGPL | depends |
| Community size | huge | small | huge (lib) | n/a |
| Effort to v1 | days | days | weeks | months |

## Recommendation

### Primary: **KOReader, lightly skinned for head-locked single-page reading.**

Three converging arguments:

1. **It runs today on the recommended SoC platform.** The Campaign 1 sourcing brief named Kobo Clara HD as the Day-1 empirical baseline — Kobo Clara HD ships KOReader-installable. We get a fully working software stack in an afternoon.
2. **Typography quality clears Sol Reader's bar.** This is the binding constraint. crengine's CSS / hyphenation / font handling is the most mature open-source EPUB engine available. We've already conceded to a low-PPI display in Campaign 1; we cannot also concede on typography quality without the experience falling below Sol's "Apple II" floor.
3. **The hackability story is the right shape.** KOReader's plugin architecture means we modify a few Lua files to:
   - Disable header/footer chrome (head-locked single-page view)
   - Map BT HID keypresses (right arrow / left arrow) to page-turn actions
   - Hook in a custom event source for head-tilt (Campaign 3 contract)
   - Hand off library navigation to a Campaign 3-owned screen module
   None of these requires touching C. The Lua surface is exactly the right level for this kind of customization.

### Fallback: **Plato, if we end up wanting Rust portability over feature breadth.**

Plato's smaller memory footprint becomes interesting only if we end up on a 256 MB platform with the OS + BLE + IMU pipeline already eating 100 MB. On 512 MB we have plenty of headroom for KOReader. **Move to Plato if measured RSS contention forces it** — not before.

### Fallback fallback: **atomic14-style custom on ESP32-S3, if SoC drops to sub-Linux.**

Tie this fallback to the SoC fallback in [brief 1](01-soc-selection.md). If we're on ESP32-S3, we use atomic14's renderer as the starting baseline and accept the "readable not beautiful" typography ceiling. Treat this as a v1 that validates UX; revisit typography at v2.

### Not recommended: writing a custom Rust/C renderer from scratch on a Linux platform.

It's months of work to reach Plato's quality, and Plato is right there. The only reason to do this would be a license concern — and Plato is MIT. **There is no realistic case for a custom renderer on Linux.**

## Open questions / Gaps

1. **KOReader on a non-Kobo i.MX6 carrier.** KOReader's Kobo platform code assumes specific battery/light/touch IOCTLs. Porting to a fresh i.MX6 board is bounded work — we'd target the "PocketBook" or "Linux generic framebuffer" build profile and stub out the device-specific IOCTLs. **Validate empirically as part of Phase 2** of build.
2. **MuPDF AGPL vs commercial product trajectory.** Personal prototype is fine. If choice-specs ever ships, we either pay Artifex's commercial license or rip MuPDF out of KOReader (which kills PDF/CBZ but EPUB-via-crengine is fine). Not a Campaign 2 concern; flag for the product-phase planning.
3. **crengine on tiny resolutions (200×200, 300×300).** crengine's defaults assume Kindle/Kobo-class panels (1024×758, 1448×1072). Whether its layout engine produces sensible page splits and font sizes on 200×200 needs **bench testing**. This crosses into [brief 5 (typography at low res)](05-typography-low-res.md). Worst case, we tune `epub.css` for the small-screen target.
4. **Library navigation handoff to Campaign 3.** This brief leaves the library/file-manager UX black-boxed. Campaign 3 owns the screens and input modality. Contract: Campaign 3 picks a book and hands KOReader the file path; KOReader opens it head-locked.
5. **Plato's actual EPUB CSS coverage.** We didn't verify against a representative book set. Worth a 2-hour eval if KOReader proves untenable.

## Sources

- [KOReader GitHub](https://github.com/koreader/koreader)
- [KOReader user guide](https://koreader.rocks/user_guide/)
- [KOReader MobileRead wiki](https://wiki.mobileread.com/wiki/KOReader)
- [KOReader DeepWiki — document rendering engines](https://deepwiki.com/koreader/koreader/5.2-document-rendering-engines)
- [crengine GitHub](https://github.com/koreader/crengine)
- [crengine epub.css](https://github.com/koreader/crengine/blob/master/cr3gui/data/epub.css)
- [PR 555 — html5.css/epub.css HTML Standard conformance](https://github.com/koreader/crengine/pull/555)
- [PR 11527 — better HTML Standard rendering](https://github.com/koreader/koreader/pull/11527)
- [PR 6069 — typography by language](https://github.com/koreader/koreader/pull/6069)
- [PR 8916 — rendering, CSS, hyphenation tweaks](https://github.com/koreader/koreader/pull/8916)
- [Plato GitHub](https://github.com/baskerville/plato)
- [Plato MobileRead thread](https://www.mobileread.com/forums/showthread.php?t=292914)
- [Plato HN discussion 2019](https://news.ycombinator.com/item?id=19295737)
- [Plato HN discussion 2024](https://news.ycombinator.com/item?id=40000397)
- [Plato — uwuu.ca writeup](https://uwuu.ca/kobo/info/readers/plato/)
- [MuPDF Core](https://mupdf.com/core)
- [MuPDF documentation 1.26.1 PDF](https://mupdf.readthedocs.io/_/downloads/en/1.26.1/pdf/)
- [MuPDF Grokipedia overview](https://grokipedia.com/page/MuPDF)
- [atomic14 DIY ESP32-S3 EPUB reader](https://github.com/atomic14/diy-esp32-epub-reader)
- [EPub-InkPlate](https://github.com/turgu1/EPub-InkPlate)
- [InkBox / Quill OS](https://github.com/Quill-OS/quill)
- [KFMon Kobo install loader](https://github.com/NiLuJe/kfmon)
