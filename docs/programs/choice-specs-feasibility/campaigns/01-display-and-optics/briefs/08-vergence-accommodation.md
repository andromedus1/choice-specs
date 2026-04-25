---
description: "Vergence-accommodation conflict + eye comfort for 90-minute reading sessions — where should the virtual page sit?"
type: brief
campaign: 01-display-and-optics
program: choice-specs-feasibility
research_method: deep-research
confidence: speculative
status: draft
updated: 2026-04-25
---

# Vergence-Accommodation Conflict + Eye Comfort

## Question

This may be the single most important UX question for choice-specs. Reading is a 90-minute, sustained, near-focal-distance visual task — exactly the regime where vergence-accommodation conflict (VAC) accumulates fatigue. Where should the virtual page sit? Optical infinity (relaxed eye), arm's length (~50cm, natural reading distance), or somewhere else? What does the optometry literature actually say about VAC over long sessions? Does monocular vs. binocular change the calculus? What focal distance do shipping AR glasses present, and how do users describe long-term comfort?

## Findings

### What VAC is, briefly

In normal vision, **vergence** (the rotation angle of each eye to converge on a target) and **accommodation** (the focusing effort of each eye's lens) are linked: looking at something near both converges and accommodates near; looking far does both far. The brain treats them as a single coupled signal.

In a stereoscopic HMD, the **vergence cue** is determined by stereo image disparity (the brain converges to where the simulated object appears in 3D space) but the **accommodation cue** is locked to the physical focal plane of the optic — typically a fixed virtual distance baked in by the lens. When these two diverge, the brain receives conflicting cues. [Wikipedia VAC](https://en.wikipedia.org/wiki/Vergence-accommodation_conflict) [Vergence-accommodation conflicts hinder visual performance and cause visual fatigue (PMC)](https://pmc.ncbi.nlm.nih.gov/articles/PMC2879326/)

In a **monocular** display, there is no stereo cue, so vergence is theoretically unconstrained — the user's eyes converge wherever they naturally do (typically the perceived object distance, set by accommodation and any monocular depth cues). VAC in monocular is a different shape than in binocular.

### Quantitative comfort thresholds

The Banks Lab consensus (multiple papers from 2008–2014, the canonical research):

- **Up to ~0.4 D of VAC is within the comfort zone for most people** for short tasks. [Banks et al — JOV](https://jov.arvojournals.org/article.aspx?articleid=2122611)
- **The "zone of comfort" extends to ~0.6 D for foreground objects (closer than the focal plane) and ~1.0 D for background objects (farther).** [Resolving VAC in HMDs (Kramida 2016)](https://3dvar.com/Kramida2016Resolving.pdf)
- **Beyond ~1.5 D of VAC, fatigue accumulates over minutes.** Headaches, blurred vision, oculomotor disorientation reported.
- **Subjective symptoms (nausea, oculomotor disorder, disorientation) increase significantly after 30 minutes** of exposure to high VAC. [Effect of VAC-induced (Journal of Optometry)](https://www.journalofoptometry.org/en-download-pdf-S1888429624000128)
- **Rate of change of VAC also matters** — rapid swaps between near/far focus are more fatiguing than steady. [Rate of change of VAC affects visual discomfort (PMC)](https://pmc.ncbi.nlm.nih.gov/articles/PMC4294985/)

For a static page in choice-specs, **rate of change is essentially zero** (the page doesn't move during reading). The remaining question is the static accommodation distance.

### Typical fixed-focus HMD focal distances

[VR/AR Wiki on VAC](https://vrarwiki.com/wiki/Vergence-Accommodation_Conflict): "Fixed-focus HMDs typically have internal display screens viewed through lenses that create a virtual image of the screens, making them appear to be located at a fixed focal distance, typically between 1.3 and 2 meters."

This range was chosen by the AR/VR industry deliberately:
- **At 2m (~0.5 D away)**, accommodation is mostly relaxed (close to far-vision rest state).
- **At 1.3m (~0.77 D)**, accommodation is moderately near but still in comfort zone for most users.
- **At 50cm (2.0 D — natural reading distance)**, accommodation is significantly engaged. This is what holding a Kindle does. **It's not pathological — humans read at 50cm for hours every day on books, Kindles, and phones.** But VAC matters here only if the displayed object's vergence cue contradicts.

### What this means for choice-specs

In a **monocular HMD with a single page** (no stereo), the vergence cue is weak — the brain has no left-right disparity forcing convergence to a particular distance. The user's eyes converge based on:

1. **Natural reading habit** — people convergence-accommodate at ~50cm by trained reflex when reading text.
2. **Monocular depth cues** in the page (texture, font size). For a single virtual page filling FoV, these are minimal.
3. **The focal cue** — wherever the optic's focal plane is, accommodation tends there.

**For a monocular static reading task, VAC is a much smaller problem than for binocular VR.** The dominant fatigue source is **prolonged near accommodation** — the same fatigue that hits you reading a paperback or Kindle for an hour, regardless of whether it's a real or virtual page.

### What focal distance shipping AR glasses use

Hard data is thin in product marketing but inferable:

- **Vuzix Z100 / Even G1** — diffractive waveguide HUD class. Per device specs, virtual image at "~5m" or "perceived as far." (No published Diopter value, but the use case is glanceable notifications, not extended reading.) [KGOnTech on Even G1 minimalist design](https://kguttag.com/2024/08/18/even-realities-g1-minimalist-ar-glasses-with-integrated-prescription-lenses/)
- **XReal One Pro** — micro-OLED + birdbath; "virtual screen at 4m, 130-inch equivalent." ~0.25 D, very relaxed accommodation. User reviews uniformly cite low eye strain even on multi-hour binge sessions.
- **Sol Reader** — pancake + e-ink; "comfortable reading distance." From CEO interviews, they target ~50cm equivalent virtual distance. **This puts Sol's eyes in active reading-accommodation, just like a paperback.**
- **Apple Vision Pro** — variable focal plane research-grade implementation; default is ~1.5m. Long-session reviewers (Marques Brownlee, etc.) report eye fatigue at 30–60 min — but this is a 600g headset with VAC + weight + heat compounding.
- **Halliday glasses** — direct projection; the optic moves the focus from "very close to very far (likely near infinity)." [KGOnTech on Halliday](https://kguttag.com/2025/01/25/halliday-arnot-ai-glasses/) Designed for glanceable use, not extended reading.

### The choice for choice-specs

Two reasonable design targets:

**Option A — virtual page at ~2m / 0.5 D (relaxed accommodation).**
- Pros: lowest accommodation effort = lowest accommodative fatigue. Aligns with XReal-class long-session comfort.
- Cons: 2m focal distance feels visually like a "billboard far away," which isn't the natural mental model for reading. Some users describe AR glasses' far focal as "surreal" for extended reading specifically because it conflicts with mental model of a book held at 50cm.

**Option B — virtual page at ~50cm / 2.0 D (natural reading distance).**
- Pros: matches mental model of reading a book; convergence and accommodation are in their normal reading state; user's brain does not have to recalibrate.
- Cons: prolonged near accommodation does fatigue the ciliary muscle (this is the same "tired eyes from reading" we already know from books).

**The empirical literature (Penang/Kookmin display-distance studies) finds:**
- Reading at **near (40cm) for 60 minutes** produces measurable accommodation drift toward near (transient pseudomyopia) but no chronic harm. [Effects of visual display distance on accommodation, head posture (PubMed)](https://pubmed.ncbi.nlm.nih.gov/17915601/)
- Reading at **far (~6m) is more relaxed** and produces less transient accommodation shift.
- Reading at **intermediate (1.5–2m) is best for sustained near work** in the absence of a contraflicker mechanism.

### The monocular angle

Monocular reading is **2.1% slower** than binocular reading in normal-vision subjects but with **8.9% longer fixation duration** and **longer regressive saccades**. [Monocular vs binocular reading performance (PubMed)](https://pubmed.ncbi.nlm.nih.gov/24612111/) That is a small effect. Importantly: **monocular reading has no VAC because there's no stereo cue.** Monocular essentially sidesteps the canonical VAC failure mode.

The monocular fatigue mode is different: **interocular conflict** — the unviewing eye sees ambient (or the inside of a frame), which the brain has to suppress. Over extended sessions this can produce a different kind of fatigue. [PLOS ONE on interocular conflict from monocular AR](https://journals.plos.org/plosone/article?id=10.1371/journal.pone.0256766)

Mitigation: **occlude the unviewing eye** (opaque inner side of frame). For bedtime in a dark room this is benign — the unviewing eye is dark anyway.

### Which eye?

- **Use the dominant eye** for the display. Reading with the non-dominant eye shows mildly impaired metrics (saccade length +0.2 chars). [Monocular vs binocular reading (PubMed)](https://pubmed.ncbi.nlm.nih.gov/24612111/)
- For "personal first" prototype: tune to user's specific dominant eye. Standard tests (sighting test, hole-in-card test) confirm.

### What 90 minutes actually feels like

This is the empirical gap. There are no published studies of 90-minute sustained AR-glasses reading at any focal distance. We can extrapolate from:

- **Kindle / paperback at 50cm for 90 minutes** — well-tolerated by hundreds of millions of readers. The proven baseline.
- **Apple Vision Pro reading at 1.5m for 30 min** — many reviewers report fatigue. Compounds VAC, weight, heat.
- **XReal-class reading at 4m for hours** — anecdotal Reddit reports describe comfortable multi-hour use but caveat dryness/strain.

For a 50g monocular glasses presenting a static page, the most pessimistic model is that fatigue ≈ Kindle baseline (the same near-accommodation fatigue), with a small overlay of monocular interocular adaptation in the first 5–10 minutes. This is **well within "tolerable for nightly bedtime use."**

## Recommendation

**Set the virtual page focal distance at ~50cm (2.0 D) — match natural reading distance. Use monocular display on the dominant eye. Occlude the unviewing eye in the frame.**

Reasoning:
- Monocular sidesteps stereoscopic VAC entirely. No conflict to resolve.
- Matching natural reading distance (50cm) keeps the user's mental model intact and the accommodation behavior identical to reading a Kindle.
- Far-focus alternatives (1.5–2m) reduce accommodative effort but break the mental model and have not been validated for extended reading specifically.
- Hundreds of millions of readers tolerate 50cm reading for hours daily on books — the baseline empirical comfort case is overwhelmingly positive.

Specifically:

1. **Design the optical engine for ~50cm virtual focal distance.** This is a lens-prescription choice in the pancake stack — set the magnification ratio so the page appears at arm's length.
2. **Use the dominant eye.** Test the user (sighting / hole-in-card) before fitting. For "personal first," this is one user.
3. **Occlude the unviewing eye** with an opaque inner frame surface. Reduces interocular conflict and ambient distraction.
4. **Build a 60-minute comfort baseline test as the first behavioral validation.** Have the user wear the prototype for 60 minutes reading; record self-reported eye strain at 15-min intervals. The goal is "no worse than Kindle."
5. **If 60-minute comfort is a problem, consider switching to ~2m focal distance** as a swap-in lens option. This is design-time configurable per pancake-lens magnification.
6. **Do NOT pursue a varifocal display** (research-stage tech, expensive, complex, single user is not stereoscopic so this is way over-engineered).
7. **Document the focal distance choice prominently** for downstream campaigns — Campaign 4's lens design depends on it.

The headline: **monocular + 50cm + dominant eye + occluded other eye = the simplest, most-mental-model-aligned, lowest-VAC-risk path.** Almost all the AR-VR-VAC literature is about binocular stereoscopic systems — choice-specs's monocular-static design dodges most of it.

## Open questions / Gaps

- **Empirical 60-minute comfort** at 50cm monocular vs 2m monocular — no published comparison; needs prototype A/B once optics are buildable.
- **Asymmetric ambient adaptation** — the unviewing eye in a dark room with the frame occluded; over 60 minutes, does the adapted dark-eye produce odd binocular sensations when the device is removed and the room is dimly lit? Anecdotal pirate-eye-patch literature suggests yes (mild "uneven brightness" perception for ~30s); benign.
- **Prescription users** — for users with significant astigmatism, a fixed-focal pancake without prescription correction is uncomfortable. Per principle 7, defer to product phase. For personal prototype, the user's prescription strength matters; if user is significantly myopic, may need an inserted prescription corrector (Sol Reader supports this; Quest 3 supports it via Zenni/HonsVR).
- **Very near focal options (~30cm)** — would reduce magnification effort and could feel "more booklike" but pushes accommodation harder. Probably not worth exploring; literature supports 40–50cm as the sweet spot.
- **Effect of head-locked vs world-locked virtual page on VAC.** Choice-specs is head-locked (page follows head). World-locked would require SLAM (out of scope). Head-locked may itself reduce some monocular vergence drift. Unstudied.

## Sources

- [Vergence–accommodation conflicts hinder visual performance and cause visual fatigue (PMC)](https://pmc.ncbi.nlm.nih.gov/articles/PMC2879326/)
- [Banks et al — VAC hinders visual performance, JOV (ARVO)](https://jov.arvojournals.org/article.aspx?articleid=2122611)
- [The Rate of Change of VAC Affects Visual Discomfort (PMC)](https://pmc.ncbi.nlm.nih.gov/articles/PMC4294985/)
- [Rate of change of VAC affects visual discomfort (ScienceDirect)](https://www.sciencedirect.com/science/article/pii/S0042698914002545)
- [Resolving VAC in HMDs — Kramida 2016 PDF](https://3dvar.com/Kramida2016Resolving.pdf)
- [Resolving VAC in HMDs — UMD Kramida review](https://www.cs.umd.edu/sites/default/files/scholarly_papers/Kramidarev.pdf)
- [Effect of VAC-induced visual fatigue (Journal of Optometry)](https://www.journalofoptometry.org/en-download-pdf-S1888429624000128)
- [Wikipedia — Vergence-accommodation conflict](https://en.wikipedia.org/wiki/Vergence%E2%80%93accommodation_conflict)
- [VR/AR Wiki — VAC](https://vrarwiki.com/wiki/Vergence-Accommodation_Conflict)
- [Effects of Visual Display Distance on Accommodation, Head Posture, Vision, Neck Symptoms (PubMed)](https://pubmed.ncbi.nlm.nih.gov/17915601/)
- [Effects of Visual Display Distance on Accommodation, ResearchGate](https://www.researchgate.net/publication/5928611_The_Effects_of_Visual_Display_Distance_on_Eye_Accommodation_Head_Posture_and_Vision_and_Neck_Symptoms)
- [Multifocal displays review and prospect (PhotoniX)](https://photonix.springeropen.com/articles/10.1186/s43074-020-00010-0)
- [AR/VR Displays — Perspectives and Challenges (PMC)](https://pmc.ncbi.nlm.nih.gov/articles/PMC7404571/)
- [Effects of virtual distant viewing technology on nearwork-induced eye changes (PMC)](https://pmc.ncbi.nlm.nih.gov/articles/PMC11179458/)
- [Monocular and binocular reading performance (PubMed)](https://pubmed.ncbi.nlm.nih.gov/24612111/)
- [Interocular conflict from a monocular AR display (PLOS ONE)](https://journals.plos.org/plosone/article?id=10.1371/journal.pone.0256766)
- [Interocular conflict from monocular AR (PMC)](https://pmc.ncbi.nlm.nih.gov/articles/PMC8412327/)
- [The effect of contrast on monocular vs binocular reading (JOV)](https://jov.arvojournals.org/article.aspx?articleid=2121861)
- [KGOnTech on Even G1 design](https://kguttag.com/2024/08/18/even-realities-g1-minimalist-ar-glasses-with-integrated-prescription-lenses/)
- [KGOnTech on Halliday AR glasses focal range](https://kguttag.com/2025/01/25/halliday-arnot-ai-glasses/)
- [Even Realities — Smart Glasses HUD vs Virtual Screen](https://www.evenrealities.com/en-NZ/blogs/buyers-guide/smart-glasses-with-display)
