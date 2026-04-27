# Gallery Ripple + Multi-Focus · Scene Orchestration Philosophy

> A **reusable visual orchestration structure** distilled from huashu-design hero animation v9 (25 seconds, 8 scenes).
> This is not an animation production pipeline — it's a guide to **when this orchestration is "the right call"**.
> Production reference: [demos/hero-animation-v9.mp4](../demos/hero-animation-v9.mp4) · [https://www.huasheng.ai/huashu-design-hero/](https://www.huasheng.ai/huashu-design-hero/)

## In One Sentence

> **When you have 20+ visually homogeneous assets and the scene needs to "express scale and depth," reach for Gallery Ripple + Multi-Focus before you reach for layout stacking.**

Generic SaaS feature animations, product launches, skill promotions, series portfolio showcases — as long as you have enough assets with a consistent style, this structure almost always delivers.

---

## What This Technique Is Actually Expressing

It's not "showing off assets" — it uses **two rhythmic shifts** to tell a story:

**First beat · Ripple Expansion (~1.5s)**: 48 cards radiate outward from center, and the audience is stunned by *quantity* — "oh, this thing has produced this much."

**Second beat · Multi-Focus (~8s, 4 cycles)**: While the camera slowly pans, it dims + desaturates the background 4 times, bringing a single card to the center of the screen — the audience shifts from "shock at quantity" to "contemplation of quality." Each cycle runs at a steady 1.7s rhythm.

**Core narrative structure**: **Scale (Ripple) → Contemplation (Focus × 4) → Fade Out (Walloff)**. Together these three beats express "Breadth × Depth" — not just prolific, but every single one worth pausing to look at.

By contrast:

| Approach | Audience Perception |
|------|---------|
| 48 cards in static arrangement (no Ripple) | Attractive but no narrative — like a grid screenshot |
| One-by-one fast cuts (no Gallery context) | Feels like a slideshow — loses "sense of scale" |
| Only Ripple, no Focus | Stunned by quantity but doesn't remember any specific card |
| **Ripple + Focus × 4 (this recipe)** | **First overwhelmed by quantity, then drawn into quality, then calm resolution — a complete emotional arc** |

---

## Prerequisites (All Four Must Be Met)

This orchestration **is not universal**. All 4 of the following must apply:

1. **Asset count ≥ 20, ideally 30+**
   Fewer than 20 assets and Ripple looks "sparse" — every cell in the 48-slot grid needs to be moving to achieve density. v9 used 48 slots × 32 images (cycled to fill).

2. **Assets share a consistent visual style**
   All 16:9 slide previews / all app screenshots / all cover designs — aspect ratio, tone, and layout need to read as "a set." Mixed styles make the Gallery look like a clipboard.

3. **Assets remain readable when enlarged**
   Focus zooms a card to 960px wide. If the original blurs out or has too little content at that size, the Focus beat fails. Reverse validation: can you pick 4 cards from the 48 as the "most representative"? If not, the asset quality isn't consistent enough.

4. **The scene is landscape or square, not portrait**
   The Gallery's 3D tilt (`rotateX(14deg) rotateY(-10deg)`) requires horizontal extension. Portrait orientation makes the tilt look narrow and awkward.

**Fallback paths when conditions aren't met**:

| Missing What | Fall Back To |
|-------|-----------|
| Assets < 20 | "3–5 cards side-by-side static display + individual focus" |
| Inconsistent style | "Cover + 3 chapter hero images" keynote-style |
| Content too sparse | "Data-driven dashboard" or "headline + large text" |
| Portrait orientation | "Vertical scroll + sticky cards" |

---

## Technical Recipe (v9 Production Parameters)

### 4-Layer Structure

```
viewport (1920×1080, perspective: 2400px)
  └─ canvas (4320×2520, large overflow) → 3D tilt + pan
      └─ 8×6 grid = 48 cards (gap 40px, padding 60px)
          └─ img (16:9, border-radius 9px)
      └─ focus-overlay (absolute center, z-index 40)
          └─ img (matches selected slide)
```

**Key**: Canvas is 2.25× larger than the viewport — this gives the pan its "glimpsing a larger world" feel.

### Ripple Expansion (Distance-Delay Algorithm)

```js
// Each card's entry time = distance from center × 0.8s delay
const col = i % 8, row = Math.floor(i / 8);
const dc = col - 3.5, dr = row - 2.5;       // Offset from center
const dist = Math.hypot(dc, dr);
const maxDist = Math.hypot(3.5, 2.5);
const delay = (dist / maxDist) * 0.8;       // 0 → 0.8s
const localT = Math.max(0, (t - rippleStart - delay) / 0.7);
const opacity = expoOut(Math.min(1, localT));
```

**Core parameters**:
- Total duration 1.7s (`T.s3_ripple: [8.3, 10.0]`)
- Maximum delay 0.8s (center exits first, corners exit last)
- Each card entry duration 0.7s
- Easing: `expoOut` (explosive feel, not smooth)

**Simultaneously**: canvas scales 1.25 → 0.94 (zoom out to reveal) — synchronized with the cards' appearance for a sense of pulling back.

### Multi-Focus (4-Beat Rhythm)

```js
T.focuses = [
  { start: 11.0, end: 12.7, idx: 2  },  // 1.7s
  { start: 13.3, end: 15.0, idx: 3  },  // 1.7s
  { start: 15.6, end: 17.3, idx: 10 },  // 1.7s
  { start: 17.9, end: 19.6, idx: 16 },  // 1.7s
];
```

**Rhythm pattern**: Each focus 1.7s, 0.6s breathing gap between. Total 8s (11.0–19.6s).

**Inside each focus**:
- In ramp: 0.4s (`expoOut`)
- Hold: middle 0.9s (`focusIntensity = 1`)
- Out ramp: 0.4s (`easeOut`)

**Background changes (this is the key)**:

```js
if (focusIntensity > 0) {
  const dimOp = entryOp * (1 - 0.6 * focusIntensity);  // dim to 40%
  const brt = 1 - 0.32 * focusIntensity;                // brightness 68%
  const sat = 1 - 0.35 * focusIntensity;                // saturate 65%
  card.style.filter = `brightness(${brt}) saturate(${sat})`;
}
```

**It's not just opacity — simultaneously desaturate + darken**. This makes the foreground overlay's color "pop," not just "get brighter."

**Focus overlay size animation**:
- From 400×225 (entry) → 960×540 (hold state)
- 3-layer shadow + 3px accent-color outline ring around it — creates a "framed" feeling

### Pan (Continuity That Keeps Static From Feeling Boring)

```js
const panT = Math.max(0, t - 8.6);
const panX = Math.sin(panT * 0.12) * 220 - panT * 8;
const panY = Math.cos(panT * 0.09) * 120 - panT * 5;
```

- Dual-layer motion: sine wave + linear drift — not a pure loop; position is unique at every moment
- X/Y frequencies differ (0.12 vs 0.09) to prevent the audience from visually detecting a "regular cycle"
- Clamped to ±900/500px to prevent drifting off screen

**Why not use pure linear pan**: Pure linear lets the audience "predict" where it goes next. Sine + drift makes every second new territory — under the 3D tilt, it produces a gentle "slight sea-legs" sensation (the good kind) that holds attention.

---

## 5 Reusable Patterns (Distilled From v6→v9 Iterations)

### 1. **expoOut as Primary Easing, Not cubicOut**

`easeOut = 1 - (1-t)³` (smooth) vs `expoOut = 1 - 2^(-10t)` (explosive then rapidly settling).

**Why**: expoOut reaches 90% of its destination in the first 30% of its duration — more like physical damping, matching the intuition of "something heavy landing." Especially good for:
- Card entries (sense of weight)
- Ripple expansion (shockwave feel)
- Brand element rising (settled, landed feeling)

**When to still use cubicOut**: focus out ramps, symmetric micro-animations.

### 2. **Paper-Tone Background + Terracotta Orange Accent (Anthropic DNA)**

```css
--bg: #F7F4EE;        /* Warm paper */
--ink: #1D1D1F;       /* Near-black */
--accent: #D97757;    /* Terracotta orange */
--hairline: #E4DED2;  /* Warm line */
```

**Why**: Warm backgrounds retain a sense of "breathing" even after GIF compression — unlike pure white, which looks "screen-ish." Terracotta orange as the sole accent threads through terminal prompt, selected dir-card, cursor, brand hyphen, focus ring — every visual anchor tied together by one color.

**v5 lesson**: Added noise overlay to simulate "paper grain" — but GIF frame compression destroyed it entirely (every frame looked different). v6 switched to "background color + warm shadow only"; paper feel retained 90%, GIF file size reduced 60%.

### 3. **Two Shadow Tiers to Simulate Depth — No Real 3D**

```css
.gallery-card.depth-near { box-shadow: 0 32px 80px -22px rgba(60,40,20,0.22), ... }
.gallery-card.depth-far  { box-shadow: 0 14px 40px -16px rgba(60,40,20,0.10), ... }
```

A deterministic `sin(i × 1.7) + cos(i × 0.73)` algorithm assigns each card to near/mid/far shadow tier — **visually creates a "3D stacked" feel, but per-frame transforms never change, so GPU cost is 0**.

**The cost of real 3D**: Every card with individual `translateZ`, GPU calculating 48 transforms + shadow blurs every frame. Tried it in v4 — Playwright struggled to record at 25fps. Two-tier shadow in v6 is <5% visually different from real 3D, but 10× cheaper.

### 4. **Font Weight Variation (font-variation-settings) Is More Cinematic Than Font Size Variation**

```js
const wght = 100 + (700 - 100) * morphP;  // 100 → 700 over 0.9s
wordmark.style.fontVariationSettings = `"wght" ${wght.toFixed(0)}`;
```

Brand wordmark transitions from Thin → Bold over 0.9s, with subtle letter-spacing adjustment (−0.045 → −0.048em).

**Why it's better than scaling**:
- Scale-in/out has been seen so many times the audience has a fixed expectation
- Weight change is "internal fullness" — like a balloon inflating, not "being pushed closer"
- Variable fonts are a 2020+ technology; audiences subconsciously register "modern"

**Limitation**: Requires a font that supports variable font format (Inter / Roboto Flex / Recursive, etc.). Static fonts can only fake it by switching between fixed weights, which produces a jump.

### 5. **Corner Brand — Low-Intensity Persistent Signature**

During the Gallery phase, a small `HUASHU · DESIGN` mark appears in the top-left: 16% opacity, 12px type, wide letter-spacing.

**Why include it**:
- After the Ripple explosion, audiences easily "defocus" and forget what they're watching. The corner mark helps anchor them
- More premium than a full-screen logo — experienced brand people know brand signatures don't need to shout
- When the GIF gets screenshot and shared, attribution signal still comes through

**Rule**: Only appears during the middle section (when the frame is busy), hidden on entry (doesn't block the terminal), hidden on exit (brand reveal is the star).

---

## Counter-Examples: When Not to Use This Orchestration

**❌ Product demo (needs to demonstrate features)**: Gallery lets every card flash by; the audience won't remember any specific feature. Switch to "single-screen focus + tooltip annotations."

**❌ Data-driven content**: The audience needs to read numbers, and Gallery's fast pace doesn't give time for that. Switch to "data charts + step-by-step reveal."

**❌ Story narrative**: Gallery is a "parallel" structure; stories need "causality." Switch to keynote chapter transitions.

**❌ Only 3–5 assets**: Ripple density is too low — looks like a "patch job." Switch to "static arrangement + one-by-one highlight."

**❌ Portrait (9:16)**: 3D tilt needs horizontal extension; in portrait it looks "tilted" rather than "opened up."

---

## How to Determine If Your Task Suits This Orchestration

Three-step quick check:

**Step 1 · Asset count**: Count your visually homogeneous assets. < 15 → stop; 15–25 → marginal; 25+ → use it.

**Step 2 · Consistency test**: Place 4 random assets side by side — do they read as "a set"? If not → unify the style first or change approach.

**Step 3 · Narrative fit**: Are you trying to express "Breadth × Depth" (quantity × quality)? Or is it "process," "features," "story"? If it's not the former, don't force it.

All three are yes → fork the v6 HTML directly, swap the `SLIDE_FILES` array and timeline to reuse. Change the palette via `--bg / --accent / --ink` — swap the skin, keep the bones.

---

## Related References

- Full technical flow: [references/animations.md](animations.md) · [references/animation-best-practices.md](animation-best-practices.md)
- Animation export pipeline: [references/video-export.md](video-export.md)
- Audio setup (BGM + SFX dual-track): [references/audio-design-rules.md](audio-design-rules.md)
- Lateral reference for Apple gallery style: [references/apple-gallery-showcase.md](apple-gallery-showcase.md)
- Source HTML (v6 + audio integration): `www.huasheng.ai/huashu-design-hero/index.html`
