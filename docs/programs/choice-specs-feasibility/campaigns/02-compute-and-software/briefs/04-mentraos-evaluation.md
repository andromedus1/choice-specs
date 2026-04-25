---
description: "MentraOS evaluation as 'skip hardware' path — REJECTED for choice-specs as primary; useful as a quick UX-validation prototype only. Display-and-IMU primitives are insufficient for a real EPUB reader, and the phone-tether requirement violates principle 1."
type: brief
campaign: 02-compute-and-software
program: choice-specs-feasibility
research_method: deep-research
confidence: speculative
status: draft
updated: 2026-04-25
---

# MentraOS Deep Evaluation

## Question

This is the most important brief in Campaign 2. The thesis is: **if MentraOS-on-existing-hardware can render EPUB and accept head-tilt input on a Vuzix Z100 or Even Realities G1, choice-specs may be a software project, not a hardware project.** That would change the entire roadmap — Campaign 4 (industrial design / power / safety) becomes mostly moot, the build cost drops from ~$2,000 to ~$600 (one pair of glasses), and the timeline drops from months to weeks.

We have to evaluate this honestly because the upside if it works is enormous, and we should not let "we want to build hardware" bias us against a software-only path that might genuinely satisfy the use case.

The specific questions to answer:

a. Can a MentraOS app on Vuzix Z100 or Even Realities G1 actually render EPUB pages?
b. Can the SDK access the IMU directly for head-tilt? Latency?
c. Can we pair a BT clicker through the MentraOS layer, or only through the phone?
d. What's the dev cycle?
e. What's the locked-in cost (vendor dependency)?
f. What's the standalone-ness — does it require phone tether? Does that violate principle 1?

## Findings

### MentraOS overview

[MentraOS](https://github.com/Mentra-Community/MentraOS) is the leading open-source smart-glasses platform per its own marketing — and the framing is plausibly correct. MIT-licensed; TypeScript SDK; backed by Mentra (Mentra-Community organization on GitHub, formerly AugmentOS, $8M Series A circa 2025). Three currently-supported glasses:

- **Mentra Live** — Mentra's own camera glasses
- **Even Realities G1** — 640×200 monochrome green microLED waveguide, ~43g, $599
- **Vuzix Z100** — 640×480 monochrome green microLED waveguide, 38g, $1,000-class

[Glasses-compatibility.md](https://github.com/Mentra-Community/MentraOS/blob/main/glasses-compatibility.md) is the canonical capability matrix. The relevant rows (per direct fetch):

| Capability | Mentra Live | Even Realities G1 | Vuzix Z100 |
|---|---|---|---|
| Text display | ❌ | ✅ | ✅ |
| Image display (bitmap) | ❌ | ✅ | ❌ |

**This is a binding finding.** We need to render an EPUB page — which means rendering rasterized text glyphs into a bitmap and pushing that bitmap to the display. Text-as-strings ("show this string") is the simple text path; bitmap rendering is the only way to get controlled typography (fonts, line spacing, hyphenation) onto the display.

### Display: what each device can actually render

#### Even Realities G1
- Resolution: **640×200 monochrome green** at **20 Hz refresh** ([minimicroled.com](https://www.minimicroled.com/even-realities-g1-ar-glasses-micro-led-waveguide-tech/), [arcompare.com](https://www.arcompare.com/products/even-realities-g1)).
- Bitmap support: yes, via MentraOS [LayoutManager / showBitmapView](https://docs.mentra.glass/reference/managers/layout-manager/) ([cloud-docs.mentra.glass](https://cloud-docs.mentra.glass/sdk/display-layouts)). Example dimensions in docs cite `width: 400, height: 240` — suggesting bitmaps up to roughly the panel resolution.
- **Display update rate constraint: ~200–300 ms minimum between updates per the Layout Manager documentation, with rapid updates queued or dropped.** This is fast enough for a page-turn but rules out scrolling animations.

The G1 is built around HUD-class messaging — short notifications, glanceable info, teleprompter strips. Even Realities reviews ([Engadget](https://www.engadget.com/wearables/even-realities-g1-review-limited-but-effective-smart-glasses-140059586.html), [Tom's Guide](https://www.tomsguide.com/computing/smart-glasses/even-realities-g1-smart-glasses-review)) describe the experience as "easy to read short text, doesn't obstruct view." The teleprompter is the closest existing use case to choice-specs and shows 4–8 lines of text per frame, manually paged.

A 640×200 native page rendering: at typical Kindle font size (~16-18 px tall body text on real e-reader), 640×200 fits **~1 paragraph** — about 4–5 lines of text per "page." That's roughly **30–40 words per page**. A standard novel page on a Kindle is ~250 words. **Choice-specs reading on G1 would mean turning pages 5–8× more often than on a Kindle.** Not unusable, but a meaningful UX degradation.

The waveguide projects only on the upper portion of the lens ("only takes up a small portion at the top of the lenses" per Engadget). The display is positioned above the natural reading sightline — fine for HUD glances, less ideal for prolonged reading where the user wants the content centered.

#### Vuzix Z100
- Resolution: **640×480 monochrome green** at 4:3, **30° FOV**, right eye, weighing 38g, 48-hour battery ([Vuzix Z100 product page](https://www.vuzix.com/products/z100-smart-glasses)).
- **Bitmap display in MentraOS: NOT SUPPORTED.** Per `glasses-compatibility.md`, Z100 supports text display only.
- This means: any EPUB-reading scenario on Z100 must be done via text-string layouts only. We cannot render a typeset page bitmap. We're stuck with whatever text-rendering the device firmware does internally.

The Z100's text-only restriction in MentraOS is **binding**: we cannot ship a Kindle-class typography experience because we don't control the text rendering. We're at the mercy of Vuzix's font and layout choices.

#### Mentra Live
Camera-glasses class device. Per `glasses-compatibility.md`, no display capabilities at all — the device is for capturing video, not displaying content. **Out for this use case.**

### EPUB rendering feasibility

Honest answer: **no, MentraOS does not provide a credible EPUB reading experience on either G1 or Z100 today.**

For G1 (the better-equipped of the two):
- We'd render EPUB pages on the cloud or phone, push 640×200 bitmaps to the glasses every page-turn.
- 30-40 words per page is a category-different reading experience from a Kindle's ~250 wpp.
- Ghosting/persistence issues are unknown — microLED waveguide doesn't ghost like e-ink, but **20 Hz refresh + 200-300 ms update latency is borderline**.
- The display is biased to the upper visual field; choice-specs's principle is centered on ergonomic single-page-in-front-of-eye.

For Z100:
- Bitmap not supported. Text-only display via MentraOS APIs.
- We could implement a "text strings paged" UI but typography is controlled by the device firmware, not by us.
- This violates principle 3 (eye-comfort beats eye-candy) and the Sol-baseline test — we cannot deliver a controlled typography experience.

**Neither path produces an EPUB reading experience comparable to a Kindle**, let alone the Sol Reader baseline.

### IMU access

MentraOS documentation consistently mentions "sensor access" as a SDK capability ([mentraglass.com/os](https://mentraglass.com/os): "display control, sensor access, AI pipelines"). However, the public docs don't surface concrete IMU APIs at the level of "subscribe to accelerometer events at 50 Hz with quaternion output" — searches for accelerometer / IMU specifically return mostly the marketing claim, not API references.

**This is a real research gap, not a known capability.** We'd have to:
1. Read the MentraOS SDK source on GitHub directly to see what sensor events are actually exposed.
2. Verify the latency of those events (is the IMU sampled on-glasses, transmitted via BLE to the phone, then to the cloud, then to our app server? Each hop adds latency).
3. Verify whether head-tilt detection has the responsiveness for page-turn UX.

**Architectural concern:** MentraOS's [three-tier architecture](https://github.com/Mentra-Community/MentraOS/blob/main/AGENTS.md) is **glasses ↔ BLE ↔ phone ↔ HTTP ↔ cloud ↔ HTTPS ↔ app server**. The MentraOS SDK runs in the **app server** layer (TypeScript on the developer's backend). If IMU events have to traverse glasses→phone→cloud→app server before our code sees them, the round-trip latency is plausibly 200–500 ms minimum. **For a head-tilt page-turn, that latency budget is already at the edge of "feels responsive."**

If MentraOS allows phone-side or glasses-side code execution (a "MiniApp" running locally), latency improves. The docs hint at MiniApps but the capability surface is unclear.

### BT clicker pairing

There is no documented MentraOS API for "pair an arbitrary BT HID device through the glasses" or "pair through the phone." The model is: glasses ↔ phone ↔ cloud ↔ app server. Adding an external BT input device into this loop would require the input device to pair to the **phone** (since the glasses' BT is already occupied talking to the phone), then the phone-side MentraOS app forwards key events to the app server, then the app server tells the glasses what to display.

**This is workable but adds another network hop.** The phone is now load-bearing for input — battery, Bluetooth, internet connectivity all matter for whether the page turns.

### Dev cycle

The MentraOS SDK ([@mentra/sdk on npm](https://www.npmjs.com/package/@mentra/sdk)) is TypeScript. Workflow:

1. Write a TypeScript MentraOS app — handles session events, sends layouts to the LayoutManager.
2. Deploy it to **your own server** (Mentra Cloud is the intermediary; the app code runs on a server you control).
3. User opens MentraOS phone app, registers your app's URL, connects glasses.
4. Glasses + phone + cloud + your server form the runtime loop.

**This is web-app development, not embedded development.** Iteration is fast (hot-reload TypeScript on your laptop). But the architecture means **the user's reading session depends on phone connectivity and cloud uptime** — every page-turn round-trips through MentraOS Cloud and back to your server.

[GitHub example apps](https://github.com/Mentra-Community/MentraOS-Display-Example-App) confirm this architecture.

### Vendor lock / cost

- Vuzix Z100 dev edition: ~$700–1,000 ([unboundxr.com listing](https://unboundxr.com/vuzix-z100-smart-glasses-developer-s-edition))
- Even Realities G1: $599 ([evenrealities.com](https://www.evenrealities.com/g1))

Both cheaper than building custom hardware (~$2k per Campaign 1 sourcing), but **non-trivially priced**.

Vendor lock:
- MentraOS itself is MIT-licensed open source. **No vendor lock at the SDK layer.**
- The **glasses are vendor hardware** — Even Realities and Vuzix control firmware, and only support what they've chosen to support (e.g., Z100 doesn't do bitmaps even though MentraOS as a platform could in principle).
- If Even Realities or Vuzix EOLs the device or breaks an API, we have no recourse beyond MentraOS's own engineering response.
- **Mentra's company stability matters.** $8M Series A 2025; small team; not yet revenue-positive. Five-year horizon for Mentra's continued existence is not guaranteed.

### Standalone-ness — does it violate principle 1?

This is the hardest finding. **MentraOS apps require a phone running the Mentra app, paired by BLE to the glasses, with internet access for the cloud-app-server loop.** ([per the architecture document](https://github.com/Mentra-Community/MentraOS/blob/main/AGENTS.md))

Choice-specs principle 1 (single purpose, no compromise) and the Out-of-Scope explicit "no phone tethering" together mean **MentraOS as currently architected violates a core north-star principle**. Reading is supposed to work without a phone present.

There may be a path where:
- The user pairs the glasses to the phone once at setup.
- During reading, the phone sits charging on the nightstand running the Mentra app.
- Once paired, BLE communication continues during the reading session.

This is "phone-tethered" by the strict reading of the principle. It also adds failure modes: phone Bluetooth flakes, phone battery dies, phone OS update breaks the Mentra app, phone Wi-Fi drops disrupting the cloud loop.

**The principle is in tension with the MentraOS architecture, full stop.** A choice-specs-on-MentraOS prototype would have to either:
- Accept the violation as a v1 compromise to validate UX before building hardware.
- Engineer a "phone disconnects fully during reading" mode — which means rendering pages locally on the phone or pre-caching on the glasses; not the MentraOS happy path.

### Summary of MentraOS feasibility

| Question | Answer |
|---|---|
| Can a MentraOS app render EPUB pages on Z100? | **No** — bitmap not supported on Z100; text-only without typography control. |
| Can a MentraOS app render EPUB pages on G1? | **Marginal** — bitmap supported but 640×200 limits to ~30-40 words per page; 200-300 ms refresh latency; display biased to upper field. |
| Can the SDK access IMU directly with low latency? | **Unclear** — sensor access claimed; IMU API not documented in surfaced sources; latency through glasses↔phone↔cloud↔app server is a serious concern (~200–500 ms estimated). |
| Can we pair a BT clicker through MentraOS? | **Indirect** — would pair to phone, phone forwards events. Adds dependency on phone. |
| Dev cycle? | **Fast** — TypeScript, web-app workflow, hot-reload. |
| Vendor lock? | **Moderate** — SDK is MIT; hardware is vendor-controlled; Mentra company stability is a 5-year question. |
| Phone tether requirement? | **Yes** — violates principle 1. |

## Recommendation

### **Reject MentraOS as the primary "skip hardware" path. Custom-build is recommended primary.**

Three independent reasons converge:

1. **The display capability is below Sol Reader's already-criticized baseline.** Sol's 256×256/eye produced ~125 wpp magnified (per the typography-words-per-page math in [brief 5](05-typography-low-res.md)); G1's 640×200 (with no magnification — the waveguide projects directly into the eye at native resolution) produces ~30-40 wpp on screen. **A choice-specs-on-G1 prototype would have a worse reading experience than (the now-discontinued) Sol** on the dominant axis (resolution + words-per-page). For a project whose principle 3 is "eye-comfort beats eye-candy" and whose primary historical reading reference is Sol, **going below Sol's bar contradicts the principle**.
2. **Phone tether violates principle 1 and the explicit "no phone tethering" out-of-scope item.** The MentraOS architecture is fundamentally cloud-first / phone-mediated. We cannot make a MentraOS app standalone without re-architecting around their model. The principle was made for exactly this kind of decision: the device should not depend on a phone working at 2 a.m.
3. **IMU access and BT-clicker pairing are uncertain at best, latency-bound at worst.** The 200-500 ms estimated round-trip for events through MentraOS Cloud is at the edge of "feels responsive" for page-turn — and we'd be inheriting that uncertainty into the input pipeline that Campaign 3 owns.

The custom-build path on i.MX6 + KOReader (per [briefs 1–3](01-soc-selection.md)) gives us:
- Sol-baseline display quality with the right e-ink panel
- Standalone operation (principle 1 satisfied)
- Direct IMU access (50–100 Hz, ~5 ms latency to user code)
- Direct BT HID pairing
- Validated typography stack

### **Use MentraOS-on-G1 as a $599 UX-validation toy, not as the build target.**

Buy a G1, write a 200-line MentraOS app that displays EPUB-derived text strings (4-line snippets), pair a BT ring, validate the *interaction model* (head-tilt for page turns, ring fallback) before committing to the hardware build. **Treat it as a usability prototype, not as the deliverable.**

With Sol Reader discontinued (per [Campaign 1 brief 9](../../01-display-and-optics/briefs/09-sol-reader-postmortem.md)), no shipping head-worn EPUB device exists for empirical baseline; MentraOS-on-G1 is the closest available shipping device for UX validation, even though its display is below Sol's already-criticized resolution. **For UX validation only, MentraOS on G1 is potentially valuable.** For shipping the actual choice-specs experience, it doesn't qualify.

### **When MentraOS becomes preferable**

- If MentraOS or its successor adds a **standalone runtime mode** (app code runs on glasses without phone or cloud), some of the principle-1 conflict resolves.
- If a future vendor ships a **higher-resolution monochrome display in glasses form factor at <50g** — say, 1280×720 monochrome — and exposes bitmap rendering through MentraOS, the EPUB story improves enough to warrant re-evaluation.
- If our custom-build path proves unworkable in some specific way (sourcing collapse, mass overshoot, thermal failure), MentraOS becomes the fallback that validates "the use case works at all" while we redesign.

In current state, none of these conditions are met. Custom build is the answer.

## Open questions / Gaps

1. **IMU API surface.** We did not directly read the MentraOS GitHub TypeScript SDK source for sensor APIs. **Required follow-up:** clone the repo, look at the SDK's sensor hooks, confirm or deny IMU access, measure latency empirically with a small test app.
2. **MiniApp / on-device runtime.** Mentra's docs hint at apps running on phone or glasses directly without cloud. Capability surface is unclear from public docs. **Required follow-up:** read SDK source and developer Discord.
3. **Even Realities G2 — released 2025-11.** [9to5google](https://9to5google.com/2025/11/12/even-realities-launches-g2-smart-glasses-r1-control-ring/) reports a G2 with "spatial display" and an R1 controller ring; [Mentra blog](https://mentraglass.com/blogs/blog/even-realities-g2-supported-on-mentraos) confirms MentraOS support. The R1 ring is interesting — first-party BLE input designed for this glasses class. **G2 capability matrix needs another look** before fully committing to custom-build. Worth ~30 minutes of follow-up if budget allows.
4. **Even Realities Hub EPUB app.** [Android Authority](https://www.androidauthority.com/even-realities-g2-smart-glasses-apps-3654769/) reports an Even Realities-developed EPUB reader for G2 in their app store. **First-party validation** of the use case at low resolution. Worth empirical eval if we end up needing the MentraOS fallback.
5. **MentraOS Cloud uptime / latency.** No published SLAs or latency numbers. We didn't measure. **For UX validation use, this isn't critical; for shipping, it would be a blocker.**
6. **G1's ergonomics for prolonged reading.** Reviews note the display "doesn't obstruct view" but tend to come from short-session use. Whether 60-min reading at the upper-visual-field display position is comfortable is empirically untested. Buy and try.
7. **Locked-in cost trajectory.** If Mentra raises Series B, the platform becomes more durable; if not, MentraOS could vanish. Choice-specs's 5-year horizon should not depend on it.

## Sources

- [MentraOS GitHub](https://github.com/Mentra-Community/MentraOS)
- [MentraOS glasses-compatibility.md](https://github.com/Mentra-Community/MentraOS/blob/main/glasses-compatibility.md) — **the binding capability matrix**
- [MentraOS AGENTS.md (architecture)](https://github.com/Mentra-Community/MentraOS/blob/main/AGENTS.md)
- [MentraOS docs / Welcome](https://docs.mentraglass.com)
- [MentraOS Quickstart](https://docs.mentraglass.com/app-devs/getting-started/quickstart)
- [MentraOS Cloud SDK Integration](https://cloud-docs.mentra.glass/cloud-overview/sdk-integration)
- [MentraOS LayoutManager reference](https://docs.mentra.glass/reference/managers/layout-manager/)
- [MentraOS Cloud Display Layouts](https://cloud-docs.mentra.glass/sdk/display-layouts)
- [@mentra/sdk on npm](https://www.npmjs.com/package/@mentra/sdk)
- [MentraOS Display Example App](https://github.com/Mentra-Community/MentraOS-Display-Example-App)
- [MentraOS Extended Example App](https://github.com/Mentra-Community/MentraOS-Extended-Example-App)
- [Mentra OS marketing page](https://mentraglass.com/os)
- [Mentra Live page](https://mentraglass.com/live)
- [Even Realities G1 product](https://www.evenrealities.com/g1)
- [G1 Engadget review](https://www.engadget.com/wearables/even-realities-g1-review-limited-but-effective-smart-glasses-140059586.html)
- [G1 Tom's Guide review](https://www.tomsguide.com/computing/smart-glasses/even-realities-g1-smart-glasses-review)
- [G1 KGOnTech writeup](https://kguttag.com/2024/08/18/even-realities-g1-minimalist-ar-glasses-with-integrated-prescription-lenses/)
- [G1 microLED waveguide details](https://www.minimicroled.com/even-realities-g1-ar-glasses-micro-led-waveguide-tech/)
- [G1 AR Compare](https://www.arcompare.com/products/even-realities-g1)
- [Even Realities G2 launch](https://9to5google.com/2025/11/12/even-realities-launches-g2-smart-glasses-r1-control-ring/)
- [Even Realities G2 + MentraOS support](https://mentraglass.com/blogs/blog/even-realities-g2-supported-on-mentraos)
- [G2 EPUB and chess apps — Android Authority](https://www.androidauthority.com/even-realities-g2-smart-glasses-apps-3654769/)
- [Vuzix Z100 product page](https://www.vuzix.com/products/z100-smart-glasses)
- [Vuzix Z100 dev edition](https://unboundxr.com/vuzix-z100-smart-glasses-developer-s-edition)
- [Vuzix Z100 hands-on](https://precisionopticspro.com/hands-on-with-vuzix-z100/)
- [Mentra Nightscout viewer (G1B reference impl)](https://github.com/DanielGTdiabetes/mentra-nightscout)
- [my-examples-and-posts-of-g1-even-realities-smart-glasses](https://github.com/hpssjellis/my-examples-and-posts-of-g1-even-realities-smart-glasses)
