# Editable PPTX Export: HTML Hard Constraints + Size Decisions + Common Errors

This document covers the path of **using `scripts/html2pptx.js` + `pptxgenjs` to translate HTML element-by-element into true editable PowerPoint text boxes**, which is the only path supported by `export_deck_pptx.mjs`.

> **Core prerequisite**: To take this path, HTML must follow the 4 constraints below from the very first line. **Not write first then convert** — retroactive fixes trigger 2-3 hours of rework (real trap from 2026-04-20 stock options private board project).
>
> For scenarios where visual freedom is the priority (animations / web components / CSS gradients / complex SVG), switch to the PDF path (`export_deck_pdf.mjs` / `export_deck_stage_pdf.mjs`) instead — **don't** expect pptx export to achieve both visual fidelity and editability. This is a physical constraint of the PPTX file format itself (see "Why the 4 Constraints Are Not Bugs But Physical Constraints" at the end).

---

## Canvas Size: Use 960×540pt (LAYOUT_WIDE)

PPTX units are **inches** (physical dimensions), not px. Decision principle: body's computedStyle dimensions must **match the presentation layout's inch dimensions** (±0.1", enforced by `html2pptx.js`'s `validateDimensions`).

### 3 Candidate Size Comparison

| HTML body | Physical size | Corresponding PPT layout | When to use |
|---|---|---|---|
| **`960pt × 540pt`** | **13.333″ × 7.5″** | **pptxgenjs `LAYOUT_WIDE`** | ✅ **Default recommended** (modern PowerPoint 16:9 standard) |
| `720pt × 405pt` | 10″ × 5.625″ | Custom | Only when user specifies "old PowerPoint Widescreen" template |
| `1920px × 1080px` | 20″ × 11.25″ | Custom | ❌ Non-standard size, fonts appear abnormally small after projection |

**Don't think of HTML size as resolution.** PPTX is a vector document — body size determines **physical dimensions**, not clarity. A large body (20″×11.25″) won't make text clearer — it only makes the pt font size small relative to the canvas, which looks worse when projected/printed.

### Three Equivalent body Writing Options

```css
body { width: 960pt;  height: 540pt; }    /* Clearest, recommended */
body { width: 1280px; height: 720px; }    /* Equivalent, px habit */
body { width: 13.333in; height: 7.5in; }  /* Equivalent, inch intuition */
```

Corresponding pptxgenjs code:

```js
const pptx = new pptxgen();
pptx.layout = 'LAYOUT_WIDE';  // 13.333 × 7.5 inch, no custom definition needed
```

---

## 4 Hard Constraints (Violation Causes Direct Error)

`html2pptx.js` translates the HTML DOM element-by-element into PowerPoint objects. PowerPoint's format constraints projected onto HTML = the 4 rules below.

### Rule 1: No Text Directly in DIV — Must Use `<p>` or `<h1>`-`<h6>` to Wrap

```html
<!-- ❌ Wrong: text directly in div -->
<div class="title">Q3 Revenue Grew 23%</div>

<!-- ✅ Correct: text in <p> or <h1>-<h6> -->
<div class="title"><h1>Q3 Revenue Grew 23%</h1></div>
<div class="body"><p>New users are the main driver</p></div>
```

**Why**: PowerPoint text must exist in a text frame, which corresponds to HTML's paragraph-level elements (p/h*/li). Bare `<div>` has no corresponding text container in PPTX.

**Also cannot use `<span>` to carry main text** — span is an inline element and cannot independently align as a text box. span can only be **nested inside p/h\*** for local styling (bold, color change).

### Rule 2: CSS Gradients Not Supported — Only Solid Colors

```css
/* ❌ Wrong */
background: linear-gradient(to right, #FF6B6B, #4ECDC4);

/* ✅ Correct: solid color */
background: #FF6B6B;

/* ✅ If multi-color stripes are required, use flex children each with solid color */
.stripe-bar { display: flex; }
.stripe-bar div { flex: 1; }
.red   { background: #FF6B6B; }
.teal  { background: #4ECDC4; }
```

**Why**: PowerPoint's shape fill only supports solid/gradient-fill two types, but pptxgenjs's `fill: { color: ... }` only maps solid. Gradients in PowerPoint's native gradient format require different structure — the current toolchain doesn't support it.

### Rule 3: Background/Border/Shadow Only on DIV, Not on Text Tags

```html
<!-- ❌ Wrong: <p> has background color -->
<p style="background: #FFD700; border-radius: 4px;">Important Content</p>

<!-- ✅ Correct: outer div carries background/border, <p> only handles text -->
<div style="background: #FFD700; border-radius: 4px; padding: 8pt 12pt;">
  <p>Important Content</p>
</div>
```

**Why**: In PowerPoint, shape (rectangle/rounded rectangle) and text frame are two separate objects. HTML's `<p>` only translates to a text frame; background/border/shadow belong to the shape — they must be written on the **div that wraps the text**.

### Rule 4: DIV Cannot Use `background-image` — Use `<img>` Tag

```html
<!-- ❌ Wrong -->
<div style="background-image: url('chart.png')"></div>

<!-- ✅ Correct -->
<img src="chart.png" style="position: absolute; left: 50%; top: 20%; width: 300pt; height: 200pt;" />
```

**Why**: `html2pptx.js` only extracts image paths from `<img>` elements — it does not parse CSS `background-image` URLs.

---

## Path A HTML Template Skeleton

Each slide is an independent HTML file with isolated scope (avoiding CSS pollution from single-file decks).

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<style>
  * { margin: 0; padding: 0; box-sizing: border-box; }
  body {
    width: 960pt; height: 540pt;           /* ⚠️ Match LAYOUT_WIDE */
    font-family: system-ui, -apple-system, "PingFang SC", sans-serif;
    background: #FEFEF9;                    /* Solid color, no gradients */
    overflow: hidden;
  }
  /* DIV handles layout/background/border */
  .card {
    position: absolute;
    background: #1A4A8A;                    /* Background on DIV */
    border-radius: 4pt;
    padding: 12pt 16pt;
  }
  /* Text tags only handle font styling, no background/border */
  .card h2 { font-size: 24pt; color: #FFFFFF; font-weight: 700; }
  .card p  { font-size: 14pt; color: rgba(255,255,255,0.85); }
</style>
</head>
<body>

  <!-- Title area: outer div positions, inner text tags -->
  <div style="position: absolute; top: 40pt; left: 60pt; right: 60pt;">
    <h1 style="font-size: 36pt; color: #1A1A1A; font-weight: 700;">Title uses declarative sentences, not topic words</h1>
    <p style="font-size: 16pt; color: #555555; margin-top: 10pt;">Subtitle with supplementary explanation</p>
  </div>

  <!-- Content card: div handles background, h2/p handle text -->
  <div class="card" style="top: 130pt; left: 60pt; width: 240pt; height: 160pt;">
    <h2>Point One</h2>
    <p>Brief explanatory text</p>
  </div>

  <!-- List: use ul/li, not manual • symbols -->
  <div style="position: absolute; top: 320pt; left: 60pt; width: 540pt;">
    <ul style="font-size: 16pt; color: #1A1A1A; padding-left: 24pt; list-style: disc;">
      <li>First key point</li>
      <li>Second key point</li>
      <li>Third key point</li>
    </ul>
  </div>

  <!-- Illustration: use <img> tag, not background-image -->
  <img src="illustration.png" style="position: absolute; right: 60pt; top: 110pt; width: 320pt; height: 240pt;" />

</body>
</html>
```

---

## Common Error Quick Reference

| Error message | Cause | Fix |
|---------|------|---------|
| `DIV element contains unwrapped text "XXX"` | Bare text in div | Wrap text in `<p>` or `<h1>`-`<h6>` |
| `CSS gradients are not supported` | Used linear/radial-gradient | Change to solid color, or use flex children for segmentation |
| `Text element <p> has background` | `<p>` tag has background color | Wrap in outer `<div>` for background, `<p>` only writes text |
| `Background images on DIV elements are not supported` | div uses background-image | Change to `<img>` tag |
| `HTML content overflows body by Xpt vertically` | Content exceeds 540pt | Reduce content or decrease font size, or use `overflow: hidden` to clip |
| `HTML dimensions don't match presentation layout` | body size doesn't match presentation layout | Use `960pt × 540pt` for body with `LAYOUT_WIDE`; or defineLayout for custom size |
| `Text box "XXX" ends too close to bottom edge` | Large `<p>` is less than 0.5 inch from body bottom edge | Move it up, leave adequate bottom margin; PPT itself will have some bottom covered |

---

## Basic Workflow (3 Steps to PPTX)

### Step 1: Write Each Page as Independent HTML Per Constraints

```
My Deck/
├── slides/
│   ├── 01-cover.html    # Each file is a complete 960×540pt HTML
│   ├── 02-agenda.html
│   └── ...
└── illustration/        # Images referenced by all <img> tags
    ├── chart1.png
    └── ...
```

### Step 2: Write build.js to Call `html2pptx.js`

```js
const pptxgen = require('pptxgenjs');
const html2pptx = require('../scripts/html2pptx.js');  // this skill's script

(async () => {
  const pres = new pptxgen();
  pres.layout = 'LAYOUT_WIDE';  // 13.333 × 7.5 inch, matches HTML's 960×540pt

  const slides = ['01-cover.html', '02-agenda.html', '03-content.html'];
  for (const file of slides) {
    await html2pptx(`./slides/${file}`, pres);
  }

  await pres.writeFile({ fileName: 'deck.pptx' });
})();
```

### Step 3: Open and Inspect

- Open exported PPTX in PowerPoint/Keynote
- Double-click any text — should be directly editable (if it's an image, Rule 1 was violated)
- Verify overflow: each page should be within body bounds, not cropped

---

## This Path vs Other Options (When to Choose What)

| Need | Choose |
|------|------|
| Colleagues will edit PPTX text / send to non-technical people for continued editing | **This path** (editable, requires HTML written per 4 constraints from the start) |
| Just for presentation / archive, no more editing | `export_deck_pdf.mjs` (multi-file) or `export_deck_stage_pdf.mjs` (single-file deck-stage), output vector PDF |
| Visual freedom is priority (animations, web components, CSS gradients, complex SVG), accepting non-editable | **PDF** (same as above) — PDF is both faithful and cross-platform, more appropriate than "image PPTX" |

**Never run html2pptx on a visually-free HTML and expect it to pass** — real testing shows visual-driven HTML has <30% pass rate; the remaining per-page fixes are slower than rewriting. This scenario should output PDF, not force PPTX.

---

## Fallback: Have Visual Draft But User Insists on Editable PPTX

Occasionally you'll encounter this scenario: you/the user has already written a visually-driven HTML (gradients, web components, complex SVG all used), PDF would be most appropriate, but the user explicitly says "no, must be editable PPTX."

**Don't run `html2pptx` expecting it to pass** — real testing shows visual-driven HTML has <30% pass rate on html2pptx, the remaining 70% will error or look wrong. The correct fallback is:

### Step 1 · First Communicate the Limitations (Transparent Communication)

Tell the user three things in one sentence:

> "Your current HTML uses [list specifically: gradients / web components / complex SVG / ...], which will fail to convert to editable PPTX directly. I have two options:
> - A. **Output PDF** (recommended) — visual is 100% preserved, recipients can view and print but can't edit text
> - B. **Rewrite an editable HTML based on your visual draft** (preserving design decisions on color/layout/copy, but reorganizing HTML structure per the 4 hard constraints, **sacrificing** visual capabilities like gradients, web components, complex SVG) → then export editable PPTX
>
> Which do you prefer?"

Don't downplay option B — clearly state **what will be lost**. Let the user make the tradeoff.

### Step 2 · If User Chooses B: AI Actively Rewrites, Don't Require the User to Write Themselves

The doctrine here is: **the user provides design intent; you're responsible for translating it into compliant implementation**. Not asking the user to learn the 4 hard constraints and rewrite themselves.

Principles for rewriting:
- **Preserve**: Color system (primary/secondary/neutral), information hierarchy (heading/subtitle/body/annotation), core copy, layout skeleton (top-middle-bottom / left-right columns / grid), page rhythm
- **Downgrade**: CSS gradients → solid color or flex segments; web components → paragraph-level HTML; complex SVG → simplified `<img>` or solid geometric; shadows → deleted or reduced to very subtle; custom fonts → toward system fonts
- **Rewrite**: Bare text → wrapped in `<p>` / `<h*>`; `background-image` → `<img>` tag; background/border on `<p>` → outer div carries it

### Step 3 · Produce a Comparison List (Transparent Delivery)

After rewriting, give the user a before/after comparison so they know which visual details were simplified:

```
Original design → editable version adjustments
- Title area purple gradient → primary color #5B3DE8 solid background
- Data card shadow → removed (changed to 2pt outline for separation)
- Complex SVG line chart → simplified to <img> PNG (screenshot from HTML)
- Hero area web component animation → static first frame (web components cannot be translated)
```

### Step 4 · Export & Dual-format Delivery

- `editable` HTML version → run `scripts/export_deck_pptx.mjs` for editable PPTX
- **Recommend also keeping** original visual draft → run `scripts/export_deck_pdf.mjs` for high-fidelity PDF
- Deliver both formats to user: high-fidelity PDF from visual draft + editable PPTX, each serving its purpose

### When to Outright Decline Option B

In certain scenarios rewriting is too costly — advise the user to abandon editable PPTX:
- HTML's core value is animation or interaction (after rewriting only a static first frame remains — information loss >50%)
- Pages > 30, rewriting cost exceeds 2 hours
- Visual design deeply depends on precise SVG / custom filters (after rewriting it's nearly unrelated to the original)

In these cases, tell the user: "This deck is too costly to rewrite. I recommend outputting PDF instead of PPTX. If recipients truly need pptx format, accept that the visuals will be significantly simplified — want to switch to PDF?"

---

## Why the 4 Constraints Are Not Bugs But Physical Constraints

These 4 constraints are not the `html2pptx.js` author being lazy — they are the constraints of the **PowerPoint file format (OOXML) itself** projected onto HTML:

- PPTX text must be in a text frame (`<a:txBody>`), corresponding to HTML's paragraph-level elements
- PPTX shape and text frame are two separate objects — cannot simultaneously have background drawn and text written on the same element
- PPTX shape fill has limited gradient support (only certain preset gradients, doesn't support arbitrary CSS angle gradients)
- PPTX picture objects must reference real image files, not CSS properties

Understanding this, **don't expect the tool to become smarter** — HTML writing must adapt to PPTX format, not the other way around.
