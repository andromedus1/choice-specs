---
description: "Component sourcing as a hobbyist — vendor channels, MOQs, lead times, dev kits for displays, optics, electronics"
type: brief
campaign: 01-display-and-optics
program: choice-specs-feasibility
research_method: deep-research
confidence: speculative
status: draft
updated: 2026-04-25
---

# Component Sourcing as a Hobbyist

## Question

Choice-specs is a personal feasibility prototype with a $2–5k budget cap. End-to-end: how does someone with $2–5k actually buy the components for an AR-glasses-class device? What are the realistic distributors, direct vendor relationships, MOQs, lead times, sample policies, and bundled dev kits? Where do hobbyists actually source AR motherboards, microdisplays, optics, IMUs, batteries, BT clickers — and what are the community channels for trading or pooled buys?

## Findings

### Tier 1: Mainstream electronic distributors

These are the easiest, fastest, most-reliable channels. They carry "everything except the most exotic AR optics."

- **DigiKey** — global, US-warehoused, single-piece MOQ on most parts, same-day shipping. Carries E Ink (Pervasive Displays badge), Sharp Memory LCD, JDI MIP, all standard MCUs and SoCs (RK3566 dev boards, ESP32-S3, i.MX6, Pi Zero), batteries, Bluetooth modules, IMUs (Bosch BMI270, ICM-20948), connectors. **Default first stop for any component you can name.** [DigiKey](https://www.digikey.com/) [DigiKey distributors page](https://www.digikey.com/en/supplier-centers/e-ink) [DigiKey Pervasive Displays](https://www.digikey.com/en/supplier-centers/pervasive-displays)
- **Mouser** — same role as DigiKey; sometimes cheaper on specific lines. Slightly less stocked on Asian vendors. Use as backup.
- **Arrow / Avnet** — corporate-leaning; fewer dev-kit listings for individuals. Skip.
- **Adafruit** — boutique hobbyist-friendly. Stocks small e-paper, OLED, sensors; great for breakout boards and Adafruit-branded BT clicker prototypes. [Adafruit ePaper category](https://www.adafruit.com/category/150)
- **SparkFun** — similar to Adafruit; less e-paper focus.
- **The Pi Hut (UK)** — UK Adafruit-equivalent. Stocks Pervasive Displays small panels. [The Pi Hut 1.54" e-ink panel](https://thepihut.com/products/eink-screen-1-54-200x200-display-only)
- **Crystalfontz** — niche US distributor with OLED + ePaper specialization. [Crystalfontz ePaper](https://www.crystalfontz.com/c/epaper-displays/519)

For **standard components** (MCU, IMU, battery cell, BMS, USB-C charging IC, connectors, BT module), DigiKey + Mouser cover everything choice-specs needs at single-unit MOQ in 1–3 day shipping.

### Tier 2: Chinese marketplaces and AR-specialty resellers

For **microdisplays + optics**, the supply chain is largely China-direct.

- **LCSC (Shenzhen)** — Chinese DigiKey-equivalent; strong on JLCPCB-compatible inventory; prices ~30–50% lower than DigiKey for common components. 5–10 day shipping. Single-unit MOQ. [LCSC at Accio](https://www.accio.com/t-v2/plp/lcsc-electronics) Useful for second-source on standard parts and for cheaper IMUs / sensors.
- **Alibaba** — wholesale-oriented, MOQ varies (often 1–10 units for sample, 100+ for production). Best channel for: micro-OLED panels with driver boards (BOE, SeeYa, Sony ECX-clones — $300–800), AR motherboards (Rockchip RK3566/RK3588 reference boards — $100–300), pancake lens assemblies (questionable quality, $30–150), birdbath optical modules (off-brand resellers $300–800). [AR glasses components on Alibaba](https://www.alibaba.com/showroom/ar-glasses-components-list.html) [AI glasses with display wholesalers](https://www.alibaba.com/supplier/ai-glasses-with-display-wholesaler.html) **Caveat:** quality is highly variable; first orders should be small. Vendor reputation matters; use suppliers with 5+ years of history and confirmed Trade Assurance.
- **AliExpress** — retail-side of Alibaba; single-unit ordering; includes low-volume hobbyist parts. Slightly higher price, easier dispute resolution.
- **Made-in-China.com** — alternative to Alibaba, sometimes better for industrial-spec listings (e.g., the 0.71" SeeYa/BOE microdisplays). [Made-in-China 0.71" microdisplays](https://www.made-in-china.com/showroom/0e8010ae45e28857/product-detailCOMtNyoulGfc/China-Seeya-Boe-Sony-0-7-0-71-0-72inch-Amoled-Microdisplays-Si-OLED-1920X1200-1080P-FHD-1920X1080-Ecx335s-Vr-Ar-Microoled-Display.html)
- **DisplayModule (US-based)** — specializes in AR/VR optical modules. Birdbath dev kits, custom monocular/binocular modules, often integrated micro-OLED. $400–2,500 per kit. [DisplayModule birdbath dev kit](https://www.displaymodule.com/products/binocular-birdbath-ar-headset-development-kit) **Best one-stop for someone who wants a working AR optical engine in the box.**
- **Hicenda** — DisplayModule competitor; similar product range. Sometimes better pricing. [Hicenda binocular birdbath](https://hicenda.com/product/Birdbath-optics-Binocular-AR-Glasses.html)
- **YXMicroDisplay** — Chinese AR-optics reseller, US shipping. Birdbath modules, micro-OLED, LCoS sometimes. [YXMicroDisplay binocular birdbath](https://www.yxmicrodisplay.com/products/binocular-ar-glasses-birdbath-1920-1080-50-fov-optical-module-lvds)

### Tier 3: Direct vendor relationships

For panels and optics that require co-development:

- **E Ink Corp (Taiwan)** — direct only for sub-1.3" reading-grade panels. NDA + MOQ in the 1k–10k range. NRE in $50k–250k for custom panel work. **Out of scope for personal prototype.** Productization-only.
- **Pervasive Displays (Taiwan)** — distributor channel via DigiKey for standard panels; direct sales for custom work. Single-unit MOQ on 1.54"–4.2" standard panels. **Default e-ink sourcing channel for choice-specs.**
- **Good Display (China)** — direct sales via website, some panels stocked on AliExpress / Alibaba. Single-unit MOQ for most panels. Best for unusual sizes (0.97" EOL, 1.02" flexible, etc.). [Good Display 0.97"](https://www.good-display.com/product/434.html)
- **Kopin (US)** — direct via sales@kopin.com. No public retail channel. Sample orders likely $500–2,000 with eval kit; lead time 4–8 weeks. **Required for LCoS path.**
- **ForthDD (UK)** — direct only. Higher cost ($3k–8k per panel). Aimed at scientific buyers. Not hobbyist channel.
- **Sony Semiconductor** — gated through Japanese OEM channels. Effectively unavailable to US individuals.
- **JBD MicroLED (China)** — direct sales for OEM only; small-volume hobbyist sample policy unclear. [JBD MicroLED](https://www.jb-display.com/)

### Tier 4: Used / salvaged hardware

For optics specifically, **eBay and disassembled consumer products** are a real channel.

- **Used Quest 3 / Quest Pro** — $200–400 on eBay in 2026. Pancake lenses are the salvage target (Brief 5). 1–2 week shipping; condition varies.
- **Used Epson Moverio BT-40 / BT-300** — $200–400 on eBay. Birdbath-class optics + OLED engine. (Brief 7 covers; not recommended.)
- **Used XReal / Rokid / VITURE glasses** — $150–400 used. Birdbath + micro-OLED. Could be torn down for components but each one is a custom integration.
- **eBay for Sharp Memory LCD, JDI MIP** — surplus listings sometimes appear.

Used hardware is **the only practical way to get "high quality" pancake optics as an individual**.

### Optical components from optics specialists

For custom optical engineering (Brief 6 closed-birdbath path, custom illumination assemblies):

- **Edmund Optics** — premier US optics distributor. Mirrors, beamsplitters, polarizers, lenses, prisms. Single-unit MOQ. $50–500 typical part. [Edmund Optics](https://www.edmundoptics.com)
- **Thorlabs** — lab-grade optics; broader range than Edmund; same price range. [Thorlabs](https://www.thorlabs.com)
- **Newport, Newport-MKS** — pricier, better for custom-spec parts.
- **Surplus Shed (US)** — surplus optics; cheap for prototyping. Quality variable.

For warm-LED illumination (Brief 2 LCoS path): a 590–605nm amber LED + driver from DigiKey is $5–20.

### IMU / sensor sourcing

- **Bosch BMI270** — gold-standard 6-axis IMU for wearables; DigiKey single-unit ~$5. Used in Apple Watch, Fitbit. Low power, embedded ML support.
- **Bosch BMI323** — newer, slightly better motion classification. ~$5.
- **InvenSense ICM-20948** — 9-axis (mag + accel + gyro). DigiKey single-unit ~$15. Higher accuracy on absolute orientation; slightly more power.
- **STMicro LSM6DSV** — 6-axis with embedded sensor fusion. Newer.
- **Adafruit BNO055 breakout** — 9-axis with on-chip sensor fusion (no SoC math required). Pricier (~$30) but easy for prototype. Great for first IMU integration.

[Adafruit BNO055 — from sensor catalog](https://www.adafruit.com/category/150) DigiKey carries all of the above.

### Compute platform sourcing

- **Rockchip RK3566 SBC (Radxa Zero 3, Pine64 Quartz64)** — $30–80 from manufacturer or distributor. Single-unit. Linux-native. [Eleclifeoem RK3588 motherboard](https://eleclifeoem.com/product/ar-vr-smart-glasses-and-smart-helmets-motherboard-solution-based-on-rockchip-rk3588/)
- **Raspberry Pi Zero 2W** — $15. Ubiquitous. Slightly higher power than RK3566.
- **ESP32-S3 dev board** — $10–20. For minimum-viable-compute path.
- **Kobo Clara HD (used)** — $30–50 on eBay. Pre-built KOReader-validated stack (per scout-landscape Theme F). Buy one, hack with KFMon, validate the software stack on the cheapest possible hardware before committing to custom.

### BT clicker / ring sourcing

- **Gevo BT Page Turner Ring** — Amazon $20–30. Pairs as BT HID. Per scout-landscape Theme K, **the fallback input is fully de-risked at $25.** [Gevo ring at goodereader.com](https://goodereader.com/blog/technology/gevo-bluetooth-page-turner-ring-will-allow-for-a-hands-free-reading-experience)
- **Generic BT 5.0 page-turn clickers** — Amazon $10–20.
- **Genki Wave Ring** — over-featured but demonstrates haptic confirmation. $200+. [Genki](https://genkiinstruments.com/)
- **Kobo Remote** — Kobo branded; works as BT HID. $30–40.

### Battery + power

- **LiPo cells** — Adafruit, SparkFun, Mouser. 100–500mAh single-cell flat packs $5–15. **Avoid Alibaba LiPo cells without certification** — sleep-context safety per Bose Sleepbuds lesson.
- **USB-C PD charging IC + BMS** — TI BQ25180 family from DigiKey ~$2 + breakout designs from Adafruit.
- **PMIC** — RK3566 has a companion PMIC (RK809) bundled with most SBCs.

### Frame / mechanical

- **3D printing service:** PCBWay, JLCPCB-3D, Shapeways. Resin SLA for $20–60 per frame iteration. **Multi-material 3D-print is the sweet-spot fabrication path** (per Campaign 4 territory).
- **TR-90 / acetate frame blanks** — eyewear-specialty supply (BMC Frames, Eyewear-Specialist) $50–150 per blank.
- **Hinges, nose pads** — eyewear-supply specialists; $5–20 per part.

### Community channels

- **Project North Star Discord** — active, ~thousands of members; mostly North Star focused but active discussion of optical components, sourcing, calibration. [Project North Star community at Smart Prototyping](https://www.smart-prototyping.com/blog/Building-Project-North-Star)
- **MentraOS Discord** — Mentra Community / open smart glasses; active, software-focused but with some hardware discussion. [MentraOS GitHub](https://github.com/Mentra-Community/MentraOS)
- **Hackaday.io** — projects + comments; not real-time but well-archived. [Hackaday smart glasses tag](https://hackaday.com/tag/smart-glasses/) [Hackaday AR tag](https://hackaday.com/tag/augmented-reality/)
- **r/augmentedreality** — Reddit, modest activity, some sourcing questions.
- **MobileRead Forums** — e-reader-focused; useful for the e-ink-specific community. [MobileRead Sol Reader thread](https://www.mobileread.com/forums/showthread.php?t=351371)
- **KGOnTech blog** — Karl Guttag's blog, not a community but the canonical AR-optics commentary source. [KGOnTech](https://kguttag.com/)
- **Tindie** — low-volume hobbyist marketplace; some birdbath modules, AR optics. [Tindie AR birdbath](https://www.tindie.com/products/oled-module/ar-birdbath-optics-19201080-50-fov-optical-modul/)
- **Crowd Supply** — successful crowdfunding for hobbyist hardware (Modos Paper Monitor, Reform laptop). Worth monitoring for AR hardware.

**No "AR pooled-buy group" surfaced** as a formal community. Pooled buys for unusual MOQs (e.g. a 100-unit run of an obscure E Ink panel) might be coordinated through the Project North Star Discord but require the host to organize.

### Realistic budget allocation for a $2–5k prototype

| Category | Items | $ |
|---|---|---|
| Display panel(s) | 1× Pervasive Displays 1.54" e-ink + 2 spares | 50 |
| Display driver | Modos Caster FPGA dev board (or custom) | 100–250 |
| Optics | Used Quest 3 (salvaged for one pancake lens) | 250 |
| Compute | RK3566 SBC + accessories | 80 |
| IMU | Adafruit BNO055 breakout | 30 |
| BT module | Built into SoC + reference BT antenna | 5 |
| Battery + BMS + USB-C charging | 1000mAh LiPo + BQ25180 board | 30 |
| BT ring (input fallback) | Gevo BT page turner | 25 |
| Frame iteration | 3D-printed SLA frames × 5 iterations | 200 |
| Hinges, nose pads, fasteners | eyewear specialty | 75 |
| Optical-bench misc | Edmund/Thorlabs polarizers, mirrors | 200 |
| Sol Reader (empirical baseline) | $249 | 249 |
| Kobo Clara HD (software baseline) | $50 | 50 |
| Tools (SLA printer if not owned, soldering, etc.) | varies | 0–500 |
| Contingency | 25% buffer | 350 |
| **Total** | | **$1,750–2,200** |

**A solid prototype is buildable for ~$2k.** Adding LCoS path (Kopin eval kit) takes us to ~$3.5k. Adding custom PCB fab (JLCPCB) adds $200–500 per iteration.

## Recommendation

**The $2–5k budget is realistic for a working monocular e-ink prototype, with margin to spare. The key procurement decisions are sequential, not parallel — each unblocks the next.**

Specifically:

1. **Order Day 1 ($300):** Sol Reader + Kobo Clara HD. **Empirical baselines before any custom work.**
2. **Order Day 14 ($150):** Pervasive Displays 1.54" e-ink panels (3 units), Adafruit BNO055, Bosch BMI270 breakout, RK3566 SBC. Software-stack validation begins.
3. **Order Day 30 ($300–500):** Used Quest 3 for pancake salvage; Modos Caster (or alternative FPGA dev board); LiPo + BMS components. Hardware integration begins.
4. **Order Day 60 ($200):** SLA-printed frame iteration; Edmund Optics misc bench supplies; BT ring. First wearable prototype.
5. **Reserve $1,000 for unblocked downstream Campaign 2/3/4 components.** SoC change, custom PCB fab, additional optics, additional frame iterations.

**Sourcing strategy by tier:**
- **Tier 1 (DigiKey/Mouser/Adafruit) for everything you can name.** Single-unit MOQ, fast shipping, no risk.
- **Tier 2 (Alibaba/AliExpress/DisplayModule) for AR-specific optics + microdisplays only.** Always small-batch initial orders; vet supplier history.
- **Tier 3 (direct E Ink, Kopin) only if/when productizing.** Out of scope for prototype phase.
- **Tier 4 (eBay salvage) for pancake optics specifically.** No alternative for hobbyist-grade pancake.

**Community channels:**
- **Project North Star Discord and MentraOS Discord** — join both. Lurking yields sourcing tips that don't surface in web search.
- **MobileRead Forums** — for the e-ink-specific community (Sol Reader users, Kobo hackers, KOReader devs).
- **KGOnTech blog** — read everything Karl Guttag writes about AR optics; he's the single best signal source on what's real vs. vaporware.

**Don't try to:**
- Order from E Ink Corp directly. (Wrong tier for prototype.)
- Order custom Kopin until the LCoS path is committed. (Premature.)
- Order from sketchy Alibaba "ECX335 SeeYa" listings without supplier history. (Quality risk.)
- Salvage Moverio. (Bad ROI per Brief 7.)

The headline: **build with mainstream distributors + Sol Reader for empirical baseline + a Quest 3 for optics salvage. The supply chain is friendlier than the scout's "no sub-1" e-ink microdisplay" finding implied.**

## Open questions / Gaps

- **MentraOS / Vuzix Z100 Developer Edition retail availability and price** — relevant if Campaign 2 chooses the "skip-hardware via MentraOS" path. Vuzix Z100 lists at ~$500–700 retail; MentraOS SDK is open. [Vuzix Z100](https://www.vuzix.com/products/z100-smart-glasses)
- **Pooled-buy logistics** for E Ink Corp custom panels at hobbyist tier — could a 10-person Discord group order 100 panels and split? Unproven but possible. Worth asking on Project North Star Discord.
- **Custom PCB lead time and cost** — JLCPCB / PCBWay 4-layer assembled boards run $50–200 in 1–2 weeks. Standard procurement; not an unknown.
- **Component shortages** — 2024–2025 MCU shortages have largely cleared but specific lines (RK3566, certain Bosch IMUs) may have lead-time blips. Check stock at order time.
- **Tariff impact (US 2025–2026)** — variable; Chinese components have shifting tariff exposure. Add 10–25% buffer for Tier 2 / 3 orders if shipping to US.

## Sources

- [DigiKey](https://www.digikey.com/)
- [DigiKey — E Ink supplier center](https://www.digikey.com/en/supplier-centers/e-ink)
- [DigiKey — Pervasive Displays supplier center](https://www.digikey.com/en/supplier-centers/pervasive-displays)
- [DigiKey on Wikipedia](https://en.wikipedia.org/wiki/DigiKey)
- [Adafruit ePaper category](https://www.adafruit.com/category/150)
- [Adafruit 1.54" e-Ink Display product](https://www.adafruit.com/product/4196)
- [Crystalfontz ePaper](https://www.crystalfontz.com/c/epaper-displays/519)
- [The Pi Hut — 1.54" e-ink panel](https://thepihut.com/products/eink-screen-1-54-200x200-display-only)
- [LCSC at Accio](https://www.accio.com/t-v2/plp/lcsc-electronics)
- [Made-in-China — 0.71" microdisplays](https://www.made-in-china.com/showroom/0e8010ae45e28857/product-detailCOMtNyoulGfc/China-Seeya-Boe-Sony-0-7-0-71-0-72inch-Amoled-Microdisplays-Si-OLED-1920X1200-1080P-FHD-1920X1080-Ecx335s-Vr-Ar-Microoled-Display.html)
- [Alibaba — AR glasses components](https://www.alibaba.com/showroom/ar-glasses-components-list.html)
- [Alibaba — AI glasses with display wholesalers](https://www.alibaba.com/supplier/ai-glasses-with-display-wholesaler.html)
- [DisplayModule — Birdbath AR Headset Development Kit](https://www.displaymodule.com/products/binocular-birdbath-ar-headset-development-kit)
- [DisplayModule — Birdbath optical module](https://www.displaymodule.com/products/binocular-araugmented-reality-glasses-birdbath-1920-1080-47-fov-optical-module-with-lvds)
- [Hicenda — Birdbath optics binocular AR glasses](https://hicenda.com/product/Birdbath-optics-Binocular-AR-Glasses.html)
- [YXMicroDisplay — Birdbath module](https://www.yxmicrodisplay.com/products/binocular-ar-glasses-birdbath-1920-1080-50-fov-optical-module-lvds)
- [Tindie — AR birdbath optics 1920×1080](https://www.tindie.com/products/oled-module/ar-birdbath-optics-19201080-50-fov-optical-modul/)
- [INAIRSPACE — DIY AR Glass guide](https://inairspace.com/blogs/learn-with-inair/diy-ar-glass-your-ultimate-guide-to-building-affordable-augmented-reality-at-home)
- [INAIRSPACE — Make Your Own AR Glasses](https://inairspace.com/blogs/learn-with-inair/make-your-own-ar-glasses-a-comprehensive-diy-guide-to-building-your-personal-augmented-reality)
- [Hackaday — Pi Zero To AR DIY](https://hackaday.com/2024/11/04/pi-zero-to-ar-building-diy-augmented-reality-glasses/)
- [Hackaday smart glasses tag](https://hackaday.com/tag/smart-glasses/)
- [Hackaday augmented reality tag](https://hackaday.com/tag/augmented-reality/)
- [Hackaday.io — PsuedoPancakes](https://hackaday.io/project/187343-the-psuedopancakes)
- [Hackaday.io — DIY Smartglasses with Raspberry Pi](https://hackaday.io/project/166218-diy-smartglasses-with-the-raspberry-pi)
- [Hackster.io — Build Your Own AR Glasses From Scratch](https://www.hackster.io/news/you-can-actually-build-your-own-ar-glasses-from-scratch-d95811cd6c64)
- [Altium — components behind AR glasses](https://resources.altium.com/p/smart-and-ar-glasses-components)
- [Eleclifeoem — Rockchip RK3588 motherboard for AR/VR/smart glasses](https://eleclifeoem.com/product/ar-vr-smart-glasses-and-smart-helmets-motherboard-solution-based-on-rockchip-rk3588/)
- [Vuzix Z100 Smart Glasses](https://www.vuzix.com/products/z100-smart-glasses)
- [Mentra Community — MentraOS GitHub](https://github.com/Mentra-Community/MentraOS)
- [Smart Prototyping building Project North Star](https://www.smart-prototyping.com/blog/Building-Project-North-Star)
- [Combine Reality — Project North Star kits](https://combinereality.com/)
- [KGOnTech blog (Karl Guttag)](https://kguttag.com/)
- [MobileRead Forums — Sol Reader e-ink Headset](https://www.mobileread.com/forums/showthread.php?t=351371)
- [Gevo BT Page Turner Ring](https://goodereader.com/blog/technology/gevo-bluetooth-page-turner-ring-will-allow-for-a-hands-free-reading-experience)
- [Genki Wave Ring](https://genkiinstruments.com/)
- [Modos Labs (Caster FPGA driver)](https://github.com/Modos-Labs)
- [Pervasive Displays main site](https://www.pervasivedisplays.com/)
- [Good Display 0.97"](https://www.good-display.com/product/434.html)
