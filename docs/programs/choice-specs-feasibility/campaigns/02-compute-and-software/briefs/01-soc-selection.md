---
description: "SoC tradeoff matrix for choice-specs — RK3566 recommended primary; ESP32-S3 fallback. Power, RAM, dev experience, FPGA driver compatibility, and BT/WiFi integration ranked against the actual workload."
type: brief
campaign: 02-compute-and-software
program: choice-specs-feasibility
research_method: deep-research
confidence: speculative
status: draft
updated: 2026-04-25
---

# SoC Selection Tradeoff Matrix

## Question

What SoC should drive the choice-specs prototype? The chosen part has to do five things, none of them especially hard individually but all of them together inside a glasses-class power and weight envelope:

1. Drive a Pervasive Displays e-ink panel (200×200 → 300×300) through a Modos Caster FPGA at a 1–5 Hz page-turn refresh rate (with rare 30 Hz bursts for transitions).
2. Render reflowed EPUB pages — minimum tier of typography quality matters (Sol Reader's "Apple II" complaints are the ceiling we have to clear; see [Sol Reader brief](../../01-display-and-optics/briefs/09-sol-reader-postmortem.md)).
3. Sample IMU data for head-tilt input at 50–100 Hz and run a small classifier in real time.
4. Maintain a BLE link to a fallback ring/clicker.
5. Deep-sleep between page turns and wake on input within ~250 ms.

Within Campaign 1's inherited budget of ~300–500 mW active typical and ≤50g system weight, the SoC is the single decision that most constrains everything downstream — battery capacity, frame thermal design, weight distribution, and software stack are all keyed off this choice. This brief picks a primary and a documented fallback.

## Findings

### Candidate set

Per the program brief, the candidates are: RK3566, RK3568, ESP32-S3, NXP i.MX6 / i.MX6 Solo, Raspberry Pi Zero 2W, Allwinner V3s/T113-S3, Ambiq Apollo4, Qualcomm AR1 Gen 1.

### Tradeoff dimensions, with measurements

#### Active power (typical) and deep-sleep power

| SoC | Active typical | Deep sleep | Source |
|---|---|---|---|
| RK3566 (Cortex-A55 ×4 @ 1.8 GHz, Mali-G52) | ~500 mW–2.5 W (idle Linux ~500 mW; 2D UI ~1 W) | ~30 mW (5 V × 6 mA) | [Boardcon EM3566 / vendor figures](https://www.armdesigner.com/EM3566_SBC/), [CNX Software](https://www.cnx-software.com/2021/07/28/boardcon-em3566-linux-sbc-is-powered-by-a-rockchip-rk3566-system-on-module/) |
| i.MX6 Solo (Cortex-A9 @ 800 MHz–1 GHz) | ~300–500 mW (Kobo Clara HD measured) | ~5–15 mW DSM ([NXP DSM mode](https://www.nxp.com/products/processors-and-microcontrollers/arm-processors/i-mx-6-processors)) | [Kobo battery drain issues](https://github.com/koreader/koreader/issues/7565) |
| Pi Zero 2W (Cortex-A53 ×4 @ 1 GHz, BCM2710A) | ~600 mW idle (120 mA × 5 V); 2.5 W max | **No documented sleep state** — the BCM2710 has no real low-power mode | [CNX Software deep dive](https://www.cnx-software.com/2021/12/09/raspberry-pi-zero-2-w-power-consumption/), [Hackaday](https://hackaday.com/2021/11/01/the-pi-zero-2-w-is-the-most-efficient-pi/) |
| ESP32-S3 + 8 MB PSRAM | ~150–250 mW active rendering | ~10 µA (deep sleep RTC only); ~0.8 mA light sleep | [atomic14 DIY EPUB reader](https://github.com/atomic14/diy-esp32-epub-reader) |
| Allwinner T113-S3 (Cortex-A7 ×2 @ 1.2 GHz + RISC-V + DSP) | ~400–700 mW active Linux | ~5–10 mW with PMIC sleep | [CNX Software](https://www.cnx-software.com/2022/02/13/allwinner-t113-s3-dual-core-cortex-a7-soc-features-128mb-ddr3-targets-automotive-and-industrial-applications/), [Forlinx writeup](https://medium.com/@forlinx2021/allwinner-t113-som-t113i-parameters-performance-power-consumption-data-price-310a431f01e7) |
| Ambiq Apollo4 (Cortex-M4F @ 192 MHz, 2.75 MB SRAM) | ~5 µA/MHz ≈ 1 mW @ 192 MHz | <10 µA deep sleep | [Apollo4 datasheet](https://contentportal.ambiq.com/documents/20123/388400/Apollo4-SoC-Datasheet.pdf) |
| Qualcomm AR1 Gen 1 | Closed; targeted at <500 mW glasses | Not public | [Qualcomm AR1 page](https://www.qualcomm.com/xr-vr-ar/products/ar-series/snapdragon-ar1-gen-1-platform) |

The most striking number is the RK3566's **idle Linux floor of ~500 mW**. That's almost the entire Campaign 1 budget. By contrast, the i.MX6 Solo running KOReader inside Kobo's tuned Linux measures ~300–500 mW *active* (page-rendering moments) and drops cleanly to 5–15 mW DSM between pages. The RK3566's published e-paper-class measurements (1.2 mA static, <30 mA during a 15-second page turn at the 10.3" e-paper reference design) only happen with vendor BSP work — out-of-the-box Buildroot doesn't get there.

#### RAM minimum for KOReader / Plato / MuPDF

[KOReader's MobileRead wiki](https://wiki.mobileread.com/wiki/KOReader) and the [KOReader GitHub](https://github.com/koreader/koreader) document a working minimum of 256 MB, with explicit `restart limit` advice for ≤256 MB devices. Battle-tested baseline = **i.MX6 Solo + 512 MB LPDDR3** (Kobo Clara HD). [Plato](https://github.com/baskerville/plato) is leaner (Rust, no Lua VM, custom EPUB engine) — community reports of running on 256 MB Kobos with headroom. [MuPDF Core](https://mupdf.com/core) has a 2 MB binary footprint and 10–30 MB working set per rendered page using banded rendering; [the embedded build with band rendering](https://mupdf.readthedocs.io/_/downloads/en/1.26.1/pdf/) fits comfortably in 64 MB total RAM.

[atomic14's ESP32-S3 EPUB reader](https://github.com/atomic14/diy-esp32-epub-reader) demonstrates EPUB on **8 MB PSRAM with no full Linux at all** — but it's a primitive HTML-subset reader without CSS reflow; explicitly not Kindle-grade typography. The [EPub-InkPlate project](https://github.com/turgu1/EPub-InkPlate) is similar.

#### Modos Caster FPGA driver compatibility

Caster targets a [Xilinx Spartan-6 LX16 with on-board STM32H750](https://github.com/Modos-Labs/Caster) and exposes the host interface as either **HDMI** (for the 13.3" Paper Monitor) or **SPI/parallel** (for the dev kit driving raw e-paper panels). The Modos Paper Dev Kit announcement at [CNX Software](https://www.cnx-software.com/2025/08/06/fpga-modos-paper-dev-kit-supports-e-ink-displays-75-hz-refresh-rate/) confirms the host-side requirements:

- **For HDMI host path:** any SoC with HDMI out works. RK3566, RK3568, Pi Zero 2W, i.MX6 (with HDMI variant), AR1 — all qualify. Allwinner T113-S3 has RGB/MIPI-DSI but **no native HDMI** (would need a bridge IC). ESP32-S3 has neither HDMI nor MIPI-DSI; out for the HDMI path. Apollo4: out.
- **For SPI/parallel path on the dev kit:** any SoC with SPI works, including Apollo4 and ESP32-S3. This is the lower-effort integration but at lower bandwidth — **5 Hz partial refresh of a 300×300 panel is well within budget on SPI at 20 MHz**, so the parallel path isn't required for our use case.

The clean picture: **for any Linux-class SoC we choose, the Caster integration is "wire HDMI to its HDMI input" — solved.** For ESP32-S3, we'd be using Caster's SPI bridging; this works for static page display but Caster's >75 Hz video-class capability becomes inaccessible. We don't need the high-refresh path for static EPUB reading anyway, so this isn't a blocker for ESP32-S3 — it's just a feature mismatch.

**Open caveat:** Caster's reference firmware assumes a host providing dithered RGB or grayscale via HDMI/USB; it does not assume we want full control of the EPD waveform from the SoC. We'd be configuring Caster as a "dumb" controller with our SoC as the framebuffer source, exactly as Modos's Paper Monitor uses it.

#### BT/WiFi integration

| SoC | BT | WiFi | Path |
|---|---|---|---|
| RK3566 | None on SoC | None on SoC | External combo (e.g. AP6256) over SDIO |
| i.MX6 Solo | None | None | External combo over SDIO |
| Pi Zero 2W | BT 4.2 onboard | 2.4 GHz onboard | Fully integrated |
| ESP32-S3 | BT 5.0 LE onboard | 2.4 GHz onboard | Fully integrated |
| Allwinner T113-S3 | None on SoC standalone; *Nagami SoM bundles ESP32* | Via SoM/external | [NetCube Nagami SoM bundles ESP32 for wireless](https://www.cnx-software.com/2025/12/01/netcube-systems-nagami-allwinner-t113-s3-mini-pcie-som-features-esp32-wireless-soc-supports-mainline-linux/) |
| Apollo4 Blue | BLE 5.1 onboard | None | BLE only |
| AR1 Gen 1 | BT, WiFi 7 | Yes | Built-in |

For choice-specs the BT side carries more weight than WiFi: BT for the ring fallback is mandatory; WiFi is for once-per-week book sync. **ESP32-S3 and Pi Zero 2W are the only candidates with both onboard.** Other SoCs all need an external combo module — adds ~$3–5 BOM, ~50–80 mW active radio, and a couple of square cm of PCB. Not a blocker but a real cost.

#### Dev experience

- **i.MX6 Solo + Kobo:** unbeatable. Buy a $30 Kobo Clara HD, install [KFMon](https://github.com/NiLuJe/kfmon) → KOReader, you have a working EPUB reader on representative hardware in an afternoon. NXP's Yocto BSP is mature and well-documented. **The "fastest path to validating typography" lever from the Campaign 1 sourcing brief.**
- **RK3566:** Mature mainline kernel; Buildroot, Debian, Yocto, and Android all officially supported. [Radxa Zero 3W](https://www.cnx-software.com/2023/12/08/radxa-zero-3w-sbc-rockchip-rk3566-8gb-ram-wifi6-raspberry-pi-zero-2-w-form-factor/) is the closest thing to a "Pi Zero shape with RK3566". Vendor BSPs are good. Some pain points around the Mali-G52 GPU and HDMI hot-plug edge cases.
- **Pi Zero 2W:** Easiest Linux experience; massive community; Raspbian works out of the box. **But the BCM2710 has no documented sleep state per [the Raspberry Pi forums](https://forums.raspberrypi.com/viewtopic.php?t=392265).** This is fatal for our deep-sleep-between-pages architecture.
- **ESP32-S3:** Arduino IDE or ESP-IDF; massive community; PSRAM and FreeRTOS are well-documented. **No Linux**, so no KOReader/Plato — we'd be writing a custom EPUB stack on top of [atomic14's work](https://github.com/atomic14/diy-esp32-epub-reader). The dev experience is excellent for the platform; the typography ceiling is the issue, not the dev cost.
- **Allwinner T113-S3:** Mainline Linux since 6.4 ([linux-sunxi.org](https://linux-sunxi.org/T113-s3)) but smaller ecosystem than Rockchip. Tina Linux SDK works but is less polished. Not first-pick for a fast prototype.
- **Apollo4:** Cortex-M class; bare-metal or Zephyr. No Linux. SRAM is 2.75 MB max — not enough for a real EPUB engine; would need cleverer architecture (offline pre-render). Out for our use case.
- **AR1 Gen 1:** Closed Meta-gated SDK. Not a personal-prototype target. **Rule out per program brief.**

#### BOM cost (at single-unit hobbyist scale)

| SoC | Module/SBC | Cost | Notes |
|---|---|---|---|
| RK3566 | Radxa Zero 3W (8 GB) | ~$50 | Pi Zero shape, full Linux |
| i.MX6 Solo | Kobo Clara HD (used) | ~$30–50 | salvage path; whole device |
| Pi Zero 2W | Pi Zero 2W | $15 | with WiFi/BT |
| ESP32-S3 | ESP32-S3-WROOM-1 + 8 MB PSRAM | $5–10 | bare module |
| Allwinner T113-S3 | T113 SoM (e.g. Forlinx) | ~$20–30 | mini SoM |
| Apollo4 Blue Plus | Eval board | $50 | dev only; product BOM lower |

#### Weight (PCB area, rough)

PCB area dominates more than the chip's mass. Rule of thumb: ~0.6 g/cm² for a four-layer FR4 PCB, plus components.

| SoC | Min board area | Rough PCB weight |
|---|---|---|
| RK3566 SoM (Radxa Zero 3W) | 30 × 65 mm = 19.5 cm² | ~12 g |
| i.MX6 Solo (custom) | ~15 × 25 mm = 3.75 cm² | ~3 g |
| Pi Zero 2W | 30 × 65 mm = 19.5 cm² | ~9 g |
| ESP32-S3-WROOM-1 | 18 × 25 mm | ~3 g |
| T113-S3 SoM | ~30 × 30 mm | ~6 g |
| Apollo4 Blue Plus | 5 × 6 mm BGA + supporting | ~2 g |

For a temple-mounted board with display driver + IMU + BT/WiFi + USB-C + battery management, realistic total PCB mass is **~5–8 g for ESP32-S3 / Apollo4-class custom**, **~12–18 g for an SBC-shape Linux solution** unless we design a custom carrier. The 50g cliff requires we go custom for any Linux SoC; an SoM-based solution (RK3566 SoM on a custom carrier) is the realistic path.

#### Thermal

All candidates dissipate heat in the 10s–100s of milliwatts range during steady-state EPUB reading. None require active cooling. The i.MX6 Solo and Apollo4 are passive-cool friendly even under load. The RK3566 can hit 2 W at peak load (Mali GPU active), which against the temple is uncomfortable; **we never need to invoke that load profile** because EPUB is 2D and reflow is a one-shot per page turn. With governor pinned to the 0.6 GHz step and Mali clocked low, temple thermal is fine.

### The power math: 90 minutes / night, 5–10 s deep sleep between page turns

Assume:
- **Active reading** = display latched (e-ink is bistable, no power), SoC in WFI/idle, IMU at 50 Hz, BLE radio on at low duty cycle
- **Page-turn event** every 30 s on average (~180 page turns / 90 min) = SoC wakes to render, drives e-ink update, sleeps. Wake duration ~500 ms.
- **Deep sleep** between page turns = SoC suspended, IMU on (interrupts only), BLE on (advertising)

A simplified energy model per 90-min session: `E = P_idle × 90 min × (1 - duty) + P_active × t_wake × N_pages + P_eink_update × t_update × N_pages`

For each candidate, with rough active = page render power and idle = between-page sleep:

| SoC | P_idle | P_active | P_eink | Duty active | Energy / 90 min |
|---|---|---|---|---|---|
| RK3566 (Linux suspend-to-RAM) | 30 mW | 1500 mW | 50 mW | 1.7% (180 × 500 ms / 5400 s) | **~250 mWh** |
| i.MX6 Solo (Kobo, mature DSM) | 10 mW | 500 mW | 50 mW | 1.7% | **~30 mWh** |
| Pi Zero 2W (no real sleep) | 600 mW | 1500 mW | 50 mW | 1.7% | **~925 mWh** |
| ESP32-S3 (deep sleep) | 0.05 mW (RTC) | 250 mW | 50 mW | 1.7% | **~13 mWh** |
| T113-S3 | 10 mW | 600 mW | 50 mW | 1.7% | **~32 mWh** |
| Apollo4 (constant-on M4F) | 0.2 mW | 1 mW | 50 mW | 1.7% | **~1.5 mWh** |

These are sensible-order-of-magnitude estimates; the absolute numbers depend on PMIC efficiency, radio duty cycle, and tuning. **The relative ordering is robust:** ESP32-S3 ≪ Apollo4 ≪ i.MX6 ≈ T113 ≪ RK3566 ≪ Pi Zero 2W.

For a 500 mWh battery (≈140 mAh @ 3.7 V — about 4 g of LiPo): RK3566 fits 2 sessions; i.MX6/T113 fit 16+ sessions; Pi Zero 2W barely fits one. **Pi Zero 2W is power-architecturally disqualified** despite its excellent dev experience. The lack of a real sleep state is a structural problem we cannot solve in software.

### Putting it together

| | RK3566 | i.MX6 Solo | Pi Zero 2W | ESP32-S3 | T113-S3 | Apollo4 |
|---|---|---|---|---|---|---|
| EPUB stack quality | KOReader/Plato native | KOReader native (Kobo gold path) | KOReader native | Custom subset only | KOReader works | Custom only |
| Power profile | Heavy | Mid | Disqualifying | Excellent | Mid | Excellent |
| Dev velocity | Good | **Best** (via Kobo) | Best for hello-world | Good for ESP-IDF | OK | OK |
| BT/WiFi | External | External | Onboard | Onboard | External | BLE only |
| Caster compat | HDMI clean | HDMI on some boards | HDMI clean | SPI bridge | SPI/MIPI bridge | SPI bridge |
| Frame fit | Custom carrier needed | Excellent | Custom carrier needed | Excellent | Excellent | Excellent |
| Single-unit cost | $50 | $30–50 | $15 | $5–10 | $20–30 | $50 |
| Typography ceiling | High | High | High | Low | High | Low |

## Recommendation

### Primary: **NXP i.MX6 Solo via Kobo Clara HD as software-validation rig, then i.MX6 Solo on a custom carrier for the embedded build.**

**Why:** This is the recommendation with the highest confidence-per-watt. Three converging arguments:

1. **It ships with KOReader running today.** The Campaign 1 sourcing brief already named the Kobo Clara HD as Day-1 empirical baseline at $30–50. We're not investing in a new platform — we're reusing the most validated EPUB-reader compute platform that exists. KOReader's typography is the gold standard at 256 MB / sub-1 GHz / e-ink, and its DSM (deep sleep mode) integration on i.MX6 is mature and well-tuned by Kobo's firmware team and the KOReader project together.
2. **The power profile is the second-best of the realistic Linux SoCs (only T113-S3 ties), with mature deep-sleep behavior already in production.** ~30 mWh per 90-min session is comfortably inside any reasonable battery budget. The Kobo battery-drain GitHub issues ([7565](https://github.com/koreader/koreader/issues/7565), [9435](https://github.com/koreader/koreader/issues/9435)) document edge cases (PDF open during sleep adds ~1%/hr) that are tractable.
3. **The dev path is fastest.** Buy Kobo. Install KFMon. Install KOReader. The whole software stack — typography, page-turn UX, library — is validated against real hardware before a single mm of frame is designed. **Then** when we move to a head-worn build, we keep the same SoC and the same KOReader binary; only the display driver and form factor change.

The migration path: validate on Kobo Clara HD (~$50), then either (a) salvage the Kobo's i.MX6 board into a temple housing — possible but mechanically grim — or (b) source a fresh i.MX6 Solo SoM and design a small carrier. Path (b) is more work but gives us a clean, glasses-fit PCB. **Either way, the validated software stack is identical.**

### Fallback: **ESP32-S3 + 8 MB PSRAM**

When to switch: if the i.MX6 path proves unsourceable for the temple-board form factor (NXP Solo modules are hard to find at hobbyist scale outside of dev kits), or if the power budget ends up tighter than 300 mW expected.

ESP32-S3 wins on power (~13 mWh / session vs ~30) and BT-onboard convenience. **The cost is typography quality** — atomic14's reader is "EPUB-readable" but not "Kindle-grade." We'd have to either accept that ceiling for a v1 prototype (consistent with "personal first") or invest in a richer renderer on FreeRTOS with FreeType + a custom CSS subset (significant but bounded work).

A pragmatic interpretation of the fallback: **prototype on Kobo to validate UX and typography target, then make the final-build SoC choice with that empirical baseline in hand.** If Kobo at i.MX6 + KOReader is dramatically better than what we can build on ESP32-S3, the answer is "bite the i.MX6 carrier-design cost." If a careful ESP32-S3 build at lower typography quality is "good enough" subjectively when wearing it, take the power and weight savings.

### Not recommended

- **RK3566.** The 500 mW idle floor swallows the entire Campaign 1 power budget. Even with aggressive governor tuning, it's structurally power-hungry vs i.MX6. RK3566 makes sense for an *e-ink tablet* — Xiaomi InkPalm, PocketBook InkPad — where there's a 2,000+ mAh battery to absorb it. For temple-mounted glasses with a 500 mWh budget, it doesn't fit.
- **Pi Zero 2W.** The lack of a real sleep state is fatal for the deep-sleep-between-pages architecture (per [forum confirmation](https://forums.raspberrypi.com/viewtopic.php?t=392265) and [Hackaday](https://hackaday.com/2021/11/01/the-pi-zero-2-w-is-the-most-efficient-pi/)). Excellent for a tethered prototype on a desk — wrong tool for our use case.
- **Allwinner T113-S3.** Plausible but the ecosystem is meaningfully smaller than i.MX6's, and KOReader doesn't have a first-class T113 port. We'd be doing a fresh BSP integration. Not worth the cost vs i.MX6.
- **Apollo4.** SRAM ceiling (2.75 MB) is too tight for a CSS-reflow EPUB engine. Would force a "pre-rendered bitmap pages off-device" architecture which fights against the open-format principle (#4) — books would have to be pre-paginated on a host computer, breaking reflow for typography settings changes.
- **Qualcomm AR1.** Closed SDK. Out by program directive.

## Open questions / Gaps

1. **Custom i.MX6 Solo SoM availability at hobbyist quantities.** NXP's reference modules are EOL'ing; the realistic path may be salvaging the Kobo Clara HD's PCB or finding aftermarket modules. Worth checking [Toradex Colibri](https://www.toradex.com/computer-on-modules/colibri-arm-family/nxp-imx6) and [Variscite](https://www.variscite.com/product/system-on-module-som/cortex-a9/var-som-mx6) at Mouser. **If both are EOL, the fallback to ESP32-S3 becomes more attractive.**
2. **Empirical i.MX6 + Modos Caster integration.** Caster is documented for HDMI hosts (PC, Pi, larger SBC). Whether the Kobo's i.MX6 with no HDMI out can drive Caster cleanly is unconfirmed — it may need to drive the e-ink panel directly through the Kobo's existing E Ink controller (i.MX6 has integrated EPDC on some variants — Kobo uses the EPDC). If we keep Kobo's EPDC, Caster becomes optional.
3. **The Caster question is bigger than this brief.** If we're using Kobo's existing E Ink path (integrated EPDC + their own waveform firmware), we don't need Caster at all for prototype — Caster's value is "fast refresh," and we don't need fast refresh for static text. **Caster may be a Phase 2 upgrade, not a Phase 1 requirement.** Defer unless we want >5 Hz UI animation.
4. **BT/WiFi combo module sourcing for i.MX6.** Kobo Clara HD has a combo module already; for a custom build we'd source one (AP6256, ESP32 in slave mode, or RTL8723) — not a blocker.
5. **Bench test:** measure actual power draw on Kobo Clara HD over a 90-min KOReader session with our intended display config. The 30 mWh estimate is back-of-envelope. **High-leverage measurement** — do it Day 1.

## Sources

- [NXP i.MX6 Solo product page](https://www.nxp.com/products/processors-and-microcontrollers/arm-processors/i-mx-6-processors)
- [Rockchip RK3566 SoC overview](https://rockchips.net/rk3566-soc/)
- [Boardcon EM3566 Linux SBC](https://www.armdesigner.com/EM3566_SBC/)
- [CNX Software — Boardcon EM3566 RK3566 SoM](https://www.cnx-software.com/2021/07/28/boardcon-em3566-linux-sbc-is-powered-by-a-rockchip-rk3566-system-on-module/)
- [CNX Software — Radxa Zero 3W RK3566 SBC](https://www.cnx-software.com/2023/12/08/radxa-zero-3w-sbc-rockchip-rk3566-8gb-ram-wifi6-raspberry-pi-zero-2-w-form-factor/)
- [CNX Software — Pi Zero 2W deep-dive power](https://www.cnx-software.com/2021/12/09/raspberry-pi-zero-2-w-power-consumption/)
- [Hackaday — Pi Zero 2W is the most efficient Pi](https://hackaday.com/2021/11/01/the-pi-zero-2-w-is-the-most-efficient-pi/)
- [Pi Zero 2W headless power optimization forum](https://forums.raspberrypi.com/viewtopic.php?t=392265)
- [Lo-tech RPi Zero 2W tuning guide](https://www.lo-tech.co.uk/wiki/Tuning_the_RaspberryPi_Zero2W_for_Minimum_Power_Consumption)
- [atomic14 DIY ESP32 EPUB reader](https://github.com/atomic14/diy-esp32-epub-reader)
- [EPub-InkPlate ESP32-S3 EPUB reader](https://github.com/turgu1/EPub-InkPlate)
- [CNX Software — Allwinner T113-S3](https://www.cnx-software.com/2022/02/13/allwinner-t113-s3-dual-core-cortex-a7-soc-features-128mb-ddr3-targets-automotive-and-industrial-applications/)
- [linux-sunxi T113-s3 wiki](https://linux-sunxi.org/T113-s3)
- [CNX Software — NetCube Nagami T113 + ESP32 SoM](https://www.cnx-software.com/2025/12/01/netcube-systems-nagami-allwinner-t113-s3-mini-pcie-som-features-esp32-wireless-soc-supports-mainline-linux/)
- [Apollo4 SoC datasheet](https://contentportal.ambiq.com/documents/20123/388400/Apollo4-SoC-Datasheet.pdf)
- [Apollo4 product page](https://ambiq.com/apollo4/)
- [Modos Caster GitHub](https://github.com/Modos-Labs/Caster)
- [CNX Software — Modos Paper Dev Kit 75 Hz](https://www.cnx-software.com/2025/08/06/fpga-modos-paper-dev-kit-supports-e-ink-displays-75-hz-refresh-rate/)
- [Pervasive Displays 1.54" 200×200 EPD-driving-circuit reference](https://www.pervasivedisplays.com/wp-content/uploads/2020/12/EPD-driving-circuit-of-Pervasive-Displays-rev.01.pdf)
- [Pervasive Displays product page](https://www.pervasivedisplays.com/product/e-ink-displays-etc/)
- [KOReader GitHub](https://github.com/koreader/koreader)
- [KOReader MobileRead wiki](https://wiki.mobileread.com/wiki/KOReader)
- [Plato GitHub](https://github.com/baskerville/plato)
- [KFMon (Kobo install loader)](https://github.com/NiLuJe/kfmon)
- [Kobo Clara HD KOReader battery drain (issue 7565)](https://github.com/koreader/koreader/issues/7565)
- [Kobo Clara HD KOReader battery drain (issue 9435)](https://github.com/koreader/koreader/issues/9435)
- [Toradex Colibri NXP i.MX6 SoM](https://www.toradex.com/computer-on-modules/colibri-arm-family/nxp-imx6)
- [Variscite VAR-SOM-MX6](https://www.variscite.com/product/system-on-module-som/cortex-a9/var-som-mx6)
