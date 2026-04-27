# Gallery Ripple + Multi-Focus · Scene Orchestration Philosophy

> Reusable visual orchestration distilled from huashu-design hero animation v9 (25s, 8 scenes).
> Not an animation production pipeline — guide to **when this orchestration is "the right call"**.
> Production reference: [demos/hero-animation-v9.mp4](../demos/hero-animation-v9.mp4) · [https://www.huasheng.ai/huashu-design-hero/](https://www.huasheng.ai/huashu-design-hero/)

## In One Sentence

> **When you have 20+ visually homogeneous assets and the scene needs to "express scale and depth," reach for Gallery Ripple + Multi-Focus before layout stacking.**

Generic SaaS features, product launches, skill promotions, portfolio showcases — with enough consistently styled assets, this structure almost always delivers.

---

## What This Technique Is Actually Expressing

Not "showing off assets" — uses **two rhythmic shifts** to tell a story:

**First beat · Ripple Expansion (~1.5s)**: 48 cards radiate from center; audience stunned by *quantity* — "oh, this produced this much."

**Second beat · Multi-Focus (~8s, 4 cycles)**: Camera slowly pans while dimming + desaturating background 4×, bringing one card to screen center — shifts audience from "shock at quantity" to "contemplation of quality." Each cycle runs at steady 1.7s rhythm.

**Core narrative**: **Scale (Ripple) → Contemplation (Focus × 4) → Fade Out (Walloff)**. Expresses "Breadth × Depth" — not just prolific, but every piece worth pausing on.

| Approach | Audience Perception |
|------|---------|
| 48 cards static (no Ripple) | Attractive but no narrative — like a grid screenshot |
| One-by-one fast cuts (no Gallery) | Slideshow feel — loses "sense of scale" |
| Only Ripple, no Focus | Stunned by quantity but remembers nothing specific |
| **Ripple + Focus × 4 (this recipe)** | **Overwhelmed by quantity → drawn into quality → calm resolution — complete emotional arc** |

---

## Prerequisites (All Four Must Be Met)

This orchestration **is not universal**:

1. **Asset count ≥ 20, ideally 30+**
   Fewer than 20 and Ripple looks sparse — every cell in the 48-slot grid must be moving for density. v9 used 48 slots × 32 images (cycled to fill).

2. **Assets share consistent visual style**
   All 16:9 slide previews / all app screenshots / all covers — aspect ratio, tone, layout read as "a set." Mixed styles make Gallery look like a clipboard.

3. **Assets readable when enlarged**
   Focus zooms card to 960px wide. If original blurs or has too little content at that size, Focus beat fails. Reverse validation: can you pick 4 cards as "most representative"? If not, asset quality isn't consistent enough.

4. **Landscape or square, not portrait**
   3D tilt (`rotateX(14deg) rotateY(-10deg)`) requires horizontal extension. Portrait makes tilt look narrow and awkward.

**Fallback paths**:

| Missing What | Fall Back To |
|-------|-----------|
| Assets < 20 | "3–5 cards side-by-side static + individual focus" |
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

**Key**: Canvas is 2.25× larger than viewport — gives pan its "glimpsing a larger world" feel.

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
- Max delay 0.8s (center exits first, corners last)
- Each card entry duration 0.7s
- Easing: `expoOut` (explosive, not smooth)

**Simultaneously**: canvas scales 1.25 → 0.94 (zoom out to reveal), synchronized with card appearance for "pulling back" sense.

### Multi-Focus (4-Beat Rhythm)

```js
T.focuses = [
  { start: 11.0, end: 12.7, idx: 2  },  // 1.7s
  { start: 13.3, end: 15.0, idx: 3  },  // 1.7s
  { start: 15.6, end: 17.3, idx: 10 },  // 1.7s
  { start: 17.9, end: 19.6, idx: 16 },  // 1.7s
];
```

**Rhythm**: Each focus 1.7s, 0.6s breathing gap between. Total 8s (11.0–19.6s).

**Inside each focus**:
- In ramp: 0.4s (`expoOut`)
- Hold: middle 0.9s (`focusIntensity = 1`)
- Out ramp: 0.4s (`easeOut`)

**Background changes (the key)**:

```js
if (focusIntensity > 0) {
  const dimOp = entryOp * (1 - 0.6 * focusIntensity);  // dim to 40%
  const brt = 1 - 0.32 * focusIntensity;                // brightness 68%
  const sat = 1 - 0.35 * focusIntensity;                // saturate 65%
  card.style.filter = `brightness(${brt}) saturate(${sat})`;
}
```

**Not just opacity — simultaneously desaturate + darken**. Makes foreground overlay color "pop," not just "get brighter."

**Focus overlay size**: 400×225 (entry) → 960×540 (hold). 3-layer shadow + 3px accent-color outline ring = "framed" feeling.

### Pan (Continuity That Keeps Static From Feeling Boring)

```js
const panT = Math.max(0, t - 8.6);
const panX = Math.sin(panT * 0.12) * 220 - panT * 8;
const panY = Math.cos(panT * 0.09) * 120 - panT * 5;
```

- Dual-layer motion: sine + linear drift — not pure loop; position unique at every moment
- X/Y frequencies differ (0.12 vs 0.09) to prevent audience detecting a regular cycle
- Clamped to ±900/500px to prevent drifting off screen

**Why not pure linear pan**: Pure linear lets audience "predict" next position. Sine + drift makes every second new territory — under 3D tilt produces gentle "slight sea-legs" sensation (good kind) that holds attention.

---

## 5 Reusable Patterns (Distilled From v6→v9 Iterations)

### 1. **expoOut as Primary Easing, Not cubicOut**

`easeOut = 1 - (1-t)³` (smooth) vs `expoOut = 1 - 2^(-10t)` (explosive then rapidly settling).

**Why**: expoOut reaches 90% of destination in first 30% of duration — more like physical damping. Especially good for:
- Card entries (sense of weight)
- Ripple expansion (shockwave feel)
- Brand element rising (settled, landed feeling)

**Still use cubicOut for**: focus out ramps, symmetric micro-animations.

### 2. **Paper-Tone Background + Terracotta Orange Accent (Anthropic DNA)**

```css
--bg: #F7F4EE;        /* Warm paper */
--ink: #1D1D1F;       /* Near-black */
--accent: #D97757;    /* Terracotta orange */
--hairline: #E4DED2;  /* Warm line */
```

**Why**: Warm backgrounds retain "breathing" even after GIF compression — unlike pure white which looks "screen-ish." Terracotta as sole accent threads through terminal prompt, selected dir-card, cursor, brand hyphen, focus ring — all visual anchors tied by one color.

**v5 lesson**: Noise overlay simulating "paper grain" was destroyed entirely by GIF frame compression. v6 switched to "background color + warm shadow only" — paper feel retained 90%, GIF file size reduced 60%.

### 3. **Two Shadow Tiers to Simulate Depth — No Real 3D**

```css
.gallery-card.depth-near { box-shadow: 0 32px 80px -22px rgba(60,40,20,0.22), ... }
.gallery-card.depth-far  { box-shadow: 0 14px 40px -16px rgba(60,40,20,0.10), ... }
```

Deterministic `sin(i × 1.7) + cos(i × 0.73)` assigns each card to near/mid/far shadow tier — **visually creates "3D stacked" feel, per-frame transforms never change, GPU cost is 0**.

**Cost of real 3D**: Every card with individual `translateZ`, GPU calculating 48 transforms + shadow blurs per frame. Tried in v4 — Playwright struggled at 25fps. Two-tier shadow in v6 is <5% visually different from real 3D, 10× cheaper.

### 4. **Font Weight Variation (font-variation-settings) Is More Cinematic Than Size Variation**

```js
const wght = 100 + (700 - 100) * morphP;  // 100 → 700 over 0.9s
wordmark.style.fontVariationSettings = `"wght" ${wght.toFixed(0)}`;
```

Brand wordmark transitions Thin → Bold over 0.9s, with subtle letter-spacing adjustment (−0.045 → −0.048em).

**Why it's better than scaling**:
- Scale-in/out is so familiar audiences have fixed expectations
- Weight change is "internal fullness" — like a balloon inflating, not "being pushed closer"
- Variable fonts are 2020+ technology; audiences subconsciously register "modern"

**Limitation**: Requires variable font (Inter / Roboto Flex / Recursive, etc.). Static fonts can only fake it with weight switching, which produces a jump.

### 5. **Corner Brand — Low-Intensity Persistent Signature**

During Gallery phase, small `HUASHU · DESIGN` mark in top-left: 16% opacity, 12px type, wide letter-spacing.

**Why**:
- After Ripple explosion, audiences easily defocus and forget what they're watching — corner mark anchors them
- More premium than full-screen logo — experienced brand people know signatures don't need to shout
- When GIF gets screenshotted and shared, attribution signal persists

**Rule**: Only appears during middle section (when frame is busy), hidden on entry (doesn't block terminal), hidden on exit (brand reveal is the star).

---

## Counter-Examples: When Not to Use This Orchestration

**❌ Product demo (needs to demonstrate features)**: Gallery flashes every card; audience won't remember any specific feature. Use "single-screen focus + tooltip annotations."

**❌ Data-driven content**: Audience needs to read numbers; Gallery's pace doesn't allow it. Use "data charts + step-by-step reveal."

**❌ Story narrative**: Gallery is "parallel" structure; stories need "causality." Use keynote chapter transitions.

**❌ Only 3–5 assets**: Ripple density too low — looks like a "patch job." Use "static arrangement + one-by-one highlight."

**❌ Portrait (9:16)**: 3D tilt needs horizontal extension; portrait looks "tilted" rather than "opened up."

---

## How to Determine If Your Task Suits This Orchestration

Three-step check:

**Step 1 · Asset count**: Count visually homogeneous assets. <15 → stop; 15–25 → marginal; 25+ → use it.

**Step 2 · Consistency test**: Place 4 random assets side by side — do they read as "a set"? If not → unify style first or change approach.

**Step 3 · Narrative fit**: Expressing "Breadth × Depth" (quantity × quality)? Or "process," "features," "story"? If not the former, don't force it.

All three yes → fork v6 HTML, swap `SLIDE_FILES` array and timeline. Change palette via `--bg / --accent / --ink` — swap skin, keep bones.

---

## Related References

- Full technical flow: [references/animations.md](animations.md) · [references/animation-best-practices.md](animation-best-practices.md)
- Animation export pipeline: [references/video-export.md](video-export.md)
- Audio setup (BGM + SFX dual-track): [references/audio-design-rules.md](audio-design-rules.md)
- Apple gallery style reference: [references/apple-gallery-showcase.md](apple-gallery-showcase.md)
- Source HTML (v6 + audio): `www.huasheng.ai/huashu-design-hero/index.html`
