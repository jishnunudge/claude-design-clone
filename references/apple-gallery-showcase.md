# Apple Gallery Showcase · Gallery Wall Animation Style

> Inspiration: Claude Design official hero video + Apple product page "portfolio wall"
> Origin: huashu-design launch hero v5
> Scenarios: **Product launch heroes, skill capability demos, portfolio showcases** — any case needing "multiple high-quality outputs simultaneously" while guiding attention

---

## Trigger Criteria: When to Use This Style

**Good fit**:
- 10+ real outputs on screen simultaneously (slides, apps, webpages, infographics)
- Professional audience (devs, designers, PMs) sensitive to "craftsmanship"
- Desired aesthetic: "restrained, gallery-like, premium, spatial depth"
- Need focus and overview to coexist (details without losing the whole)

**Not a good fit**:
- Single-product focus (use frontend-design product hero template)
- Emotionally driven / strong narrative (use timeline narrative template)
- Small screens / portrait orientation (angled perspective blurs in small frames)

---

## Core Visual Tokens

```css
:root {
  /* Light gallery palette */
  --bg:         #F5F5F7;   /* Main canvas — Apple website gray */
  --bg-warm:    #FAF9F5;   /* Warm off-white variant */
  --ink:        #1D1D1F;   /* Primary text */
  --ink-80:     #3A3A3D;
  --ink-60:     #545458;
  --muted:      #86868B;   /* Secondary text */
  --dim:        #C7C7CC;
  --hairline:   #E5E5EA;   /* Card 1px border */
  --accent:     #D97757;   /* Terracotta orange — Claude brand */
  --accent-deep:#B85D3D;

  --serif-cn: "Noto Serif SC", "Songti SC", Georgia, serif;
  --serif-en: "Source Serif 4", "Tiempos Headline", Georgia, serif;
  --sans:     "Inter", -apple-system, "PingFang SC", system-ui;
  --mono:     "JetBrains Mono", "SF Mono", ui-monospace;
}
```

**Key principles**:
1. **Never use pure black backgrounds** — makes work look cinematic rather than "usable deliverables"
2. **Terracotta orange is the only hue accent** — everything else grayscale + white
3. **Three-font stack** (serif-EN + serif-CN + sans + mono) creates "publication" not "internet product" feel

---

## Core Layout Patterns

### 1. Floating Card (Basic Unit)

```css
.gallery-card {
  background: #FFFFFF;
  border-radius: 14px;
  padding: 6px;                          /* Inner padding acts as "mat board" */
  border: 1px solid var(--hairline);
  box-shadow:
    0 20px 60px -20px rgba(29, 29, 31, 0.12),   /* Primary shadow: soft and long */
    0 6px 18px -6px rgba(29, 29, 31, 0.06);     /* Second layer: close-light for float */
  aspect-ratio: 16 / 9;
  overflow: hidden;
}
.gallery-card img {
  width: 100%; height: 100%;
  object-fit: cover;
  border-radius: 9px;                    /* Slightly smaller than card's radius — visual nesting */
}
```

**Anti-example**: Don't tile edge-to-edge (no padding, no border, no shadow) — that's infographic density, not gallery.

### 2. 3D Angled Portfolio Wall

```css
.gallery-viewport {
  position: absolute; inset: 0;
  overflow: hidden;
  perspective: 2400px;                   /* Deep perspective keeps tilt from looking exaggerated */
  perspective-origin: 50% 45%;
}
.gallery-canvas {
  width: 4320px;                         /* Canvas = 2.25× viewport */
  height: 2520px;
  transform-origin: center center;
  transform: perspective(2400px)
             rotateX(14deg)              /* Lean back */
             rotateY(-10deg)             /* Turn left */
             rotateZ(-2deg);             /* Slight twist — removes excessive regularity */
  display: grid;
  grid-template-columns: repeat(8, 1fr);
  gap: 40px;
  padding: 60px;
}
```

**Parameter sweet spots**:
- rotateX: 10–15deg (more looks like VIP event backdrop)
- rotateY: ±8–12deg (symmetrical feel)
- rotateZ: ±2–3deg (human touch — "not machine-placed")
- perspective: 2000–2800px (below 2000 causes fisheye; above 3000 approaches orthographic)

### 3. 2×2 Four-Corner Convergence (Selection Scene)

```css
.grid22 {
  display: grid;
  grid-template-columns: repeat(2, 800px);
  gap: 56px 64px;
  align-items: start;
}
```

Each card slides from its corner (tl/tr/bl/br) toward center + fades in:

```js
const cornerEntry = {
  tl: { dx: -700, dy: -500 },
  tr: { dx:  700, dy: -500 },
  bl: { dx: -700, dy:  500 },
  br: { dx:  700, dy:  500 },
};
```

---

## Five Core Animation Modes

### Mode A · Four-Corner Convergence (0.8–1.2s)

4 elements slide from viewport corners, scaling 0.85→1.0, ease-out. Good for opening scenes "showing choices from multiple directions."

```js
const inP = easeOut(clampLerp(t, start, end));
card.style.transform = `translate3d(${(1-inP)*ce.dx}px, ${(1-inP)*ce.dy}px, 0) scale(${0.85 + 0.15*inP})`;
card.style.opacity = inP;
```

### Mode B · Selected Card Enlarges + Others Slide Out (0.8s)

Selected card scales 1.0→1.28; others fade + blur + drift back:

```js
// Selected card
card.style.transform = `translate3d(${cellDx*outP}px, ${cellDy*outP}px, 0) scale(${1 + 0.28*easeOut(zoomP)})`;
// Non-selected cards
card.style.opacity = 1 - outP;
card.style.filter = `blur(${outP * 1.5}px)`;
```

**Key**: Non-selected cards must blur, not just fade. Blur simulates depth of field, visually "pushing" selected card forward.

### Mode C · Ripple Expansion (1.7s)

From center outward, each card fades in with distance-based delay + scales 1.25x → 0.94x ("camera pulls back"):

```js
const col = i % COLS, row = Math.floor(i / COLS);
const dc = col - (COLS-1)/2, dr = row - (ROWS-1)/2;
const dist = Math.sqrt(dc*dc + dr*dr);
const delay = (dist / maxDist) * 0.8;
const localT = Math.max(0, (t - rippleStart - delay) / 0.7);
card.style.opacity = easeOut(Math.min(1, localT));

// Simultaneously scale entire gallery 1.25→0.94
const galleryScale = 1.25 - 0.31 * easeOut(rippleProgress);
```

### Mode D · Sinusoidal Pan (Continuous Drift)

Combines sine wave with linear drift to avoid "has a start and end" marquee feel:

```js
const panX = Math.sin(panT * 0.12) * 220 - panT * 8;    // Horizontal drift left
const panY = Math.cos(panT * 0.09) * 120 - panT * 5;    // Vertical drift up
const clampedX = Math.max(-900, Math.min(900, panX));   // Prevent edge exposure
```

**Parameters**:
- Sine period `0.09–0.15 rad/s` (slow — 30–50s per oscillation)
- Linear drift `5–8 px/s` (slower than eye blink)
- Amplitude `120–220 px` (large enough to feel, small enough not to cause motion sickness)

### Mode E · Focus Overlay (Focus Switch)

Focus overlay is **flat** (not tilted), floating above angled canvas. Selected slide scales from tile (~400×225) to screen center (960×540); background dims to 45%:

```js
// Focus overlay (flat, centered)
focusOverlay.style.width = (startW + (endW - startW) * focusIntensity) + 'px';
focusOverlay.style.height = (startH + (endH - startH) * focusIntensity) + 'px';
focusOverlay.style.opacity = focusIntensity;

// Background cards dim but remain visible (critical — do not 100% mask)
card.style.opacity = entryOp * (1 - 0.55 * focusIntensity);   // 1 → 0.45
card.style.filter = `brightness(${1 - 0.3 * focusIntensity})`;
```

**Clarity rule**:
- Focus overlay `<img>` must point to original image — **never reuse compressed gallery thumbnails**
- Preload all originals into `new Image()[]` array
- Overlay `width/height` calculated per-frame; browser resamples original every frame

---

## Timeline Architecture (Reusable Skeleton)

```js
const T = {
  DURATION: 25.0,
  s1_in: [0.0, 0.8],    s1_type: [1.0, 3.2],  s1_out: [3.5, 4.0],
  s2_in: [3.9, 5.1],    s2_hold: [5.1, 7.0],  s2_out: [7.0, 7.8],
  s3_hold: [7.8, 8.3],  s3_ripple: [8.3, 10.0],
  panStart: 8.6,
  focuses: [
    { start: 11.0, end: 12.7, idx: 2  },
    { start: 13.3, end: 15.0, idx: 3  },
    { start: 15.6, end: 17.3, idx: 10 },
    { start: 17.9, end: 19.6, idx: 16 },
  ],
  s4_walloff: [21.1, 21.8], s4_in: [21.8, 22.7], s4_hold: [23.7, 25.0],
};

// Core easing
const easeOut = t => 1 - Math.pow(1 - t, 3);
const easeInOut = t => t < 0.5 ? 4*t*t*t : 1 - Math.pow(-2*t+2, 3)/2;
function lerp(time, start, end, fromV, toV, easing) {
  if (time <= start) return fromV;
  if (time >= end) return toV;
  let p = (time - start) / (end - start);
  if (easing) p = easing(p);
  return fromV + (toV - fromV) * p;
}

// Single render(t) reads timestamp, writes all elements
function render(t) { /* ... */ }
requestAnimationFrame(function tick(now) {
  const t = ((now - startMs) / 1000) % T.DURATION;
  render(t);
  requestAnimationFrame(tick);
});
```

**Architectural essence**: **All state derived from timestamp t** — no state machines, no setTimeout.
- Jump anywhere with `window.__setTime(12.3)` (great for Playwright frame-by-frame capture)
- Loops naturally seamless (t mod DURATION)
- Any frame can be frozen for debugging

---

## Texture Details (Easy to Overlook, Deadly When Missing)

### 1. SVG Noise Texture

Light backgrounds look flat. Layer barely-perceptible fractalNoise on top:

```html
<style>
.stage::before {
  content: '';
  position: absolute; inset: 0;
  background-image: url("data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='200' height='200'><filter id='n'><feTurbulence type='fractalNoise' baseFrequency='0.85' numOctaves='2' stitchTiles='stitch'/><feColorMatrix values='0 0 0 0 0.078  0 0 0 0 0.078  0 0 0 0 0.074  0 0 0 0.035 0'/></filter><rect width='100%' height='100%' filter='url(%23n)'/></svg>");
  opacity: 0.5;
  pointer-events: none;
  z-index: 30;
}
</style>
```

Looks nearly the same with it — remove it and you'll notice immediately.

### 2. Corner Brand Mark

```html
<div class="corner-brand">
  <div class="mark"></div>
  <div>HUASHU · DESIGN</div>
</div>
```

```css
.corner-brand {
  position: absolute; top: 48px; left: 72px;
  font-family: var(--mono);
  font-size: 12px;
  letter-spacing: 0.22em;
  text-transform: uppercase;
  color: var(--muted);
}
```

Only visible during gallery wall scene, fades in/out. Like a museum exhibition label.

### 3. Brand Resolution Wordmark

```css
.brand-wordmark {
  font-family: var(--sans);
  font-size: 148px;
  font-weight: 700;
  letter-spacing: -0.045em;   /* Negative tracking key — letters tighten into a mark */
}
.brand-wordmark .accent {
  color: var(--accent);
  font-weight: 500;           /* Accent character lighter — creates visual contrast */
}
```

`letter-spacing: -0.045em` is Apple's standard approach for large display text on product pages.

---

## Common Failure Modes

| Symptom | Cause | Fix |
|---|---|---|
| Looks like PowerPoint template | No shadow / hairline on cards | Add two-layer box-shadow + 1px border |
| Tilt looks cheap | Only rotateY, no rotateZ | Add ±2–3deg rotateZ to break rigidity |
| Pan feels jerky | setTimeout or CSS keyframe loops | Use rAF + sin/cos continuous functions |
| Text unreadable during Focus | Low-res gallery tile reused | Use separate overlay with original image src |
| Background too empty | Plain `#F5F5F7` | Overlay SVG fractalNoise at 0.5 opacity |
| Typography too "internet" | Only Inter | Add Serif (EN + CN) + mono — three-font stack |

---

## References

- Full implementation: `/Users/alchain/Documents/写作/01-公众号写作/项目/2026.04-huashu-design发布/配图/hero-animation-v5.html`
- Original inspiration: claude.ai/design hero video
- Reference aesthetics: Apple product pages, Dribbble shot collection pages

When encountering "display multiple high-quality outputs," copy the skeleton from this file directly, swap content, adjust timing.
