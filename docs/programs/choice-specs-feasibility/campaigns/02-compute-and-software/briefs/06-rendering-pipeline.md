---
description: "Page rendering pipeline architecture. Modules: BookLoader, ReflowEngine, Pager, Rasterizer, FrameBufferDriver, EinkRefreshPolicy, InputContract. Cache 3 pages forward + current. Full flash every 6th turn; partial otherwise. ~30 mWh per 90-min session."
type: brief
campaign: 02-compute-and-software
program: choice-specs-feasibility
research_method: deep-research
confidence: speculative
status: draft
updated: 2026-04-25
---

# Page Rendering Pipeline Architecture

## Question

Given the chosen [SoC (i.MX6 Solo)](01-soc-selection.md), [OS (Buildroot Linux)](03-os-choice.md), and [renderer (KOReader / crengine)](02-epub-renderer.md), what does the actual code path look like from "user opens a book" to "first page on the display"?

This brief decomposes the pipeline into modules and specifies the contracts between them, including:

- Book load + parse
- Layout / reflow
- Raster to framebuffer
- E-ink display update (refresh mode, ghosting management)
- Prefetch next page
- Deep-sleep until next page-turn input
- Memory budget per stage
- Cache strategy
- Where the IMU + BT input integrate (treat as black-box from C3's perspective)

The goal: a sketch the [Campaign 4 (industrial design / power)](../../program.md) work can size memory and battery against, and a contract Campaign 3 can integrate input against.

## Findings

### Module decomposition

```
┌─────────────────────────────────────────────────┐
│             InputAdapter (C3 black box)         │ ← BT HID + IMU events
└────────────────────┬────────────────────────────┘
                     │ events: NextPage, PrevPage, OpenLibrary, ...
                     ▼
┌─────────────────────────────────────────────────┐
│             AppController (Lua)                 │
│  state machine: library | reading | settings    │
└────────────────────┬────────────────────────────┘
                     │
        ┌────────────┴───────────┐
        ▼                        ▼
┌──────────────┐         ┌─────────────────┐
│ LibraryView  │         │  ReadingView    │
│  (C3 owner)  │         │ (this brief)    │
└──────────────┘         └────────┬────────┘
                                  │ open(book_path)
                                  ▼
┌─────────────────────────────────────────────────┐
│             BookLoader                          │  EPUB unzip + manifest
│  - reads container.xml                          │  parse + spine
│  - extracts content.opf                         │
│  - returns DocumentSession                      │
└────────────────────┬────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────┐
│             ReflowEngine (crengine)             │  HTML/CSS → box tree
│  - parses XHTML chapters                        │  → layout
│  - applies stylesheet (epub.css + book CSS)     │
│  - shapes text, breaks lines                    │
│  - hyphenates if enabled                        │
│  - produces flat page index (page → byte range) │
└────────────────────┬────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────┐
│             Pager                               │  manages current page
│  - maps page_id → DOM range                     │  cursor, prefetch, cache
│  - prefetch policy                              │
│  - cache eviction                               │
└────────────────────┬────────────────────────────┘
                     │ render_page(page_id)
                     ▼
┌─────────────────────────────────────────────────┐
│             Rasterizer                          │  glyphs → pixels
│  - FreeType glyph rendering (cached)            │
│  - blits glyphs to bitmap buffer                │
│  - applies grayscale dither                     │
└────────────────────┬────────────────────────────┘
                     │ bitmap (296×128 8bpp gray)
                     ▼
┌─────────────────────────────────────────────────┐
│             FrameBufferDriver                   │  writes /dev/fb0 OR
│  - Caster HDMI path (if attached)               │  Caster SPI path
│  - or direct EPDC IOCTL on i.MX6                │  or i.MX6 EPDC
└────────────────────┬────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────┐
│             EinkRefreshPolicy                   │  partial vs full vs flash
│  - tracks ghost-load                            │
│  - selects waveform per update                  │
│  - issues MXCFB_SEND_UPDATE                     │
└────────────────────┬────────────────────────────┘
                     ▼
                E-ink panel
```

### Boot-to-first-page sequence

Cold boot, user pushes power button:

1. **t=0 ms** — U-Boot loads kernel + initramfs from flash. ~500 ms.
2. **t=500 ms** — Kernel inits, mounts root, starts `/init`. ~1500 ms.
3. **t=2000 ms** — `/init` execs `koreader.lua`. ~200 ms Lua VM start.
4. **t=2200 ms** — KOReader reads last-session state from `/var/koreader/last.json`: `{book_path, page_id}`.
5. **t=2400 ms** — BookLoader reads EPUB → DocumentSession (~200 ms for 1 MB EPUB on SD-card-class flash).
6. **t=2600 ms** — ReflowEngine parses + lays out current chapter (~500 ms cold).
7. **t=3100 ms** — Rasterizer renders current page bitmap (~100 ms).
8. **t=3200 ms** — FrameBufferDriver pushes to e-ink with full-flash refresh (~750 ms with Caster, ~500 ms with EPDC native).
9. **t=3700–3950 ms** — User sees their last-read page.

**Cold boot total: ~3.7–4.0 s.** Inside our 5 s sleep-grade reliability target.

Wake-from-suspend sequence (the common case):

1. **t=0 ms** — IRQ from BT controller (ring keypress) or IMU (head-tilt classified as page event).
2. **t=50 ms** — Linux kernel resumes from `mem` suspend.
3. **t=200 ms** — KOReader event loop wakes; processes input event.
4. **t=300 ms** — Pager queries cache for next-page bitmap (cached if prefetched).
5. **t=350 ms** — FrameBufferDriver pushes to e-ink (partial refresh, ~250 ms).
6. **t=600 ms** — User sees new page.

**Wake-to-page total: ~600 ms.** Slightly above the 500 ms aspirational target; acceptable for prototype. Tunable by reducing kernel resume time (preempting non-essential drivers from suspend) and ensuring the next page is always pre-rasterized.

### Memory budget per stage

For i.MX6 Solo + 512 MB LPDDR3 (Kobo Clara HD baseline):

| Stage | Working set | Peak | Notes |
|---|---|---|---|
| Kernel + drivers | 30 MB | 35 MB | Buildroot-class minimal Linux |
| systemd / init / shell | 5 MB | 5 MB | replaced by simple init script |
| KOReader Lua VM + libs | 15 MB | 20 MB | Lua interpreter + crengine + MuPDF |
| crengine document state | 30 MB | 60 MB | text DOM + style + layout for one book |
| Glyph cache (FreeType) | 10 MB | 30 MB | cumulative; bounded |
| Rasterized page cache | 5 MB | 25 MB | 4 pages × ~300 KB ea (296×128 8bpp + chrome) |
| Framebuffer | 1 MB | 1 MB | 296×128 ×8bpp = 38 KB; Linux fb is bigger |
| BLE stack | 2 MB | 4 MB | BlueZ |
| IMU sample buffer | 1 MB | 1 MB | ring buffer for raw + classified events |
| Slack / scratch | 50 MB | — | OS page cache + transient allocations |
| **Total** | **~150 MB** | **~180 MB** | leaves ~330 MB free |

**Verdict:** comfortable on 512 MB. We could move to a 256 MB SoC if needed (KOReader's documented `--cache-size 128M` floor), but 512 MB gives headroom for stable prefetch caching and is the platform Kobo ships.

### Cache strategy

**Goal:** make page-turns feel instant. Page-turn UX latency = `render_time + display_update_time`. If `render_time = 0` (page bitmap is in cache), latency drops to display update only (~250 ms partial refresh).

**Two-level cache:**

1. **Layout cache** — `crengine` already does this internally. Once a chapter is laid out, the box tree + page boundaries are kept in memory until eviction.
2. **Rasterized page cache** — keep the bitmap of `[current-1, current, current+1, current+2]` (4 pages: previous, current, +2 prefetched ahead). At 296×128 ×8bpp = 38 KB per page; 4 pages = 152 KB. Negligible memory. **Always prefetch +2 ahead in the background.**

**Eviction triggers:**
- Font-size change → invalidate all caches; full re-layout.
- Book switch → invalidate all caches.
- Memory pressure (rare) → drop oldest cached page.

**Background prefetch:**
- After each page-turn, kick a low-priority Lua coroutine that calls Rasterizer for `current+2`.
- If the user reads at typical pace (~30 s/page), the coroutine completes before the next turn.
- If the user pages quickly, the coroutine is preempted; partial-rasterized page is discarded.

### E-ink refresh policy

Drawing on [brief 5 (typography)](05-typography-low-res.md) and [E-ink update modes references](https://hackaday.io/project/21551-paperino/log/59392-e-paper-basics-1-update-modes):

| Event | Update mode | Latency |
|---|---|---|
| Book open | Full flashing (`GC16`) | ~750 ms |
| Page turn within session | Partial (`GLR16` / `GL16`) | ~250–400 ms |
| Page turn — every 6th | Full flashing | ~750 ms |
| Font-size change | Full flashing | ~750 ms |
| Library nav (C3 territory) | Partial or A2 | ~125–250 ms |
| Settings UI | Partial | ~250 ms |
| Sleep entry | None (display latched) | 0 |

**The "every 6th flashing" pattern** is canonical e-reader behavior. Reading-mode users barely notice; ghosting is held below perceptual threshold. KOReader's setting key is `EpubFlashingPages`; default 1 (every page) is too aggressive; tune to 6.

**Ghost-load tracking:**
- Each partial update accumulates a "ghost load" estimate.
- After threshold (~6 partial updates), promote next update to full flash regardless of event type.
- Reset counter on full flash.

This is implemented in KOReader's existing Kobo platform code; we'd just reuse / lightly tune.

### Deep-sleep entry / exit

Sleep entry (idle timer fires after 30 s of no input):

1. Pager flushes any in-flight prefetch.
2. AppController persists `{book_path, page_id, scroll_offset}` to `/var/koreader/last.json` (atomic rename).
3. Userspace calls `echo mem > /sys/power/state` (or KOReader's wrapper).
4. Kernel walks driver suspend hooks; BT controller stays in low-power advertising; IMU stays in interrupt-only mode.
5. SoC enters DSM (~10 mW i.MX6 Solo).

Sleep exit (wake event):

1. BT IRQ or IMU IRQ fires.
2. Kernel resumes; ~50 ms.
3. KOReader receives the input event.
4. Renders next page (or library command), partial refresh.
5. Resets idle timer.

**Failure mode:** kernel hangs on resume. Mitigation: hardware watchdog (i.MX6 Solo has integrated WDOG); userspace pets it every 10 s during normal operation; if not petted within 30 s post-resume, hard reset.

### Power profile per page-turn

Approximate energy per page-turn event:

- Wake from sleep: 50 ms × 250 mW = 12.5 mJ
- KOReader resume + render: 200 ms × 500 mW = 100 mJ (cached page; rasterize is just blit)
- E-ink update (partial): 250 ms × 50 mW (panel) = 12.5 mJ
- Total per page-turn: **~125 mJ ≈ 0.035 mWh**

With 5 s wake → render → sleep cycle, average power between turns ≈ 25 mW; deep-sleep is 10 mW. **Total 90-min session energy:**

- 90 min × 60 = 5400 s
- Active page-turn events: 180 (one every 30 s on average)
- Active phase: 180 × 0.035 = 6.3 mWh
- Sleep phase: 5400 s - (180 × 0.5) = 5310 s × 10 mW = ~14.75 mWh
- **Total: ~21 mWh per 90-min session.**

This is below the 30 mWh estimate from [brief 1](01-soc-selection.md) (which assumed worst-case render time per page). With prefetch, most page-turns are bitmap-cached, so render time approaches the IO + display-update floor.

A 500 mWh battery (140 mAh × 3.7 V) supports **~24 sessions** between charges. Comfortable.

### IMU + BT input contract (Campaign 3 boundary)

Campaign 3 owns input. Campaign 2 specifies the contract:

```
InputAdapter publishes events to AppController via a Linux unix-domain socket
or KOReader's Lua event loop:

  event = {
    type: "key" | "tilt" | "library_nav",
    name: "next_page" | "prev_page" | "open_library" | "settings" | ...,
    confidence: 0.0..1.0,  // for IMU events
    timestamp_ms: int,
  }
```

- **BT HID device** (Gevo ring, generic clicker) pairs at OS level; KOReader receives keypresses through standard Linux input (`/dev/input/event*`). Already supported.
- **IMU classifier** runs as a separate user-space process that reads from a SPI-attached IMU (e.g., BNO055), runs gesture classification, and emits `tilt` events. Campaign 3 owns the classifier.
- **Library nav** events (open library, select book) are routed by AppController to the LibraryView module, which is a Campaign 3 owned screen.

This contract means Campaign 2's pipeline doesn't care whether input comes from a ring, a head-tilt, or future eye-blink dwell click. It just consumes events.

### Modos Caster integration: optional

Per [brief 1](01-soc-selection.md), Caster is **not required** if we use the Kobo's i.MX6 EPDC directly. Caster's value (75 Hz refresh) is unnecessary for static text. The pipeline above works equally well:

- **Path A — i.MX6 EPDC native:** Rasterizer → FrameBufferDriver writes to EPDC framebuffer → MXCFB_SEND_UPDATE IOCTL → panel. Lowest latency; battle-tested by Kobo.
- **Path B — Caster via HDMI:** Rasterizer → FrameBufferDriver writes to standard Linux fb0 → HDMI out → Caster does dithering + waveform → panel. Works if our SoC variant lacks EPDC (e.g., a non-Kobo i.MX6 carrier).

**For prototype phase, prefer Path A.** Defer Caster to a later iteration if needed.

### What stays in scope for this brief vs other campaigns

| Concern | Owner |
|---|---|
| BookLoader, ReflowEngine, Pager, Rasterizer | This brief / KOReader |
| FrameBufferDriver, EinkRefreshPolicy | This brief |
| Memory + cache budget | This brief |
| Power budget per stage | This brief; verified against C1 totals |
| InputAdapter implementation | Campaign 3 |
| LibraryView UI | Campaign 3 |
| Frame depth, IMU placement, battery cell | Campaign 4 |
| Display panel selection | Campaign 1 (with C2 input from brief 5) |
| OS init, suspend tuning | Brief 3 |

## Recommendation

**Adopt the architecture sketch above as the Campaign 2 architecture for Campaign 4 + Campaign 3 integration.**

Specifically:

1. **Build Phase 1 against the Kobo Clara HD's native EPDC path.** Validate the entire reading-view pipeline on a battle-tested platform before introducing Caster.
2. **Use a 4-page rasterized bitmap cache** (current ±1, +2 prefetch). Bound at 200 KB total; negligible memory.
3. **Refresh policy: partial on every page turn; full flash every 6th.** Default to KOReader's `EpubFlashingPages = 6`. User-tunable.
4. **Suspend on 30-second idle.** Wake on BT or IMU IRQ. Hardware watchdog as failsafe.
5. **Input contract: events delivered to KOReader's Lua event loop** as `{type, name, confidence, ts}`. Source-agnostic.
6. **Persistence: `/var/koreader/last.json` atomic write on every page turn.** Atomic rename, 4 KB max. Drop on unclean shutdown is recoverable.
7. **Don't introduce Caster until Phase 2 unless the EPDC path proves blocked.** It's a future-iteration upgrade, not a Phase 1 requirement.

Memory budget: ~180 MB peak on a 512 MB platform; comfortable margin. Power budget: ~21 mWh per 90-min session; easily fits a 500 mWh battery for 20+ sessions between charges.

## Open questions / Gaps

1. **Empirical validation on Kobo Clara HD.** Run KOReader, instrument page-turn latency and per-stage power, confirm the back-of-envelope 21 mWh estimate. Can be done Day 1 of Phase 1.
2. **Custom carrier i.MX6 EPDC availability.** If we move from Kobo's PCB to a custom carrier, the i.MX6 Solo variant we choose must include EPDC. Some i.MX6 SoMs lack the EPDC peripheral; verify before committing to a SoM.
3. **Input event routing.** The Lua event-loop integration of a custom IMU classifier is straightforward but unverified. Might need to write a small Lua plugin that reads from a Unix socket and dispatches events.
4. **Prefetch coroutine interaction with Lua GC.** Lua's incremental GC is fine in our case (no real-time deadlines beyond input event handling), but if we see frame drops we may need to tune `collectgarbage("step", N)` calls.
5. **Battery low / safe shutdown.** Below ~10% battery, current page must persist before any further action. Brownout protection on i.MX6 + atomic rename is the standard approach. Verify in Phase 2.
6. **Font cache warmup.** First page load is slow (cold FreeType). For UX, may want to warm the cache during boot — render an off-screen page in background while the user is still seeing the boot logo. Phase 2 polish.
7. **Multi-book library prefetch.** Out of scope; Campaign 3 owns library UX. We just expose `open(book_path)` to LibraryView.

## Sources

- [KOReader DeepWiki — document rendering engines](https://deepwiki.com/koreader/koreader/5.2-document-rendering-engines)
- [KOReader DeepWiki — file management / document system](https://deepwiki.com/koreader/koreader/5-file-management)
- [KOReader user guide](https://koreader.rocks/user_guide/)
- [crengine source](https://github.com/koreader/crengine)
- [crengine epub.css](https://github.com/koreader/crengine/blob/master/cr3gui/data/epub.css)
- [MuPDF Core](https://mupdf.com/core)
- [MuPDF documentation 1.26](https://mupdf.readthedocs.io/_/downloads/en/1.26.1/pdf/)
- [E-paper update modes (Hackaday)](https://hackaday.io/project/21551-paperino/log/59392-e-paper-basics-1-update-modes)
- [E-ink ghosting / refresh modes (Viwoods)](https://viwoods.com/blogs/paper-tablet/viwoods-e-ink-refresh-modes-explained)
- [BOOX e-ink technology](https://help.boox.com/hc/en-us/articles/360027486972-Understanding-E-Ink-Technology-in-BOOX-Devices)
- [NXP i.MX6 Linux EPDC driver (mxc_epdc)](https://www.nxp.com/products/processors-and-microcontrollers/arm-processors/i-mx-6-processors)
- [KOReader Kobo platform code](https://github.com/koreader/koreader/tree/master/frontend/device/kobo)
- [Kobo battery drain in suspend (issue 7565)](https://github.com/koreader/koreader/issues/7565)
- [Modos Caster GitHub](https://github.com/Modos-Labs/Caster)
- [FreeType font rendering](https://www.freetype.org/)
