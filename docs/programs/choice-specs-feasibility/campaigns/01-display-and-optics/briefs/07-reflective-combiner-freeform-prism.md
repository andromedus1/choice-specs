---
description: "Reflective combiner and freeform prism alternatives — Project North Star, IMMY dual-mirror, Olympus/Epson freeform"
type: brief
campaign: 01-display-and-optics
program: choice-specs-feasibility
research_method: deep-research
confidence: speculative
status: draft
updated: 2026-04-25
---

# Reflective Combiner / Freeform Prism Alternatives

## Question

Project North Star, IMMY's dual-curved-mirror approach, and Olympus/Epson's freeform prism designs all represent distinct optical paths from pancake/birdbath. The North Star design is open-source and well-documented; the IMMY and freeform-prism approaches are proprietary but academically described. This brief asks: could a shrunk-down monocular reflective combiner work for choice-specs? Are there documented hobbyist replications of IMMY-class dual-mirror approaches? Are freeform prism designs (Olympus, Epson Moverio) buyable or replicable?

## Findings

### Project North Star

**Open-source AR headset from Leap Motion (Ultraleap), wide-FOV, reflective ellipsoidal combiner.**

- **Optics:** ellipsoidal half-silvered mirror combiners + side-mounted 3.5" LCDs at 1600×1440/eye, 120 fps. >100° combined FOV. [Project North Star wiki at vrarwiki](https://vrarwiki.com/wiki/Project_North_Star)
- **Mechanics:** Diamond-turned PMMA (acrylic) reflectors with anti-reflective coating on the outside; AR coating critical to suppress secondary reflections. [Smart Prototyping building blog](https://www.smart-prototyping.com/blog/Building-Project-North-Star)
- **Sourcing:** Smart Prototyping (Hong Kong) sells the v3.1SPC kit — $400–800 for the optics + electronics + driver boards, single-piece combiner. [Smart Prototyping NS Optics Kit](https://www.smart-prototyping.com/Project-North-Star-Electronic-Optics-Kit) [Smart Prototyping single-piece combiner](https://www.smart-prototyping.com/Project-North-Star-Single-Piece-Combiner-Set-v3_1SPC-Bad-Lens-Replacement) [Combine Reality](https://combinereality.com/) is a US reseller stocking complete kits.
- **Community:** Active Discord, ~1.8k GitHub stars, last activity 2025. [GitHub Project North Star](https://github.com/leapmotion/ProjectNorthStar)

**Could a shrunk-down monocular North Star work?** Geometrically, yes. The ellipsoidal combiner is size-independent; you can scale it down. But two structural problems:

1. **The display source is a 3.5" LCD per eye** — that's the fundamental size. Shrinking the combiner without shrinking the display source produces magnified text but not a smaller frame. Replacing the 3.5" LCDs with a smaller microdisplay requires re-deriving the ellipsoidal geometry — significant optical engineering, outside the comfort zone of the open-source design files.
2. **The visor profile is still bulky** — North Star's combiner extends well beyond the eye line. Even a "small" North Star would have an ~50mm×30mm wide visor at the front. **Larger frame footprint than birdbath.** It is not glasses-form-factor; it's headset-form-factor.

**North Star's optical design is overkill for our use case.** Wide FOV (100°+) is a feature we don't want; reading wants narrow FOV (~30°) so the page fills the eye. Reading wants a low-power, slim optical engine; North Star's whole architecture is tuned for spatial / hand-tracking immersion at high FoV.

### IMMY dual-curved-mirror approach

KGOnTech's 2017 article describes IMMY's "different approach" — a system using **two curved mirrors instead of one curved + one flat**. [KGOnTech 2017 — birdbath and IMMY](https://kguttag.com/2017/03/03/near-eye-bird-bath-optics-pros-and-cons-and-immys-different-approach/)

The reasoning: replacing the flat beam splitter with a curved one provides more degrees of freedom in the optical path, theoretically giving better aberration correction at given FoV. IMMY's product was a binocular AR headset for ~2017 industrial use; the company has been quiet since 2020.

**No documented hobbyist replication surfaced.** This brief's web research turned up the original KGOnTech article and a few academic papers on freeform reflective designs but **zero open-source IMMY-class implementations**. The dual-curved-mirror approach is theoretically interesting but practically unbuildable for a hobbyist — you need access to free-form-machined custom mirrors, which is OEM-only.

### Freeform prism (Olympus, Epson Moverio)

Olympus's wedge-shaped prism with three freeform surfaces is documented in academic patents and ResearchGate diagrams. [Freeform wedge prism by Olympus (ResearchGate)](https://www.researchgate.net/figure/a-Freeform-wedge-prism-optical-system-proposed-by-Olympus-25-b-analysis-of-freeform_fig3_354408347) [Google Patents — freeform prism eyepiece with illumination waveguide](https://patents.google.com/patent/US9164351)

**The Olympus design is patent-encumbered**; the prism is custom-cast/machined for the specific application; you cannot buy one off the shelf.

Epson Moverio uses a different approach: **OLED + proprietary waveguide that bounces light through polycarbonate into a prism**. [Epson Moverio BT-45C](https://epson.com/For-Work/Wearables/Smart-Glasses/Moverio-BT-45C-AR-Smart-Glasses/p/V11H970020) **The optical engine is integrated into a complete product, not sold as a component.** Used Moverio BT-300 / BT-40 sell on eBay for ~$150–400 and could be torn down — but the optical engine is glued in and not easily reusable.

**Freeform prism is the proprietary-and-locked-up class of optic.** Beautiful in academic papers, unbuildable for a hobbyist.

### Other reflective / combiner approaches

- **Lumus reflective waveguide** (used in Sigma Lyte) — proprietary, OEM only, no hobbyist channel.
- **DigiLens, Snap Spectacles** — diffractive waveguides; not reflective combiners but adjacent. Closed.
- **Holographic combiners (Microsoft HoloLens, Magic Leap)** — fully proprietary stacks with custom-fabricated holographic photopolymers.

The hobbyist reflective-combiner ecosystem is **almost entirely the Project North Star ecosystem.** The IMMY/Olympus/Epson designs are paper-only or product-only.

### Comparison: this class vs. pancake/birdbath

| Class | Hobbyist availability | Form factor | Best fit for choice-specs |
|---|---|---|---|
| Pancake (Brief 5) | Salvage Quest 3 | Slim | Yes (preferred) |
| Birdbath (Brief 6) | DisplayModule kits | Bulky front | Maybe (if OLED) |
| Reflective combiner (North Star) | Smart Prototyping ~$400+ | Headset-class bulk | No (overkill, wide FoV) |
| Dual-curved-mirror (IMMY) | None | Unknown | No (unbuildable) |
| Freeform prism (Olympus/Epson) | None | Slim (Moverio class) | No (unbuildable) |

### Could we do "slim glasses by salvaging a Moverio"?

A worth-considering alternative path: **buy a used Epson Moverio BT-40 ($200–400) and use its OLED + waveguide as a complete optical engine.** [Epson Moverio BT-40 specs](https://vr-compare.com/headset/epsonmoveriobt-40)

Pros: working optical engine in the right form factor; avoids all custom optics work.

Cons:
- Moverio is a **see-through** waveguide — exactly the architecture choice-specs doesn't need. The transparent path means ambient light enters the eye on top of the page image, which in a dark room is fine but in a half-lit room creates contrast loss.
- Moverio's display is full-color OLED at 720p. Hits the spectrum / persistence concerns from Brief 3.
- The Moverio optical engine is glued/integrated; modifying or repurposing is difficult.
- Moverio is tethered to a controller box via cable; it is not standalone in any meaningful sense without significant electronics rework.

**Moverio salvage is theoretically possible but practically not the right path.** The supply-chain advantage doesn't outweigh the architectural mismatch.

## Recommendation

**Reject this entire optical class for choice-specs. Pancake (Brief 5) and birdbath (Brief 6) cover the buildable design space.**

Specifically:

1. **Project North Star is the wrong tool for the job.** Wide-FOV is a non-feature for reading; the optical engine is bulkier than birdbath; the 3.5" LCDs are wrong for our brightness/persistence/spectrum profile. Don't build North Star and try to shrink it.
2. **IMMY-class dual-curved-mirror is unbuildable as a hobbyist.** No open designs, no buyable components. Skip.
3. **Freeform prism (Olympus, Epson Moverio) is patent-encumbered or product-locked.** Not procurable. Skip.
4. **The North Star ecosystem is still useful as a reference for build process** — Smart Prototyping's documentation, the Discord community, the calibration tooling. Mine for technique even if we don't adopt the optic.
5. **There is a small possibility that a custom slim-form-factor reflective combiner is interesting** — the kind of thing a "Project North Star for slim HUD glasses" (which the scout flagged as a real gap) would represent. **Out of scope for choice-specs feasibility prototype.** If anyone open-sources such a design, revisit.

The headline: **this is a dead-end class for our use case.** Acknowledging it lets us move forward without unmade decisions hanging over the architecture phase.

## Open questions / Gaps

- **Open-source slim reflective combiner** — does not exist; the scout flagged this as a real gap. If someone publishes one in the next 6–12 months, revisit.
- **Custom freeform prism in low quantity** — possibly available from optics shops (Ohara, Schott) at $5k+ per unit. Out of scope.
- **Moverio teardown for community parts harvesting** — has anyone in the Hackaday community successfully extracted a working Moverio optical engine? Not surfaced in search; may exist in Discord / forum threads.
- **IMMY's current product line** — company appears mostly inactive post-2020. Is there continuing IP / ongoing engineering? Not surfaced; assume no.

## Sources

- [Project North Star — GitHub](https://github.com/leapmotion/ProjectNorthStar)
- [Project North Star Docs](https://docs.projectnorthstar.org/)
- [Project North Star Mechanical Guide](https://leapmotion.github.io/ProjectNorthStar/mechanical.html)
- [Project North Star wiki at vrarwiki](https://vrarwiki.com/wiki/Project_North_Star)
- [Smart Prototyping — Project North Star Optical Kit](https://www.smart-prototyping.com/Project-North-Star-Electronic-Optics-Kit)
- [Smart Prototyping — single-piece combiner v3.1SPC](https://www.smart-prototyping.com/Project-North-Star-Single-Piece-Combiner-Set-v3_1SPC-Bad-Lens-Replacement)
- [Smart Prototyping — Project North Star kit (no 3D-printed parts)](https://www.smart-prototyping.com/Project-North-Star-Kit)
- [Smart Prototyping — North Star combiner set v3.1](https://www.smart-prototyping.com/Project-North-Star-Lenses-Single-Piece-Set-v3_1-SPC)
- [Smart Prototyping building Project North Star blog](https://www.smart-prototyping.com/blog/Building-Project-North-Star)
- [Combine Reality — Project North Star headsets and components](https://combinereality.com/)
- [Combine Reality Deck X (docs)](https://docs.projectnorthstar.org/mechanical/combine-reality-deck-x)
- [KGOnTech — birdbath optics + IMMY's approach (2017)](https://kguttag.com/2017/03/03/near-eye-bird-bath-optics-pros-and-cons-and-immys-different-approach/)
- [Olympus freeform wedge prism (ResearchGate diagram)](https://www.researchgate.net/figure/a-Freeform-wedge-prism-optical-system-proposed-by-Olympus-25-b-analysis-of-freeform_fig3_354408347)
- [Google Patents US9164351B2 — freeform-prism eyepiece with illumination waveguide](https://patents.google.com/patent/US9164351)
- [Compact off-axis lightguide prism (ScienceDirect)](https://www.sciencedirect.com/science/article/abs/pii/S0030401817301426)
- [Compact off-axis reflective optical system, freeform mirror + freeform detector (ScienceDirect)](https://www.sciencedirect.com/science/article/pii/S0030401824004127)
- [Epson Moverio BT-45C](https://epson.com/For-Work/Wearables/Smart-Glasses/Moverio-BT-45C-AR-Smart-Glasses/p/V11H970020)
- [Epson Moverio BT-40 specs (vr-compare)](https://vr-compare.com/headset/epsonmoveriobt-40)
- [Portable waveguide display — freeform + volume holograms (PMC)](https://pmc.ncbi.nlm.nih.gov/articles/PMC4394758/)
- [Frontiers — AR Near-Eye Displays Review](https://www.frontiersin.org/journals/virtual-reality/articles/10.3389/frvir.2022.838237/full)
