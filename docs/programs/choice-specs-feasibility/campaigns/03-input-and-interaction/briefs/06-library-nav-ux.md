---
description: "Complete library navigation UX — three-screen design, head-tilt + ring traversal, recents-first list scaling to 1000 books"
type: brief
campaign: 03-input-and-interaction
program: choice-specs-feasibility
research_method: deep-research
confidence: speculative
status: draft
updated: 2026-04-25
---

# Library Navigation UX (Screens + Input Together)

## Question

This brief owns the entire library experience — both the *screens* (how books render in a head-locked, low-resolution monochrome panel) and the *input* (how the user traverses, selects, and exits without a keyboard).

Constraints:
- **Display:** monochrome e-ink, 200×200 to 400×400 native (per C1: 1.54" → 2.13" → 2.9" → 4.2" Pervasive Displays panels), magnified through pancake. Effective virtual size: a small "monitor" 50 cm in front of the dominant eye.
- **Input:** head-tilt gestures (4 gestures from C3 brief 3) + optional BT ring (3 buttons).
- **No voice. No held device. No eye tracking. Hands stay under the blankets.**
- **Library size:** v1 = personal library, up to 100 books. Scaling to 1000 books is the hard case worth designing through.

The screens must be **head-locked** (anchored to the user's vision regardless of head pose). Library navigation flows must be **traversable with 4 gestures** (or 3 ring-buttons + 4 gestures if both are available). Failure mode: user gets stuck on a wrong screen and can't escape without taking the device off.

The principle "single purpose, no compromise" forces minimalism. The principle "sleep-grade reliability" forces predictability — the user must be able to navigate with eyes half-closed.

## Findings

### Prior art reference points

1. **Kindle library:** [Kindle library grid (Dribbble)](https://dribbble.com/shots/2915741-Kindle-Library-Grid) and [How to Change Kindle Homescreen to List View (eBookReader)](https://blog.the-ebook-reader.com/2019/01/25/how-to-change-your-kindles-homescreen-to-list-view/): two views — grid (~6 covers) or list (~5–8 titles). Touch-driven. Default homescreen has "currently reading" emphasized + recent books. **Lesson: 5–8 items per screen is the right cardinality. Recents-first works.**

2. **Sol Reader:** Per [Good e-Reader review](https://goodereader.com/blog/tablet-slates/sol-reader-your-own-private-reading-aid-a-review), uses left/right buttons to "turn pages" through the library, center button to select. Up/down menu access. **Lesson: D-pad-class navigation works for ~5 titles per screen.**

3. **Pebble smartwatch:** 144×168 pixel display, 4-button layout (back left, up/down/select right). Lists are vertical scroll, ~3 items visible. [Pebble UX guide (Scribd)](https://www.scribd.com/document/281212800/UX-Guide-for-Pebble-Watch). **Lesson: 3 items at small resolution is fine if scrolling is fast.**

4. **Switch-scanning accessibility (iOS / Android):** Single-button row-column scanning. Item highlighter advances every N seconds; press at desired item. Slow but powerful. [Apple Switch Control](https://support.apple.com/en-us/119835), [Android Switch Access](https://support.google.com/accessibility/android/answer/6301497?hl=en). **Lesson: even a single-button input can navigate arbitrary lists — but the feel is sluggish. Two buttons (next + select) is dramatically better.**

5. **Original Kindle keyboard (2007):** tiny 800×600 e-ink, 5-way directional keypad. Library was list view, scroll with d-pad, select with center. **Lesson: the canonical e-reader UX for non-touch input is well-established. Choice-specs should not invent.**

### Resolution math

- 1.54" panel at 200×200 magnified through pancake → virtual ~10° FoV / ~15-inch screen at 50cm. About 8–12 lines of 16-pt text legible.
- 2.13" panel at 250×122 → wider but lower vertical resolution; ~6 lines.
- 2.9" panel at 296×128 → marginally better.
- 4.2" panel at 400×300 → ~12 lines, the sweet spot for typography but pushes frame depth past 25 mm.

For library list view, **6 items per screen is the design target.** Each item shows: book title (truncated to ~25 chars), author last name (~10 chars), and progress indicator (e.g., a 0–100 fill bar). At 1.54" 200×200: 6 lines × ~28 chars width is achievable with a 14-pt monospaced font.

### Three-screen architecture

Choice-specs has **three top-level screens.** Per Kindle/Kobo precedent, this is sufficient.

#### Screen 1 — Library (entry / browse)

```
┌─────────────────────── Library ─────────────────────┐
│ ▶ The Brothers Karamazov  Dostoyevsky    █████░░░░  │ ← currently reading (highlighted)
│   The Fellowship of the   Tolkien        ████████░  │
│   Children of Time        Tchaikovsky    ███░░░░░░  │
│   The Once and Future K.  White          ░░░░░░░░░  │
│   The Hard Thing          Horowitz       ████░░░░░  │
│   Project Hail Mary       Weir           ░░░░░░░░░  │
│                                                  ▾  │ ← more below
└──────────────────────────────────────────── 12 books┘
```

- **First entry is highlighted** (pointer mark, inverse-video, or both).
- **Currently-reading book is first** — the top-of-list entry on every session.
- **Recents-first ordering** by last-read timestamp.
- **Down arrow at right margin** indicates more books below.
- **Total count** at bottom-right.

#### Screen 2 — Reading

The book itself, full-page, head-locked. Per C1: virtual page at 50 cm, monocular dominant eye. Page corner reserves ~5×5 px for gesture-feedback indicator (per C3 brief 3).

```
┌──────────────────────── ─ ──────────────────────── ┐
│                                                  • │ ← gesture indicator
│   It was the best of times, it was the worst       │
│   of times, it was the age of wisdom, it was       │
│   the age of foolishness, it was the epoch         │
│   of belief, it was the epoch of incredulity,      │
│   ...                                              │
│                                                    │
│                                            42/284  │ ← page indicator
└────────────────────────────────────────────────────┘
```

Minimal chrome: just the gesture-feedback dot + page count.

#### Screen 3 — Settings

Accessed only intentionally (long pitch-up while in library, or a future deliberate combo). Not part of the normal flow. Settings cover: font size, brightness, font (later). Same vertical-list paradigm as Library.

```
┌─────────────────────── Settings ────────────────────┐
│ ▶ Font size              14 pt        ◀───▶         │
│   Brightness             40%          ◀───▶         │
│   Refresh mode           Standard                   │
│   Sleep timeout          30 min                     │
│   ─────────────                                     │
│   Pair BT ring                                      │
│   Calibrate now                                     │
│   ↩ Back to library                                 │
└─────────────────────────────────────────────────────┘
```

### Input → action mapping

| Screen | Roll right (next) | Roll left (prev) | Pitch up (menu) | Pitch down (confirm) | BT ring btn 1 (top) | BT ring btn 2 (bottom) | BT ring btn 3 (mid) |
|---|---|---|---|---|---|---|---|
| **Library** | Next item ↓ | Prev item ↑ | (no-op or alt-action) | Open selected book | Next item | Prev item | Open selected book |
| **Reading** | Next page | Prev page | Open library | (no-op or recalibrate) | Next page | Prev page | Open library |
| **Settings** | Next item ↓ | Prev item ↑ | Back to library | Activate item / +1 | Next item | Prev item | Activate / Back |

### Critical UX behaviors

#### A. List paging when more than 6 books

When the user reaches the bottom of the visible 6, the *next* roll-right scrolls the list by 3 items (a half-page advance), not by 1. Reasons:
- 1-item-at-a-time becomes tedious at 50+ books.
- Half-page shifts give visible context (some old, some new).
- Inverse: roll-left reverses by 3 items at top of visible window.

**Hold-to-fast-scroll.** If the user holds roll-right past the standard 400ms hold (i.e., continues to hold past the gesture fire), the list advances by 6 (full page) every additional 800 ms. Allows fast jump through 50+ books.

#### B. Returning to library mid-book

Pitch-up from reading screen → library. **The library re-renders with the just-exited book at top, highlighted.** This makes "exit and come back" symmetric with "stay reading" — the user doesn't lose their place in the list when they navigate.

#### C. Re-opening a book

Each book's last-read position is persisted. Opening a book always resumes at the last-read page, not the cover. (Exception: first time opening a book, starts at chapter 1.)

#### D. Long titles / author names

Truncate at the displayable width with ellipsis. **Do not** scroll-marquee the title — distracting and wastes refresh cycles.

#### E. Empty library

If the library is empty (newly flashed device): show a single placeholder line "No books yet. Sync via USB-C or WiFi." This is C2/C4 territory but the empty state must be defined.

### Library size scaling

| Library size | UX behavior |
|---|---|
| 1–10 books | No scrolling needed; all visible. |
| 10–50 books | 1–9 page-scroll positions; fast-scroll trivially handles all. |
| 50–200 books | Half-page scroll dominates. Fast-scroll critical. ~8 sec to traverse from first to last book at fast-scroll. |
| 200–1000 books | **Pure scroll breaks down.** Need search or a structural shortcut. |

**At 200+ books, add an "alphabet bar":** a row above the list showing initials A–Z. Roll right at the top of the list selects A, then B, etc.; pitch-down jumps to the first book starting with that letter. This is a Kindle/iPod-class pattern and works without keyboard input.

**For v1: defer alphabet bar until library exceeds 50 books.** Personal feasibility prototype likely starts <30 books.

### Selection feedback

Inverse-video (or distinct background fill) on the highlighted item. E-ink full-refresh costs ~1 sec but partial-refresh of just the highlighted-row delta is ~100 ms. **All selection changes use partial refresh.** Full refresh only on screen transitions (library → book → library).

### State diagram

```
                      ┌────────────────┐
                      │  POWER OFF     │
                      └────────┬───────┘
                               │ power-on (long-press temple btn)
                               ▼
                      ┌────────────────┐
                      │  CALIBRATING   │
                      │  (1.5s still)  │
                      └────────┬───────┘
                               │
                               ▼
                      ┌────────────────┐    pitch-down on selected
                      │   LIBRARY      │ ────────────────────►
                      │ (recents-first │                       ▼
                      │   list view)   │              ┌────────────────┐
                      └─┬────▲─────────┘              │   READING      │
                pitch-up│    │ pitch-up               │ (page render,  │
                  ↑     │    │   exits book           │  gesture loop) │
                        │    │                        └─┬──────────────┘
                        ▼    │                          │
                      ┌──────┴───────┐ pitch-down       │ pitch-up exits
                      │  SETTINGS    │◄───────long───── │
                      │              │ pitch-up (3 sec)
                      └──────────────┘
```

### Why not a grid view?

Grid view is appealing visually for desktop / tablet readers (cover art browsing). Choice-specs's monochrome 200×200 cannot render legible cover-art-with-title-and-author at thumbnail size. List view dominates at this resolution.

### Why no search?

- Voice ruled out (principle 2).
- Typing on head-tilt is brutal (one character per ~2 sec of compound gestures).
- Library size <100 is the v1 scope; alphabet jump suffices.

If search is later wanted: defer to a phone companion app for "send book to read next."

### Why the gesture-indicator dot persists into the library screen

Consistency: the user always knows the device is listening. In reading screen the dot is the only feedback. In library screen the highlighted-row IS the feedback but the dot also reflects gesture-state (active, hold, fire) — useful when the user is mid-gesture and watching for confirmation.

## Recommendation

**Three-screen UX (Library / Reading / Settings) with vertical-list traversal in Library and Settings, head-locked single-page in Reading.**

**Library is the entry screen.** Recents-first vertical list, 6 items visible per screen, currently-reading at top, half-page scroll on roll, hold-to-fast-scroll, partial-refresh selection feedback.

**Input mapping** (head-tilt + optional 3-button BT ring) per the table above. The same gestures mean the same thing on every screen (next, prev, menu/exit, confirm).

**Library scaling:** native list works to ~50 books; add alphabet bar at >50 books; defer beyond 200 to v2.

**No grid view, no search, no nested folders.** Keep it three screens, four gestures.

## Open questions / Gaps

1. **Library list re-ordering policy** — strict "last read" or some user-pin override? Kindle has both. Defer to user testing.
2. **Currently-reading state** — what marks a book as "currently reading" vs "finished" vs "not started"? Probably progress > 0% and < 95%. Books >95% drop to "finished" cohort below the unstarted ones, or hide entirely.
3. **Series grouping.** Series are common in fiction (Lord of the Rings, etc.). Show as one collapsed entry with the active book, or three separate entries? Defer.
4. **Settings entry path** — pitch-up in library is awkward (already used for reading→library). Proposed: long-press pitch-up (3 sec) in library opens settings. Confirm in prototype.
5. **What about a "now reading" splash on session start, before library?** Could skip library entirely if the user 95% of the time wants to resume the current book. Worth A/B testing. Probably **always go straight to currently-reading book at session start, with a 1-second timeout to roll-right (to library list) or do nothing (read).**
6. **End-of-book behavior.** When the reader finishes a book, does the device auto-return to library? Show a "Book complete — open library?" prompt? Just sit on the last page until the user navigates away? Probably the third — minimal magic, maximum predictability.
7. **Settings need an in-place value editor** (e.g., font size 14 → 16). Roll-right increments, roll-left decrements while item is selected. Pitch-down commits. Pitch-up cancels / exits to next item. Keep it consistent.
8. **Cover art rendering** — Kindle shows tiny covers in list view. At 1.54" panels we may not have pixel budget. Defer; v1 = title + author text only.

## Sources

- [Kindle Library Grid (Dribbble)](https://dribbble.com/shots/2915741-Kindle-Library-Grid)
- [How to Change Kindle Homescreen to List View (eBookReader)](https://blog.the-ebook-reader.com/2019/01/25/how-to-change-your-kindles-homescreen-to-list-view/)
- [3 Different Kindle Homescreens (eBookReader)](https://blog.the-ebook-reader.com/2020/07/09/3-different-kindle-homescreens-which-is-your-favorite/)
- [Sol Reader Review (Good e-Reader)](https://goodereader.com/blog/tablet-slates/sol-reader-your-own-private-reading-aid-a-review)
- [Pebble UX Design Developer Guide (Scribd)](https://www.scribd.com/document/281212800/UX-Guide-for-Pebble-Watch)
- [Pebble Time documentation (Rebble legacy)](https://pebble-help-legacy.rebble.io/help.getpebble.com/customer/en/portal/articles/2553606-using-pebble-time2d1a.html)
- [iOS 14 Switch Control User Guide (AbleNet)](https://www.ablenetinc.com/content/html/Downloads/Switch_Downloads/ios_14_switch_control_user_guide_english.pdf)
- [Switch Access (Android)](https://support.google.com/accessibility/android/answer/6301497?hl=en)
- [Apple Switch Control (iOS)](https://support.apple.com/en-us/119835)
- [KOReader (GitHub)](https://github.com/koreader/koreader)
- [Plato (GitHub)](https://github.com/baskerville/plato)
