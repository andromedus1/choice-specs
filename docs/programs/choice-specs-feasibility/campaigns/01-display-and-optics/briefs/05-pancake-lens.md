---
description: "Pancake lens optical design — Sol Reader's chosen path, polarization tricks, sourcing, monocular feasibility"
type: brief
campaign: 01-display-and-optics
program: choice-specs-feasibility
research_method: deep-research
confidence: speculative
status: draft
updated: 2026-04-25
---

# Pancake Lens Design

## Question

Sol Reader chose pancake optics over birdbath/waveguide for its e-ink reading goggles, and Meta validated pancake at consumer scale (Quest Pro, Quest 3) by building the supply chain "from scratch over years." [UploadVR — Meta's pancake supply chain](https://www.uploadvr.com/meta-quest-pro-lenses-supply-chain/) For choice-specs's see-through-not-required, dark-room, monocular reading case, pancake is the scout's leading optical recommendation. This brief asks: how do pancake lenses actually work; what's available off-the-shelf; what's optical efficiency vs. birdbath; and is monocular pancake materially simpler than binocular?

## Findings

### How pancake lenses work

A pancake lens uses a **folded optical path** — light from the display bounces back and forth through the lens stack 2–3 times before reaching the eye, allowing the physical display→eye distance to be roughly half what a single-pass lens requires. Reduced TTL (track length) is the core benefit: this is why VR headsets shrank from Quest 2's brick to Quest 3's slim profile. [Meta blog — pancake optics](https://www.meta.com/blog/vr-display-optics-pancake-lenses-ppd/)

The polarization trick at the core:

1. Display emits light through a **linear polarizer**.
2. Light hits a **half-mirror** in the lens — half passes, half reflects (the loss begins).
3. The transmitted half passes through a **quarter-wave plate (QWP)** which converts linear → circular polarization.
4. Light reflects off a **reflective polarizer** at the back of the lens.
5. On return, it passes the QWP again — now linear, but rotated 90° from the original.
6. The rotated linear polarization passes through the half-mirror's polarization-selective coating and hits the eye.

The full Quest 3 implementation uses two QWPs and a reflective polarizer for the second bounce, total three polarization-selective surfaces. [Meta Quest 3 teardown — Display Training Center](https://displaytrainingcenter.com/2025/11/13/meta-quest-3-teardown-optical-analysis/) [Meta Quest Pro pancake stack](https://www.meta.com/blog/vr-display-optics-pancake-lenses-ppd/)

### Optical efficiency

The half-mirror is the dominant loss source. **Pancake transmits 10–25% of display light** vs. ~50–80% for a Fresnel single-pass. [VRX on Fresnel vs pancake](https://vrx.vr-expert.com/the-difference-between-pancakes-lenses-and-current-fresnel-lenses-found-on-vr-headsets/) [Pimax blog comparison](https://pimax.com/blogs/blogs/aspheric-vs-pancake-vr-lenses-and-why-glass)

More precisely, two well-cited efficiency figures:
- **Maximum theoretical efficiency = 25%** because the half-mirror discards 50% on the first pass and 50% on the second pass; (0.5 × 0.5 = 0.25).
- **If display is unpolarized (e-ink, OLED), additional 50% loss at the input polarizer → 12.5% efficiency.** [OEjournal — breaking the optical efficiency limit of VR with nonreciprocal polarization rotator](https://www.oejournal.org//article/doi/10.29026/oea.2024.230178)
- Modern pancake designs with nonreciprocal polarization rotators (Faraday rotators) can break this 25% ceiling in research; commercial pancake stacks are still ~12.5% for unpolarized sources.

For our bedtime / dark-room case, **12.5% efficiency is fine.** We are deliberately running at <30 nits perceived, and even an e-ink panel with a 100 nit front-light + 12.5% pancake efficiency = 12.5 nits to the eye, which is close to the desired bedtime brightness. **Pancake's "low efficiency" is often a problem only in daylight see-through use; for our case it's effectively a free dimmer.**

### Sol Reader's specific implementation

Per CEO Ben Chelf interviews, Sol Reader uses two 1.3" e-ink panels behind pancake lenses with a magnification ratio that creates a virtual page at "comfortable reading distance" (probably 50cm–optical-infinity range; not specified in interviews). The ergonomic challenge they call out is that "you have to do really neat tricks with the light to get the image to look exactly right and focused when it's only an inch from your eye." [Sol Reader CEO interview at Publishers Weekly](https://www.publishersweekly.com/pw/by-topic/digital/devices/article/98085-can-e-readers-still-innovate-pw-talks-with-sol-reader-ceo-ben-chelf.html)

E-ink is **not natively polarized.** Sol Reader's pancake stack therefore takes a 50% input loss before the half-mirror losses — putting them in the 12.5% efficiency regime. This is fine because they have a front-lit panel with margin. [Liliputing on Sol Reader](https://liliputing.com/sol-readers-e-ink-headset-is-a-wearable-ereader/) The Sol Reader is 100g total and includes pancake optics, e-ink panels, electronics, and battery — which gives a rough envelope: pancake optical assembly weight is in the 5–15g per lens range.

### Off-the-shelf pancake assemblies for hobbyists

This is the thinnest area of the supply chain. Three rough sources:

1. **Salvaged Quest 3 / Quest Pro lens assemblies** — disassembled headsets sell on eBay for $200–400 with intact pancake stacks. The assembly is glued into a holder; extracting cleanly requires solvent and patience. **Most accessible "high quality" pancake source for hobbyists.**
2. **Hackaday "PsuedoPancakes" project** — a hobbyist deconstructed a phone-VR cardboard-class headset and described how to use a stack of two simple lenses + half-mirror + polarizers to approximate pancake behavior. [Hackaday.io PsuedoPancakes](https://hackaday.io/project/187343-the-psuedopancakes) Output quality is far below Meta's, but it's a buildable starting point.
3. **Chinese AR/VR optics suppliers (Alibaba)** — listings exist for "VR pancake lens" assemblies at $30–150/pair. Quality is highly variable; suspected to be Quest 2/3 clones or B-stock. [Alibaba pancake lens listings](https://arvroptical.com/pancake-optical-modules/) Not a confirmed source for production-quality optics but plausible for a $50 prototype.

**No open-source pancake design surfaced.** Sol Reader's design is proprietary; Meta's is a trade secret; academic papers describe the principle but not buildable designs. This is a real hobbyist gap — the kind of place a Project North Star equivalent for slim form factors would fit.

### Monocular vs. binocular pancake

Pancake optics are independent per eye (no optical coupling between left and right). **Monocular pancake is mechanically simpler but optically identical** — same lens stack, same polarization geometry, just one of them. The advantage is:

- **Half the weight** of the optical assembly (5–15g vs. 10–30g).
- **Half the BOM cost.**
- **Eliminates IPD adjustment** — a major mechanical complexity in binocular HMDs.
- **Eliminates inter-pupillary distance calibration** in firmware.
- **Sidesteps stereo content rendering** — choice-specs displays a single page, not a stereo scene; binocular rendering is wasted compute.

The downsides are documented in Brief 8 (vergence-accommodation conflict) and the dichoptic literature: **monocular near-eye displays can cause interocular conflict** — your unviewing eye sees the inside of the frame (or open ambient), creating asymmetric stimulation. [PLOS ONE — interocular conflict from monocular AR](https://journals.plos.org/plosone/article?id=10.1371/journal.pone.0256766) Mitigations: occlude the unviewing eye with an opaque cover (the bedtime case is dark anyway, so this is mild). Reading speed and accuracy in monocular vs binocular show ~2% difference for normal-vision users — small. [Monocular vs binocular reading performance — PubMed](https://pubmed.ncbi.nlm.nih.gov/24612111/)

For choice-specs, **monocular is preferred:**
- Aligns with "personal first" principle (single user, can use dominant eye).
- Aligns with ≤50g principle (half the optical mass).
- Aligns with single-purpose principle (one display, one job).
- Bedtime occlusion of the non-viewing eye is benign or beneficial (the room is dark; the unviewing eye is closed or unstimulated).

### Power and heat

Pancake optics themselves draw zero power. The display behind them does. The pancake stack absorbs / reflects roughly 75–87.5% of display light as heat — but at <30 nits perceived brightness on a 1.3" panel, total absorbed heat is in the milliwatt range. No thermal concern.

### Build complexity

Pancake stacks are **the highest precision-tolerance optic** in any modern HMD. Misalignment between half-mirror and reflective polarizer of >0.5° causes visible ghosting, double-vision, or polarization leakage that looks like haze. Meta's "supply chain from scratch" framing reflects this — there are perhaps a half-dozen vendors worldwide who can manufacture pancake stacks to spec.

For a hobbyist build, this argues strongly for **using a salvaged Quest 3 lens** rather than trying to laminate a custom pancake stack. The salvage path puts a $400 known-good optic in the budget and reduces optical engineering to "design a mount."

## Recommendation

**Pancake is the recommended optical path for choice-specs. Build with a salvaged Quest 3 lens for the prototype; consider custom only if/when productizing.**

Specifically:

1. **Adopt monocular pancake as the prototype optical architecture.** Display source feeds into a single Quest 3-class lens; the display image is presented to the dominant eye; the other eye is occluded by the frame.
2. **Source by salvaging a used Quest 3 from eBay ($200–400).** Disassemble per teardown guides (DisplayTrainingCenter has good photos); extract one lens cluster intact. Discard or retain the second.
3. **Plan for the polarization compatibility of your display source.** E-ink is unpolarized (12.5% efficiency); LCoS is natively polarized (~25% efficiency through the pancake). If we go LCoS in Brief 2's fallback, this is a small efficiency win.
4. **Brightness budget the system at the panel level**, knowing the pancake will eat 75–87.5%. For 30 nits at the eye with an unpolarized e-ink + frontlight, target ~250 nits panel-level — well within E Ink + frontlight capability.
5. **Skip the custom pancake design.** Prototype with salvaged Quest 3 optics; only revisit custom design if Quest 3 lens form factor doesn't fit the frame (which would surprise — Quest 3 lenses are ~25mm thick × ~50mm wide, comparable to a thick eyeglass lens).
6. **Build a quick-look pseudopancake** (Hackaday-style two-lens + half-mirror) as a $30 backup if the Quest 3 lens path falls through. Lower image quality but proves the geometry.
7. **Validate early on a Quest 3 itself.** Sideload an EPUB into a Quest 3 reading app at 30 nits brightness with an "amber tint" filter and dark passthrough — this approximates the pancake-optical experience using the actual Meta supply chain. Fast empirical signal.

The headline: **pancake is the right answer because Sol Reader proved it works for this exact problem and Meta proved the optic is mature.** The hobbyist sourcing constraint pushes us toward salvage, which is fine.

## Open questions / Gaps

- **Quest 3 lens fit in a thin glasses frame.** Quest 3 lenses are sized for a VR-headset eyebox, not a glasses-thin temple. Frame depth budget needs to absorb ~25mm of optical TTL — workable, but makes the frame visibly thicker than Even G1 / Vuzix Z100. Campaign 4 territory.
- **Custom pancake at production volume.** If choice-specs ever scales, custom pancake is the right path (Quest 3 salvage doesn't scale). Suppliers and minimum orders are unknown without engagement.
- **Image quality of salvaged Quest 3 lens at 30-nit dim brightness.** Quest 3 was tuned for ~100–500 nits to the eye; at our brightness range the visible artifacts (ghosting, edge softness) may be more or less prominent. Empirical.
- **Pancake + e-ink front-light interaction.** The front-light's emission angles may not couple efficiently into the pancake stack's polarization geometry. Possible additional loss; bench test needed.
- **Curved pancake for prescription compatibility.** Sol Reader handles prescription via a separate insert; if choice-specs wants integrated prescription, pancake makes that hard. Defer to product phase per principle 7.

## Sources

- [Meta Blog — pancake optics, PPD, infinite display](https://www.meta.com/blog/vr-display-optics-pancake-lenses-ppd/)
- [Meta Quest 3 teardown and optical analysis (Display Training Center)](https://displaytrainingcenter.com/2025/11/13/meta-quest-3-teardown-optical-analysis/)
- [Meta Created The Supply Chain For Quest Pro's Lenses (UploadVR)](https://www.uploadvr.com/meta-quest-pro-lenses-supply-chain/)
- [Mixed-news on Quest 3 Pro lenses](https://mixed-news.com/en/meta-quest-3-comes-with-quest-pro-super-lenses/)
- [Pancake vs Fresnel — VRX](https://vrx.vr-expert.com/the-difference-between-pancakes-lenses-and-current-fresnel-lenses-found-on-vr-headsets/)
- [Aspheric vs Pancake vs Fresnel (Pimax)](https://pimax.com/blogs/blogs/aspheric-vs-pancake-vr-lenses-and-why-glass)
- [Pancake vs Fresnel lenses (Tech Tactician)](https://techtactician.com/fresnel-vs-pancake-lenses-for-vr-which-ones-are-better/)
- [Pancake vs Fresnel — ExpandReality blog](https://blogs.expandreality.io/pancake-lenses-vs-fresnel-lenses-whats-the-difference)
- [Revolutionizing pancake optics with novel design (EurekAlert)](https://www.eurekalert.org/news-releases/1033555)
- [Tripling optical efficiency of pancake optics (MDPI Crystals)](https://www.mdpi.com/2073-4352/15/1/49)
- [Breaking the optical efficiency limit of VR with nonreciprocal polarization rotator (OEjournal)](https://www.oejournal.org//article/doi/10.29026/oea.2024.230178)
- [PsuedoPancakes hobbyist project (Hackaday.io)](https://hackaday.io/project/187343-the-psuedopancakes)
- [Best pancake optical module overview (arvroptical)](https://arvroptical.com/pancake-optical-modules/)
- [Sol Reader CEO interview at Publishers Weekly](https://www.publishersweekly.com/pw/by-topic/digital/devices/article/98085-can-e-readers-still-innovate-pw-talks-with-sol-reader-ceo-ben-chelf.html)
- [Sol Reader e-ink headset (Liliputing)](https://liliputing.com/sol-readers-e-ink-headset-is-a-wearable-ereader/)
- [Monocular vs binocular reading performance (PubMed)](https://pubmed.ncbi.nlm.nih.gov/24612111/)
- [Interocular conflict from a monocular AR display (PLOS ONE)](https://journals.plos.org/plosone/article?id=10.1371/journal.pone.0256766)
