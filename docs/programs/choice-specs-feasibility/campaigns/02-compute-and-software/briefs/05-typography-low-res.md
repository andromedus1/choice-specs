---
description: "Typography at low resolution — the math says 200×200 is below the comfort threshold. Recommend 2.13\" 250×212 minimum or 2.9\" 296×128. Bookerly-class hinted body font, no hyphenation needed at very-narrow column widths, full-flash refresh between pages."
type: brief
campaign: 02-compute-and-software
program: choice-specs-feasibility
research_method: deep-research
confidence: speculative
status: draft
updated: 2026-04-25
---

# Typography at Low Resolution / Monochrome

## Question

What does an EPUB book actually look like on the candidate displays? Specifically:

a. Optimal font choice for low-DPI e-ink reading.
b. Font size, leading, hyphenation, page-chunking strategies for small-screen EPUB.
c. Anti-aliasing tricks at low refresh rate.
d. Magnification physics — Kindle Paperwhite at 300 PPI vs Sol Reader's 256×256/eye magnified to ~300 PPI virtual. Are users complaining about Sol's resolution because magnification hurts perceived sharpness, or because the underlying panel has too few pixels?
e. Empirical references — what does Sol-class 256×256 + magnification actually look like in photos / videos?

This brief informs **two decisions**: which Pervasive Displays panel size to actually buy from Campaign 1's options (1.54" 200×200, 2.13" 250×122, 2.9" 296×128, 4.2" 400×300), and what typography defaults choice-specs's KOReader build should ship with.

## Findings

### Resolution baseline: how many pixels does a body font need to be legible?

**Hard floor for body text:** 8 px tall lowercase ("x-height ~5 px"). Below that, glyph features collapse and character recognition becomes a guessing game. [Terence Eden's "stupidly small eInk font"](https://shkspr.mobi/blog/2023/07/stupidly-small-eink-font/) experiment finds that **5pt is the absolute floor; 8pt is the practical minimum.** This is Watchy / 1.54" 200×200 territory.

**Comfortable reading body text:** 14–18 px tall. This is what Kindle Paperwhite users actually read at — the device's 300 PPI panel renders 14 px body at ~9 px x-height with crisp anti-aliasing.

**At 200×200 (1.54" panel, 183 DPI):**
- 14 px body text = 14/183 = 0.077" = ~1.95 mm tall — comfortable real-world size.
- A 200-pixel-wide column at 14 px font holds **~25–35 characters per line** (English averages 5 chars/word + space = 6 px-width-units per character at this font size, but real character-width varies).
- Real-world character width at 14 px serif: ~7-8 px each, so 200 px / 8 = **~25 characters per line** — that's ~5 words per line.
- Vertical: 14 px font + 4 px leading = 18 px per line; 200 px / 18 = **~11 lines per page**.
- Total: **~50–55 words per page on 1.54" 200×200**.

For comparison, Kindle Paperwhite at user-default font-size displays ~250 words per page. **A 1.54" 200×200 e-ink page holds 1/5 the text of a Kindle page.** This means 5x the page-turn frequency for the same content — every 6 seconds at average reading speed instead of every 30 seconds.

**At 2.13" 250×122 (118 DPI):**
- 14 px body text = 14/118 = 0.119" = 3 mm tall — slightly larger than Kindle.
- 250-pixel-wide column at 14 px font ≈ **30 characters per line** (~6 words).
- 122-pixel-tall column with 18 px per line = **~6–7 lines per page**.
- Total: **~35–40 words per page**. Worse than 1.54" because the panel is shorter.

**At 2.9" 296×128 (105 DPI):**
- 14 px body text = 0.133" = 3.4 mm — readable but pushing large.
- 296 px wide / 8 px char = **~37 characters per line**.
- 128 px tall / 18 px per line = **~7 lines per page**.
- Total: **~40–45 words per page**.

**At 4.2" 400×300 (104 DPI):**
- 14 px body text at 0.135" = comfortable.
- 400 px / 8 = **~50 characters per line**.
- 300 px / 18 = **~17 lines per page**.
- Total: **~80–90 words per page**. Much closer to Kindle territory.

These numbers are first-order; they ignore margin, font hinting, and CJK use. The pattern is clear: **physical panel size matters much more than panel DPI for words-per-page.** A bigger lower-DPI panel reads better than a small higher-DPI panel for text.

### Magnification physics — does pancake magnification hurt sharpness?

Sol Reader's 256×256 panel, viewed through a pancake lens, produces a virtual image at ~300 PPI as perceived. ([Campaign 1 brief 9](../../01-display-and-optics/briefs/09-sol-reader-postmortem.md)) The "300 PPI virtual" is the angular pixel density at the eye — equivalent to a 300 PPI panel held at 50 cm.

The question: why do users still complain about Sol's resolution if it's "300 PPI virtual"?

**Answer is twofold:**

1. **Pancake lenses and magnification add aberration.** Even good pancake optics (Quest 3 class) introduce chromatic and geometric aberration at the edges of the image, plus polarization losses that reduce contrast. The "300 PPI" is angular pixel density; perceived sharpness is a function of contrast × angular pixel density × aberration. Magnified e-ink loses contrast (panels go from ~12:1 native contrast to ~6–8:1 after pancake polarization losses).

2. **Words per page is the real metric.** A 256×256 panel has 65,536 pixels total. At 14 px body font with 4 px leading, that's **~75–90 words on the panel**. After eyestrain-tolerant margins, ~60–70 words per page. Users complain about resolution but the actual pain point may be "I'm turning pages every 8 seconds" — which feels like a low-resolution experience even if individual glyphs are crisp.

**Implication for choice-specs:** going below Sol Reader's panel size (say, 1.54" 200×200) means **fewer words per page**, even if our "virtual PPI" after pancake matches Sol's. The legibility-at-magnification game is mostly won at the panel-size level, not at the magnification level.

### Font choice for low-DPI e-ink

[Bookerly](https://en.wikipedia.org/wiki/Bookerly) is the gold standard for e-ink body text. Designed by Dalton Maag for Amazon, deployed on every Kindle since the third-gen Paperwhite (2015). Eye-tracking tests show it reads ~2% faster than its predecessor Caecilia with reduced strain ([Fast Company](https://www.fastcompany.com/3049705/the-improved-bookerly-font-is-now-available-for-most-kindles), [GeekWire](https://www.geekwire.com/2015/amazons-new-paperwhite-introduces-exclusive-bookerly-font-and-new-typesetting-engine/)).

**Bookerly's design principles for e-ink — adoptable by choice-specs:**
- Increased leading (line spacing) vs paper-targeted fonts
- Refined stroke weights for grayscale rendering (black-only e-ink struggles with thin strokes; Bookerly's strokes are slightly heavy)
- Letterforms tuned for "balance aesthetic appeal with practical readability on low-resolution e-ink technology"

**Bookerly is proprietary to Amazon.** Choice-specs cannot ship it. Open alternatives that perform similarly on e-ink:

- **Literata** (Google Fonts; designed for Google Play Books). Excellent body font; open-source via SIL OFL. Strong choice.
- **Source Serif** (Adobe; OFL). Good x-height, e-ink-friendly stroke weights.
- **Crimson Pro** (OFL). Less e-ink-tuned but readable.
- **Charis SIL / Gentium** (SIL). Designed for Latin-script reading; OFL.
- **EB Garamond** (OFL). Beautiful but thin strokes — fights e-ink contrast.

For UI/menus (sans-serif): **Inter, Source Sans 3, Noto Sans** all work. KOReader ships with [a curated default set](https://github.com/koreader/koreader/tree/master/fonts) that includes Noto Sans + Source Serif as defaults — adopt those.

**Recommendation for body font default:** **Literata 16 px**. Hand-tune leading to 1.4× line-height. User can override per-book via KOReader settings.

### Hyphenation strategy

[KOReader's crengine ships TeX-style hyphenation](../../02-compute-and-software/briefs/02-epub-renderer.md) for ~50 languages. **The question for choice-specs is whether to enable it.**

At narrow column widths (~25 chars/line on 1.54" panel), hyphenation prevents large gaps in justified text. Without hyphenation, justified text has rivers and ragged spacing.

But: **left-aligned (ragged-right) text at narrow column widths is fine and arguably preferable** — no hyphenation needed, fewer rendering artifacts, easier on the eye. This is what Kindle's "Bookerly" mode defaults to at small font sizes.

**Recommendation:** ship choice-specs in left-aligned mode by default; expose justification + hyphenation in settings as options.

### Page-chunking for small screens

The architectural question: at 50 words per page (1.54" panel), the user pages every 6–8 seconds. Three responses:

1. **Accept the higher page-turn rate.** Make page turns instant (partial-refresh, no flash). User adapts; bedtime reading is comfortable with rapid micro-turns. Counter: 10x more page turns over a 90-min session = 900+ head-tilts. Fatigue concern.

2. **Use a larger panel.** Move from 1.54" 200×200 → 2.9" 296×128 → 4.2" 400×300. Each step doubles words-per-page roughly. **Practical ceiling is set by the optical engine** — the pancake lens magnifies the panel into a virtual image; bigger panel + same magnification = wider FoV; fits up to ~30-40° FoV before needing more aggressive (less efficient) optics.

3. **Render multi-line "pages" with smaller body text.** 10–12 px body text (vs 14 px) doubles words-per-page but pushes against the 8 px legibility floor. Risky.

**Recommendation:** target 2.9" 296×128 or 4.2" 400×300 if frame depth allows; settle for 2.13" or 1.54" if not.

### Anti-aliasing at low refresh

E-ink's typical 16-level grayscale supports anti-aliased glyphs natively (KOReader uses crengine's font hinter; FreeType's grayscale anti-aliasing is enabled by default). **No tricks needed.**

The interaction with refresh modes:
- **Flashing full refresh** (~750 ms): full grayscale, best glyph rendering, full ghost clearance. Use for: book open, font-size change, every N page turns.
- **Non-flashing partial refresh** (~250–400 ms): retain grayscale, light ghost. Use for: page turn within reading session.
- **A2 / fast monochrome refresh** (~125 ms): black/white only, jagged glyphs. **Avoid for reading.** Acceptable for menu navigation if the user prefers speed.

KOReader defaults align with this; we'd just confirm the defaults are right for our panel.

### Even Realities G1 (640×200 monochrome) — what would EPUB look like?

Per [brief 4 (MentraOS)](04-mentraos-evaluation.md), G1's 640×200 panel can hold **~30-40 words per page** at readable font sizes. The display is monochrome green — no grayscale, only on/off pixels — so glyphs are aliased.

**This is below the Sol baseline and below 1.54" e-ink.** For the choice-specs use case, G1's typography ceiling is the worst of the candidates evaluated. The waveguide also positions the display at the upper edge of the user's vision, adding ergonomic strain for prolonged reading.

**Conclusion:** G1 is not a credible EPUB-reading display. Confirms brief 4's recommendation against MentraOS-on-G1 as the build target.

### Sol Reader empirical reference

Per [Sol Reader brief](../../01-display-and-optics/briefs/09-sol-reader-postmortem.md), Sol shipped briefly in 2023 and was discontinued. While production reviews from MobileRead, Good e-Reader, and HN are still available as historical record, we cannot purchase a Sol unit for direct evaluation. The historical user feedback converges on "Apple II / Commodore 64 / microfiche" descriptions. The dominant complaint was **resolution**, with secondary complaints about **eye strain in users with refractive errors**.

The empirical takeaway from the historical record: 256×256 magnified through pancake to 300 PPI virtual is **legible but visibly pixelated**. Users noticed every pixel boundary on individual glyph strokes. Reading was functional but felt low-fi — and it's reasonable to assume this was a contributor to Sol's commercial failure.

To exceed Sol's reported experience, choice-specs needs **either**:
- More native pixels (e.g., 320×240 at 1.54" panel — not in Pervasive Displays' lineup; would require larger panel)
- Larger panel (2.9" 296×128 ≈ 38k pixels vs Sol's 65k/eye but better vertical room) at lower magnification
- Both

**The 2.13" 250×122 = ~30k pixels** is below Sol on raw count. **2.9" 296×128 = ~38k pixels.** **4.2" 400×300 = 120k pixels** — meaningfully exceeds Sol's per-eye count.

**Without a purchasable Sol unit for direct A/B**, the resolution/typography target needs to be set against either (a) the historical Sol reviews as proxy, (b) MentraOS-on-G1 hands-on, or (c) bench-mocked Pervasive Displays panels viewed through pancake at the prototype focal distance. Each is imperfect; bench-mocking is the highest-fidelity option once panels arrive.

## Recommendation

### Minimum acceptable native resolution: **≥ ~40,000 pixels per panel.**

This rules out:
- 1.54" 200×200 (40k pixels — at the floor, not above)
- 2.13" 250×122 (30k pixels — below Sol baseline)

And brings into target:
- 2.9" 296×128 (38k — at floor; marginal)
- 4.2" 400×300 (120k — meaningfully above Sol)
- Custom panel co-developed with E Ink Corp at higher PPI (Phase 2+)

**Buy 2.9" 296×128 as the prototype default. Have a 4.2" 400×300 in the parts pool for the second-build iteration if frame depth allows.** The 1.54" panel from Campaign 1's sourcing brief is fine for **bench bringup** (validating SPI driver, Caster integration, framebuffer write loop) but should not be the worn-prototype display.

### Typography defaults

- **Body font:** Literata 14–16 px (default 16 for our small displays).
- **Line height:** 1.4–1.5× font-size (≥4 px leading minimum).
- **Alignment:** left-aligned, ragged right. Justification + hyphenation available in settings.
- **Margins:** 4-6 px outer; 8 px on G1-class displays for safety.
- **Anti-aliasing:** crengine default (FreeType grayscale).
- **Anti-flicker:** non-flashing partial refresh on page turn; full flashing refresh every 6th page turn (KOReader's `EpubFlashingPages = 6` setting).
- **Hyphenation:** off by default at narrow column widths; on if user enables justified mode.

### Fallback typography strategy if we end up with sub-300×300 panel

If sourcing forces us onto a 200×200 panel (1.54" Pervasive Displays from the sourcing brief, $15 single unit):

- Drop body font to 12 px.
- Tighten leading to 1.3×.
- Margin to 2 px.
- Net: ~70 words per page.
- Accept 5× the page-turn frequency. Make page-turn input as low-friction as possible (head-tilt with low-effort threshold; ring single-press).
- Treat as v1; plan v2 around a larger panel.

**This fallback is acceptable for personal feasibility validation but not for "feels like a Kindle."**

### Display tier ranking for choice-specs reading experience (best to worst)

1. Custom co-engineered E Ink panel at higher PPI (300+ DPI native) — Sol Reader's path; out of personal-prototype scope.
2. **Pervasive Displays 4.2" 400×300** — best of the realistic options.
3. **Pervasive Displays 2.9" 296×128** — second; smaller frame depth; recommended default.
4. **Sol Reader (1.3" 256×256 ×2 binocular)** — historical reference (now discontinued; not procurable).
5. Pervasive Displays 1.54" 200×200 — bench bringup only.
6. Pervasive Displays 2.13" 250×122 — too short vertically; below Sol pixel count.
7. Even Realities G1 (640×200 monochrome microLED) — below Sol; not e-ink, no grayscale.
8. Vuzix Z100 (640×480 monochrome microLED) — typography not under our control via MentraOS.

## Open questions / Gaps

1. **Empirical viewing test.** None of these panels has been read on by the user yet. With Sol Reader discontinued, the empirical baseline shifts to bench-mocked Pervasive Displays panels (Day-30 sourcing brief delivery) viewed through pancake optics at prototype focal distance. **All other findings remain speculative until that test happens.**
2. **Pancake magnification's effect on perceived contrast** at our specific panel + lens combination. Brief 5 of Campaign 1 noted 12.5% transmission for the pancake; we don't yet know what that does to text readability empirically. Bench test required.
3. **Subjective threshold for "feels like a Kindle".** Words-per-page is a necessary metric but not sufficient. Glyph crispness, scrolling vs paging UX, refresh artifact tolerance — all matter. Bench test once Pervasive Displays panels arrive; supplement with Kindle Paperwhite and (if MentraOS-on-G1 is purchased) G1 hands-on for comparison.
4. **Custom panel option.** [Campaign 1 brief 1](../../01-display-and-optics/briefs/01-eink-microdisplay-landscape.md) ruled out direct E Ink Corp engagement at $50–250k NRE. **At product phase, revisit.** Even a modest 1.0–1.3" 384×384 custom panel would dramatically improve typography vs the off-the-shelf options.
5. **E Ink "Carta 1300" availability in small sizes.** Newer E Ink generation has higher native contrast. Pervasive Displays' Carta 1300 lineup may include sizes/resolutions not in this brief. Worth checking distributor catalog.
6. **OLED amber-tinted typography** if we end up on the OLED fallback. OLED at 1024×768 mono-amber would have very different typography characteristics — better resolution, worse contrast in dark conditions. Punt to Campaign 1 brief 3.

## Sources

- [Bookerly typeface (Wikipedia)](https://en.wikipedia.org/wiki/Bookerly)
- [Bookerly font announcement (Fast Company)](https://www.fastcompany.com/3049705/the-improved-bookerly-font-is-now-available-for-most-kindles)
- [Kindle Paperwhite + Bookerly (GeekWire)](https://www.geekwire.com/2015/amazons-new-paperwhite-introduces-exclusive-bookerly-font-and-new-typesetting-engine/)
- [Best fonts for ebooks (EditionGuard)](https://www.editionguard.com/learn/best-fonts-e-books/)
- [Will there be increased resolution for e-ink Kindles (Good e-Reader)](https://goodereader.com/blog/kindle/will-there-be-increased-resolution-for-e-ink-kindles)
- [Stupidly small eInk font experiment (Terence Eden)](https://shkspr.mobi/blog/2023/07/stupidly-small-eink-font/)
- [E Ink ghosting decoded (Viwoods)](https://viwoods.com/blogs/paper-tablet/e-ink-ghosting-explained)
- [E-Ink refresh modes explained (Viwoods)](https://viwoods.com/blogs/paper-tablet/viwoods-e-ink-refresh-modes-explained)
- [Why does e-ink refresh slowly (Orient Display)](https://www.orientdisplay.com/why-does-e-ink-refresh-slowly/)
- [Understanding E Ink Technology (BOOX Help Center)](https://help.boox.com/hc/en-us/articles/360027486972-Understanding-E-Ink-Technology-in-BOOX-Devices)
- [E-paper basics: update modes (Hackaday.io)](https://hackaday.io/project/21551-paperino/log/59392-e-paper-basics-1-update-modes)
- [Why does an electronic paper display flicker (Visionect)](https://www.visionect.com/blog/why-epaper-blinks/)
- [Sol Reader review — resolution complaints (Good e-Reader)](https://goodereader.com/blog/tablet-slates/sol-reader-your-own-private-reading-aid-a-review)
- [Sol Reader on Hacker News](https://news.ycombinator.com/item?id=37955244)
- [256x256 display guide (Alibaba)](https://electronics.alibaba.com/buyingguides/256x256-display-guide-when-it-matters-when-it-doesn%E2%80%99t)
- [Pervasive Displays 1.54" 200×200 panel (Pi Hut)](https://thepihut.com/products/eink-display-module-spi-1-54-200x200)
- [Crystalfontz 1.54" 200×200 ePaper](https://www.crystalfontz.com/product/cfap200200a30154-200x200-square-epaper-display)
- [Pervasive Displays product page](https://www.pervasivedisplays.com/product/e-ink-displays-etc/)
- [Literata font (Google Fonts)](https://fonts.google.com/specimen/Literata)
- [Source Serif font (Google Fonts)](https://fonts.google.com/specimen/Source+Serif+4)
- [crengine epub.css source](https://github.com/koreader/crengine/blob/master/cr3gui/data/epub.css)
- [KOReader fonts directory](https://github.com/koreader/koreader/tree/master/fonts)
