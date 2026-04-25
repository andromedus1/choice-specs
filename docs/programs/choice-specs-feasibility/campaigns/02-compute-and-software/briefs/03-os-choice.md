---
description: "OS choice — Buildroot Linux on i.MX6 Solo recommended primary; FreeRTOS as fallback if SoC drops to ESP32-S3. Debian/Yocto rejected for prototype scope; bare-metal and microkernels rejected as overengineering."
type: brief
campaign: 02-compute-and-software
program: choice-specs-feasibility
research_method: deep-research
confidence: speculative
status: draft
updated: 2026-04-25
---

# OS Choice

## Question

Given the [recommended SoC of i.MX6 Solo](01-soc-selection.md) (with ESP32-S3 as fallback) and the [recommended renderer of KOReader](02-epub-renderer.md) (with custom on ESP32-S3 as fallback), what operating system runs on the device?

The OS choice has to satisfy:

1. **KOReader / Plato compatibility** — both assume Linux + framebuffer (`/dev/fb0`). This biases hard toward Linux on the primary SoC path.
2. **Deep sleep + fast wake** — must drop to <50 mW between page turns and wake within ~250 ms on a BT key event or IMU interrupt. The actual page-turn user-experience latency budget is ~500 ms total.
3. **Boot time** — first-power-on can take longer; from-deep-sleep wake must be sub-second.
4. **Update path** — the device must be updatable from a host computer (USB-C connected). Bricking the device by a half-finished update is unacceptable per principle 5 (sleep-grade reliability).
5. **Power consumption at idle** — see brief 1 power math; the OS choice contributes to the idle floor.
6. **Build complexity for a single hobbyist developer** — every hour spent fighting the build system is an hour not spent on the actual product.

## Findings

### Linux flavors: Buildroot, Yocto, Debian/Ubuntu, Alpine

The [Buildroot vs Yocto comparison literature](https://www.incredibuild.com/blog/yocto-or-buildroot-which-to-use-when-building-your-custom-embedded-systems) converges on a clear pattern:

- **Buildroot** wins for: small teams, single-purpose devices, fast iteration, smallest possible image, simplest mental model. Build is `make`-based and the whole system is comprehensible to one developer. Image sizes typically 50–200 MB on disk for a usable embedded Linux.
- **Yocto** wins for: multi-product portfolios, long-term security maintenance, hundreds of recipes, professional commercial trajectory. ~8,400 packages available; layered architecture. Higher learning curve.
- **Debian/Ubuntu** wins for: rapid prototyping with all conveniences (apt, systemd, full bash). Larger image (~500 MB+); slower boot; less power-tuned by default. Used in dev SBCs (Raspberry Pi OS) but not in production embedded devices.
- **Alpine Linux** is a possible third path — musl libc, BusyBox, very small (~5 MB base). Used in Docker containers and some embedded products. Less common for ARM e-readers.

For choice-specs's "single-purpose device, single developer, want fast iteration, want small image" profile, **Buildroot is the textbook fit.** Yocto is the textbook anti-pattern (overengineered for solo personal projects); Debian is the comfortable-but-power-hungry option.

[ELBE](https://www.cnx-software.com/2020/11/18/elbe-is-a-simpler-alternative-to-yocto-openembedded-and-buildroot/) is a dark-horse simpler alternative (Debian-based, layered like Yocto). Mentioned for completeness; not chosen here.

### Linux deep-sleep behavior

The relevant Linux power state for our use case is **suspend-to-RAM (S3 / `mem`)**. On i.MX6 Solo with NXP's BSP, suspend-to-RAM achieves the documented **5–15 mW DSM** range. The Kobo Clara HD uses this exact path; it's the same code path KOReader uses today.

On RK3566, suspend-to-RAM can reach ~30 mW with vendor BSP work; out-of-the-box Buildroot is closer to 100–500 mW depending on what peripherals stay on. NXP's i.MX6 BSP does this work for us; Rockchip's BSP is younger.

Wake-from-suspend latency on i.MX6 Solo:
- Wake source = GPIO (BT controller IRQ or IMU IRQ): ~50–150 ms to user-space process resume.
- Wake source = RTC alarm: similar.
- KOReader-resume-then-render-page-turn additional cost: ~80–250 ms (per [brief 2](02-epub-renderer.md)).
- **Total page-turn UX latency from sleep: ~150–400 ms.** Inside the 500 ms budget.

### FreeRTOS

[FreeRTOS](https://www.freertos.org/) is the dominant RTOS for ARM Cortex-M class chips. ESP-IDF (Espressif's SDK) ships FreeRTOS as the default on ESP32-S3.

**Power profile:** Tight. Light-sleep on ESP32-S3 ≈ 0.8 mA; deep-sleep with RTC retained ≈ 10 µA. Wake-from-deep-sleep ≈ 250 ms (RTC re-init, RAM clear, app boot). Wake-from-light-sleep ≈ 1 ms.

**KOReader/Plato compatibility:** None. They require Linux. **FreeRTOS is only relevant if we take the [brief 1 fallback to ESP32-S3](01-soc-selection.md).**

**Update path:** Espressif's OTA story is mature. USB-C → ESP-IDF flasher works for development; OTA-from-WiFi works for production. Dual-partition with rollback prevents bricking.

**Boot time:** ~500 ms cold boot to user app. Excellent.

### Zephyr RTOS

[Zephyr](https://www.zephyrproject.org/) is the Linux Foundation's RTOS, increasingly the preferred path for new Cortex-M designs. Targets a broader SoC range than FreeRTOS (Cortex-M0/M3/M4/M7 plus some M55, plus RISC-V). Power features: tickless idle, multiple sleep states, well-developed power-management framework.

**Choice-specs relevance:**
- Excellent fit for Apollo4. Apollo4 has a Zephyr port; Ambiq actively contributes to the project.
- Plausible for ESP32-S3 (Espressif maintains a Zephyr port alongside ESP-IDF/FreeRTOS).
- Not relevant for Linux-class SoCs.

**KOReader/Plato compatibility:** None — same as FreeRTOS.

If we were targeting Apollo4, Zephyr would be the best fit (better than bare-metal, better-supported than FreeRTOS on that platform). But Apollo4 was rejected in [brief 1](01-soc-selection.md) for SRAM ceiling. **Zephyr drops out by transitivity.**

### Bare-metal

Direct-to-hardware code with no OS. Reasonable for ESP32-S3 / Apollo4-class chips with simple workloads. Lowest possible power consumption (no scheduler tick overhead). Highest dev cost (we own scheduling, IRQ handling, memory management).

**For choice-specs:** rejected. Even on ESP32-S3 the EPUB rendering pipeline is concurrent enough (BLE stack + IMU + render + display) that we want a scheduler. FreeRTOS or Zephyr is the right tool.

### Custom microkernel

Mentioned in the program brief as a candidate. **Rejected on overengineering grounds.** Microkernels (seL4, MINIX 3) are real engineering paths for high-assurance systems. Choice-specs is a personal bedside reader. The reliability surface is well-served by mature monolithic kernels (Linux on i.MX6, FreeRTOS on ESP32-S3).

### Per-SoC OS recommendations

| SoC | Best-fit OS | Why |
|---|---|---|
| i.MX6 Solo | **Buildroot Linux** with NXP's i.MX6 BSP | KOReader runs natively; suspend-to-RAM is mature; 5–15 mW DSM achieved in production |
| RK3566 | Buildroot Linux with Rockchip BSP | Same model; less mature suspend tuning; would need work |
| Pi Zero 2W | Raspbian (Debian-derived) | Easiest dev; not relevant since SoC was disqualified |
| ESP32-S3 | **FreeRTOS** via ESP-IDF | Native; best-supported; minimal overhead |
| T113-S3 | Tina Linux (Allwinner) or Buildroot | Mainline supported but smaller community |
| Apollo4 | Zephyr | Better-supported than FreeRTOS for that chip |

### Update path

For Linux on i.MX6:
- **USB-C OTG mass-storage mode** during a button-held boot. The device appears as a USB drive to host; user drops a new image into a `/update/` directory.
- Boot loader (U-Boot) on next reboot validates the image and either swaps active partition or reverts on failure.
- This is exactly Kobo's update path; we copy the pattern.

For FreeRTOS on ESP32-S3:
- **USB-C CDC-ACM** with `esptool.py` from a host. Standard ESP-IDF flow.
- A/B partition with rollback on boot-failure; ESP-IDF supports this natively.

Both paths satisfy "no bricking on bad update."

### Boot time targets

For sleep-grade reliability per principle 5:
- **Cold boot** (battery insertion, dead device): user-acceptable up to ~5 s. Linux on i.MX6 Solo: 8–15 s out of the box; tunable to ~3 s by stripping init scripts. FreeRTOS on ESP32-S3: ~500 ms.
- **Wake from deep-sleep**: must be ≤500 ms total (sleep-to-page-rendered). Linux suspend-to-RAM on i.MX6: ~150 ms wake + ~200 ms KOReader resume = ~350 ms — fits. ESP32-S3 light-sleep: ~10 ms — easy fit.

Cold boot tuning notes for i.MX6:
- Strip systemd if using it; init = `/init` shell script.
- Disable services: no SSH daemon, no networking init at boot, no syslog daemon (we don't need any of these for a reader).
- `quiet loglevel=0` kernel cmdline.
- Read-only root filesystem.

### Idle power consumption

From [brief 1](01-soc-selection.md)'s power math:

- i.MX6 Solo + tuned Buildroot Linux suspended: ~10 mW
- ESP32-S3 + FreeRTOS deep-sleep: ~0.05 mW (essentially RTC only)
- ESP32-S3 + FreeRTOS light-sleep: ~3 mW

These are **sleep** numbers. Active reading (display latched, SoC waiting for input) is closer to 100–200 mW for the SoC + BLE + IMU stack. But we expect to be in deep-sleep most of the time during a 90-min reading session (the reader looks at the page for ~30 s between turns; the device sleeps for ~25 s of that).

## Recommendation

### Primary: **Buildroot Linux on i.MX6 Solo, with NXP's BSP for power management.**

Concretely:

- **Build system:** Buildroot. NXP maintains an `i.MX6` Buildroot defconfig; Kobo's downstream patches are public and we cherry-pick what we need.
- **Init:** simple shell `/init` script — no systemd, no upstart. Runs `koreader.lua` directly as PID 2 after early hardware init.
- **Filesystem:** read-only root + small `/var` overlay for runtime state (last-read book, position, settings).
- **Power management:** `mem` suspend triggered by KOReader's idle timer; wake on GPIO from BT controller or IMU IRQ.
- **Update path:** U-Boot dual-partition + USB-C OTG mass-storage drop-folder. Modeled on Kobo.
- **Bootloader:** U-Boot.
- **Kernel:** mainline Linux 6.x with NXP's downstream patches.

Why not Yocto: overengineering for one-developer project. Why not Debian: power overhead, slow boot, image too large for a single-purpose device.

**Strong principle alignment:**
- Principle 5 (sleep-grade reliability): well-trodden path; Kobo has shipped this exact stack for years.
- Principle 1 (single purpose): an init script that runs only KOReader is structurally single-purpose.
- Principle 4 (open formats): Linux + KOReader has no DRM in the stack.

### Fallback: **FreeRTOS on ESP32-S3 via ESP-IDF.**

Tied to [brief 1's fallback](01-soc-selection.md). Pre-existing reference designs (atomic14, EPub-InkPlate) make this a tractable development path. We get OTA, deep-sleep, BLE, all in the box. The cost is the typography ceiling — same as in brief 2.

### Not chosen

- **Yocto.** Right tool for product-phase re-engineering (multi-variant SKUs, security maintenance contracts). Wrong tool for solo prototype.
- **Debian/Ubuntu.** Power-hungry by default; slower boot; image bloat. Fine for a desk-tethered prototype; wrong for a battery-powered head unit.
- **Zephyr.** Right answer for Apollo4 if we'd chosen it; Apollo4 was disqualified upstream.
- **Bare-metal.** Concurrency surface needs a scheduler.
- **Custom microkernel.** Overengineering for the use case.

## Open questions / Gaps

1. **NXP i.MX6 Solo BSP mainline status.** The i.MX6 family is mature but ageing; mainline kernel support is excellent. **What we should validate:** can we build a current Buildroot for i.MX6 Solo from scratch (not from Kobo's image)? Bench test in Phase 2.
2. **Boot-time floor on i.MX6 Solo.** 3-second cold boot is achievable but requires init-script discipline. We'll measure during integration.
3. **Wake-from-suspend reliability.** Production embedded Linux on i.MX6 reliably wakes from `mem` suspend. Edge cases (kernel deadlock during resume, lost IRQ) are vanishingly rare but exist. **Mitigation:** independent watchdog timer that hard-resets the SoC if user-space doesn't pet it within 30 s after a wake event. Adds ~5 µA but eliminates a class of "bricked at 2am" failures.
4. **Read-only root filesystem + writable overlay** is the right answer for sleep-grade reliability but requires care: where do KOReader's caches live? `/var/koreader/` overlay. Where do EPUBs live? `/books/` mounted from a separate partition (so we can update root without touching books).
5. **OTA over WiFi** is not a Phase-1 concern (USB-C update is fine). Defer until productization.
6. **Suspend behavior with BLE active.** BLE controller stays active during system suspend (battery cost ~2–5 mW). Worth measuring; if too high, we sleep BLE during deep nights and accept "no ring keypress wake" — only IMU and physical-button wake.

## Sources

- [Yocto vs Buildroot — Incredibuild](https://www.incredibuild.com/blog/yocto-or-buildroot-which-to-use-when-building-your-custom-embedded-systems)
- [Yocto vs Buildroot — Ezurio](https://www.ezurio.com/resources/blog/yocto-vs-buildroot-comparing-the-two-platforms)
- [Yocto vs Buildroot — Axceta](https://axceta.com/embeddedlinux/)
- [LWN — Deciding between Buildroot & Yocto](https://lwn.net/Articles/682540/)
- [4 tools for building embedded Linux — opensource.com](https://opensource.com/article/18/6/embedded-linux-build-tools)
- [Buildroot project](https://buildroot.org/)
- [Yocto Project](https://www.yoctoproject.org/)
- [ELBE — simpler alternative](https://www.cnx-software.com/2020/11/18/elbe-is-a-simpler-alternative-to-yocto-openembedded-and-buildroot/)
- [FreeRTOS](https://www.freertos.org/)
- [Zephyr Project](https://www.zephyrproject.org/)
- [ESP-IDF documentation](https://docs.espressif.com/projects/esp-idf/en/latest/esp32s3/)
- [NXP i.MX6 BSP / Yocto layer](https://www.nxp.com/design/software/embedded-software/i-mx-software/embedded-linux-for-i-mx-applications-processors:IMXLINUX)
- [Kobo postmarketOS port](https://www.cnx-software.com/2021/07/13/diy-linux-e-reader-postmarketos-now-runs-on-kobo-clara-hd/)
- [Kobo SDK](https://github.com/kobolabs/Kobo-Reader)
- [Kobo battery drain in suspend (issue 7565)](https://github.com/koreader/koreader/issues/7565)
