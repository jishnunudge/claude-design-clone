# Slide Decks: HTML Slide Deck Production Standards

High-frequency design task. Covers HTML slide decks — architecture selection, single-page design, complete PDF/PPTX export path.

**Skill capability coverage**:
- **HTML presentation version (baseline deliverable, always required)** → Each page as independent HTML + `assets/deck_index.html` aggregation, keyboard navigation in browser, full-screen presentation
- HTML → PDF export → `scripts/export_deck_pdf.mjs` / `scripts/export_deck_stage_pdf.mjs`
- HTML → Editable PPTX export → `references/editable-pptx.md` + `scripts/html2pptx.js` + `scripts/export_deck_pptx.mjs` (requires HTML written to 4 hard constraints)

> **⚠️ HTML is the foundation, PDF/PPTX are derivatives.** Regardless of final delivery format, **always** create the HTML aggregated presentation version (`index.html` + `slides/*.html`) first. PDF/PPTX are snapshots exported from HTML with a single command.
>
> **Why HTML-first**:
> - Best for presentation scenarios (projector / screen share full-screen directly, keyboard navigation, no Keynote/PPT dependency)
> - Each page can be opened individually for verification during development — no re-running export each time
> - Only upstream for PDF/PPTX export (avoids "found changes needed after export, modify HTML then re-export" dead loop)
> - Deliverable can be "HTML + PDF" or "HTML + PPTX" — recipient uses whichever they prefer
>
> 2026-04-22 moxt brochure real test: After completing 13-page HTML + index.html aggregation, `export_deck_pdf.mjs` exported PDF in one command, zero changes. HTML version itself is a deliverable presentable directly in browser.

---

## 🛑 Confirm Delivery Format Before Starting (Hardest Checkpoint)

**This decision comes before "single-file vs multi-file."** 2026-04-20 real test: **Not confirming delivery format before starting = 2-3 hours of rework.**

### Decision Tree (HTML-first Architecture)

All deliverables start from the same HTML aggregated page (`index.html` + `slides/*.html`). Delivery format only determines **HTML writing constraints** and **export commands**:

```
【Always default · required】 HTML aggregated presentation version (index.html + slides/*.html)
   │
   ├── Just browser presentation / local HTML archive   → done here, HTML has maximum visual freedom
   │
   ├── Also need PDF (print / share / archive)          → run export_deck_pdf.mjs one-command output
   │                                                       HTML writing free, no visual constraints
   │
   └── Also need editable PPTX (colleagues to edit)    → write HTML per 4 hard constraints from line one
                                                          run export_deck_pptx.mjs one-command output
                                                          sacrifice gradients / web components / complex SVG
```

### Opening Script (Copy and Use)

> Regardless of whether final delivery is HTML, PDF, or PPTX, I'll first create an HTML aggregated version for browser presentation (`index.html` with keyboard navigation) — always the default baseline deliverable. Then I'll ask whether you need additional PDF / PPTX snapshots.
>
> Which export format do you need?
> - **HTML only** (presentation/archive) → fully free visually
> - **Also PDF** → same, plus one export command
> - **Also editable PPTX** (colleagues will edit text in PPT) → must write HTML per 4 hard constraints from the first line, sacrificing some visual capabilities (no gradients, no web components, no complex SVG).

### Why "Wanting PPTX Means 4 Hard Constraints From the Start"

PPTX editability requires `html2pptx.js` to translate DOM elements one-by-one into PowerPoint objects. Requires **4 hard constraints**:

1. body fixed at 960pt × 540pt (matching `LAYOUT_WIDE`, 13.333″ × 7.5″, not 1920×1080px)
2. All text wrapped in `<p>`/`<h1>`-`<h6>` (prohibit div directly containing text, prohibit `<span>` carrying main text)
3. `<p>`/`<h*>` cannot have background/border/shadow (put on outer div)
4. `<div>` cannot use `background-image` (use `<img>` tag)
5. No CSS gradients, web components, or complex SVG decorations

**This skill's default HTML has high visual freedom** — lots of spans, nested flex, complex SVG, web components (like `<deck-stage>`), CSS gradients — **almost none naturally pass html2pptx constraints** (real test: visually-driven HTML directly through html2pptx has <30% pass rate).

### Real Path Cost Comparison (2026-04-20 real trap)

| Path | Approach | Result | Cost |
|------|------|------|------|
| ❌ **Write HTML freely first, then fix PPTX** | Single-file deck-stage + lots of SVG/span decorations | Want editable PPTX — only two options left:<br>A. Hand-write pptxgenjs hundreds of lines of hardcoded coordinates<br>B. Rewrite 17 pages of HTML to Path A format | 2-3 hours of rework, and hand-written version **carries permanent maintenance cost** |
| ✅ **Write to Path A constraints from step one** | Each page independent HTML + 4 hard constraints + 960×540pt | One command exports 100% editable PPTX, also browser-presentable full-screen | 5 extra minutes thinking "how to wrap text in `<p>`" — zero rework |

### Mixed Delivery

User says "I want HTML presentation **and** editable PPTX" — **not a mix**, PPTX requirement covers HTML requirement. Path A HTML itself can be presented full-screen in browser (add a `deck_index.html` aggregator). **No additional cost.**

User says "I want PPTX **and** animations / web components" — **real contradiction**. Tell the user: wanting editable PPTX means sacrificing these visual capabilities. Let them choose — don't secretly do hand-written pptxgenjs (permanent maintenance debt).

### PPTX Required After the Fact (Emergency Fallback)

Rare case: HTML already written before discovering PPTX is needed. Recommended **fallback process** (full description in `references/editable-pptx.md` "Fallback: Already have a visual draft but user insists on editable PPTX"):

1. **First choice: Convert to PDF** (100% visual preservation, cross-platform)
2. **Second choice: AI rewrites editable HTML based on visual draft** → export editable PPTX — preserves color/layout/copy decisions, sacrifices gradients, web components, complex SVG
3. **Not recommended: Hand-write pptxgenjs from scratch** — high maintenance cost; every HTML change requires manual PPTX re-sync

Always tell the user the options and let them decide. **Never first-instinct start hand-writing pptxgenjs** — last-resort fallback only.

---

## 🛑 Before Batch Production: Make 2-page Showcase to Define Grammar

**For any deck ≥ 5 pages, never write straight through from page 1 to the last.** Validated correct order from 2026-04-22 moxt brochure:

1. Choose **2 page types with greatest visual difference** for showcase first (e.g., "cover" + "mood/quote page")
2. Screenshot and let user confirm grammar (masthead / fonts / colors / spacing / structure / bilingual proportion)
3. After direction approved, batch-produce remaining N-2 pages, each reusing established grammar
4. After all done, assemble HTML aggregation + PDF / PPTX derivatives

**Why**: Writing straight through 13 pages → user says "direction is wrong" = 13 rewrites. First make 2-page showcase → direction wrong = 2 rewrites. Once visual grammar is established, subsequent N pages only need "how to place the content."

**Showcase page selection**: Choose the two pages with the most different visual structures. These two pass = all intermediate states pass.

| Deck Type | Recommended Showcase Pages |
|-----------|---------------------|
| B2B brochure / product launch | Cover + content page (concept/emotional page) |
| Brand launch | Cover + product features page |
| Data report | Large data visualization page + analytical conclusion page |
| Tutorial course | Chapter opening page + specific knowledge point page |

---

## 📐 Publication Grammar Template (Tested and Reusable from moxt)

Suitable for B2B brochure / product launch / long report decks. Every page reusing this structure = 13 pages visually consistent, 0 rework.

### Per-page Skeleton

```
┌─ masthead (top strip + horizontal line) ────────────┐
│  [logo 22-28px] · A Product Brochure                Issue · Date · URL │
├──────────────────────────────────────────┤
│                                          │
│  ── kicker (green short bar + uppercase label)   │
│  CHAPTER XX · SECTION NAME                 │
│                                          │
│  H1 (Chinese Noto Serif SC 900)             │
│  key words individually in brand primary color      │
│                                          │
│  English subtitle (Lora italic, subtitle)   │
│  ─────────── divider ──────────            │
│                                          │
│  [specific content: dual-column 60/40 / 2x2 grid / list] │
│                                          │
├──────────────────────────────────────────┤
│ section name                     XX / total │
└──────────────────────────────────────────┘
```

### Style Conventions (Copy Directly)

- **H1**: Chinese Noto Serif SC 900, font size 80-140px depending on information volume; key words individually in brand primary color (don't stack color on entire text)
- **English subtitle**: Lora italic 26-46px; brand signature words (e.g., "AI team") bold + primary color italic
- **Body**: Noto Serif SC 17-21px, line-height 1.75-1.85
- **Accent highlight**: Primary color bold for key words in body text, max 3 per page (too many loses anchor effect)
- **Background**: Warm cream base #FAFAFA + very subtle radial-gradient noise (`rgba(33,33,33,0.015)`) for paper feel

### Visual Lead Must Be Differentiated

13 pages all "text + one screenshot" is too monotonous. **Rotate visual lead type per page**:

| Visual Type | Suitable Section |
|---------|---------------|
| Cover typography (large text + masthead + pillar) | First page / chapter opening |
| Single character portrait (super large single momo, etc.) | Introducing single concept/character |
| Group portrait / avatar cards side by side | Team / user cases |
| Timeline card progression | Showing "long-term relationship" "evolution" |
| Knowledge graph / connected node diagram | Showing "collaboration" "flow" |
| Before/After comparison card + center arrow | Showing "change" "difference" |
| Product UI screenshot + outlined device frame | Specific feature showcase |
| Large-quote big-quote (half-page large text) | Mood page / question page / quote page |
| Real person portrait + quote card (2×2 or 1×4) | User testimonials / usage scenarios |
| Large text back cover + URL oval button | CTA / ending |

---

## ⚠️ Common Pitfalls (moxt Production Summary)

### 1. Emoji Don't Render in Chromium / Playwright Export

Chromium doesn't include color emoji fonts by default. `page.pdf()` or `page.screenshot()` shows emoji as empty boxes.

**Solution**: Use Unicode text symbols (`✦` `✓` `✕` `→` `·` `—`) instead, or plain text ("Email · 23" not "📧 23 emails").

### 2. `export_deck_pdf.mjs` Errors: `Cannot find package 'playwright'`

Cause: ESM module resolution looks for `node_modules` upward from script location. Script is in `~/.claude/skills/huashu-design/scripts/`, which has no dependencies.

**Solution**: Copy script to deck project directory (e.g., `brochure/build-pdf.mjs`), run `npm install playwright pdf-lib` at project root, then `node build-pdf.mjs --slides slides --out output/deck.pdf`.

### 3. Google Fonts Not Loaded Before Screenshot → Chinese Displays as System Heiti

Use at least `wait-for-timeout=3500` before Playwright screenshot/PDF to let webfonts download and paint. Or self-host fonts to `shared/fonts/`.

### 4. Information Density Imbalance: Content Pages Overstuffed

moxt philosophy page first version: 2×2 = 4 paragraphs + 3 bottom creeds = 7 content blocks, cramped and repetitive. Changing to 1×3 = 3 paragraphs immediately restored breathing feel.

**Solution**: Each page limited to "1 core message + 3-4 supporting points + 1 visual lead." More than that, split to new page. **Less is more** — audience has 10 seconds per page; 1 memorable point beats 4.

---

## 🛑 Define Architecture First: Single-file or Multi-file?

**This choice is the first step — getting it wrong leads to repeated pitfalls. Read this section before starting.**

### Two Architecture Comparison

| Dimension | Single-file + `deck_stage.js` | **Multi-file + `deck_index.html` aggregator** |
|------|--------------------------|--------------------------------------|
| Code structure | One HTML, all slides are `<section>` | Each page independent HTML, `index.html` aggregates with iframes |
| CSS scope | ❌ Global, one page's styles can affect all pages | ✅ Naturally isolated, each iframe has its own namespace |
| Validation granularity | ❌ Need JS goTo to switch to specific page | ✅ Single page file double-click to view in browser |
| Parallel development | ❌ One file, multiple agents editing will conflict | ✅ Multiple agents can work on different pages simultaneously, zero-conflict merge |
| Debugging difficulty | ❌ One CSS error, entire deck breaks | ✅ One page error only affects itself |
| Embedded interactions | ✅ Cross-page shared state is simple | 🟡 Between iframes requires postMessage |
| Print to PDF | ✅ Built-in | ✅ Aggregator beforeprint traverses iframes |
| Keyboard navigation | ✅ Built-in | ✅ Built into aggregator |

### Which to Choose? (Decision Tree)

```
│ Question: How many pages?
├── ≤10 pages, need in-deck animations or cross-page interactions, pitch deck → single-file
└── ≥10 pages, academic lecture, courseware, long deck, multi-agent parallel → multi-file (recommended)
```

**Default to multi-file path**. It's not a "backup option" — it's **the main path for long decks and team collaboration**. Every advantage of single-file (keyboard navigation, printing, scale) is also available in multi-file, while multi-file's scope isolation and verifiability cannot be compensated by single-file.

### Why Is This Rule So Hard? (Real Incident Record)

Single-file architecture hit four consecutive traps making an AI psychology lecture deck:

1. **CSS specificity override**: `.emotion-slide { display: grid }` (specificity 10) overrode `deck-stage > section { display: none }` (specificity 2), causing all pages to render stacked simultaneously.
2. **Shadow DOM slot rules suppressed by outer CSS**: `::slotted(section) { display: none }` couldn't block outer rule overrides, sections refused to hide.
3. **localStorage + hash navigation race condition**: After refresh, stopped at old localStorage position instead of hash position.
4. **High validation cost**: Had to `page.evaluate(d => d.goTo(n))` to screenshot a specific page — twice as slow as `goto(file://.../slides/05-X.html)` and frequently errored.

All root causes are the **single global namespace** — multi-file architecture eliminates these problems at the physical level.

---

## Path A (Default): Multi-file Architecture

### Directory Structure

```
My Deck/
├── index.html              # Copied from assets/deck_index.html, modify MANIFEST
├── shared/
│   ├── tokens.css          # Shared design tokens (color palette/font sizes/common chrome)
│   └── fonts.html          # <link> to import Google Fonts (each page includes)
└── slides/
    ├── 01-cover.html       # Each file is a complete 1920×1080 HTML
    ├── 02-agenda.html
    ├── 03-problem.html
    └── ...
```

### Template Skeleton for Each Slide

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<title>P05 · Chapter Title</title>
<link href="https://fonts.googleapis.com/css2?family=..." rel="stylesheet">
<link rel="stylesheet" href="../shared/tokens.css">
<style>
  /* Styles unique to this page. Any class names won't pollute other pages. */
  body { padding: 120px; }
  .my-thing { ... }
</style>
</head>
<body>
  <!-- 1920×1080 content (body width/height locked in tokens.css) -->
  <div class="page-header">...</div>
  <div>...</div>
  <div class="page-footer">...</div>
</body>
</html>
```

**Key constraints**:
- `<body>` is the canvas — layout directly on it. Don't wrap in `<section>` or other wrappers.
- `width: 1920px; height: 1080px` locked by the `body` rule in `shared/tokens.css`.
- Import `shared/tokens.css` for shared design tokens (color palette, font sizes, page-header/footer).
- Font `<link>` written per page (cheap, ensures each page opens independently).

### Aggregator: `deck_index.html`

**Copy directly from `assets/deck_index.html`**. Only change the `window.DECK_MANIFEST` array — list all slide filenames and labels in order:

```js
window.DECK_MANIFEST = [
  { file: "slides/01-cover.html",    label: "Cover" },
  { file: "slides/02-agenda.html",   label: "Agenda" },
  { file: "slides/03-problem.html",  label: "Problem Statement" },
  // ...
];
```

Built-in: keyboard navigation (←/→/Home/End/number keys/P to print), scale + letterbox, bottom-right counter, localStorage memory, hash page jumping, print mode (traverses iframes to output PDF page by page).

### Single-page Validation (Killer Advantage of Multi-file)

Each slide is independent HTML. **After finishing one, double-click in browser to check**:

```bash
open slides/05-personas.html
```

Playwright screenshots also directly `goto(file://.../slides/05-personas.html)` — no JS page navigation, no CSS interference from other pages. Makes "change a little, verify a little" near zero cost.

### Parallel Development

Split each slide's task to different agents running simultaneously — HTML files are independent, no conflicts at merge. Long decks compress production time to 1/N.

### What Goes in `shared/tokens.css`

Only **truly shared across pages**:

- CSS variables (color palette, font size scale, spacing scale)
- `body { width: 1920px; height: 1080px; }` canvas locking
- `.page-header` / `.page-footer` chrome used identically across all pages

**Don't** put single-page layout classes here — that would regress to single-file global pollution.

---

## Path B (Small Deck): Single-file + `deck_stage.js`

Suitable for ≤10 pages, needing cross-page shared state (e.g., a React tweaks panel controlling all pages), or pitch decks requiring extreme compactness.

### Basic Usage

1. Read content from `assets/deck_stage.js`, embed in HTML's `<script>` (or `<script src="deck_stage.js">`)
2. Use `<deck-stage>` in body to wrap slides
3. 🛑 **script tag must be placed after `</deck-stage>`** (see hard constraint below)

```html
<body>

  <deck-stage>
    <section>
      <h1>Slide 1</h1>
    </section>
    <section>
      <h1>Slide 2</h1>
    </section>
  </deck-stage>

  <!-- ✅ Correct: script after deck-stage -->
  <script src="deck_stage.js"></script>

</body>
```

### 🛑 Script Position Hard Constraint (Real trap from 2026-04-20)

**Cannot put `<script src="deck_stage.js">` in `<head>`.** Even if it can define `customElements` in `<head>`, the parser triggers `connectedCallback` when it encounters the `<deck-stage>` opening tag — child `<section>` elements haven't been parsed yet, `_collectSlides()` gets empty array, counter shows `1 / 0`, all pages render stacked.

**Three compliant writing approaches** (choose one):

```html
<!-- ✅ Most recommended: script after </deck-stage> -->
</deck-stage>
<script src="deck_stage.js"></script>

<!-- ✅ Also acceptable: script in head with defer -->
<head><script src="deck_stage.js" defer></script></head>

<!-- ✅ Also acceptable: module scripts are naturally deferred -->
<head><script src="deck_stage.js" type="module"></script></head>
```

`deck_stage.js` has a `DOMContentLoaded` deferred collection defense built in, so even if script is in head it won't completely break — but `defer` or body bottom is still cleaner.

### ⚠️ Single-file Architecture CSS Trap (Must Read)

Most common trap — **`display` property being stolen by single-page styles**.

Common mistake 1 (writing display: flex directly to section):

```css
/* ❌ Outer CSS specificity 2, overrides shadow DOM's ::slotted(section){display:none} (also 2) */
deck-stage > section {
  display: flex;            /* All pages will render stacked simultaneously! */
  flex-direction: column;
  padding: 80px;
  ...
}
```

Common mistake 2 (section has class with higher specificity):

```css
.emotion-slide { display: grid; }   /* Specificity: 10, even worse */
```

Both cause **all slides to render stacked simultaneously** — counter might show `1 / 10` looking normal, but visually page 1 overlays page 2 overlays page 3.

### ✅ Starter CSS (Copy Directly to Avoid Pitfalls)

**section itself** only manages "visible/hidden"; **layout (flex/grid etc.) goes on `.active`**:

```css
/* section only defines non-display general styles */
deck-stage > section {
  background: var(--paper);
  padding: 80px 120px;
  overflow: hidden;
  position: relative;
  /* ⚠️ Don't write display here! */
}

/* Lock "inactive = hidden" — specificity + !important double insurance */
deck-stage > section:not(.active) {
  display: none !important;
}

/* Active page writes needed display + layout */
deck-stage > section.active {
  display: flex;
  flex-direction: column;
  justify-content: center;
}

/* Print mode: all pages need to show, override :not(.active) */
@media print {
  deck-stage > section { display: flex !important; }
  deck-stage > section:not(.active) { display: flex !important; }
}
```

Alternative: **Put single-page flex/grid on inner wrapper `<div>`** — section itself is always just a `display: block/none` switch. Cleanest approach:

```html
<deck-stage>
  <section>
    <div class="slide-content flex-layout">...</div>
  </section>
</deck-stage>
```

### Custom Dimensions

```html
<deck-stage width="1080" height="1920">
  <!-- 9:16 portrait -->
</deck-stage>
```

---

## Slide Labels

Both deck_stage and deck_index label each page (counter display). Give them **more meaningful** labels:

**Multi-file**: Write `{ file, label: "04 Problem Statement" }` in `MANIFEST`
**Single-file**: Add `<section data-screen-label="04 Problem Statement">` to the section

**Key: Slide numbers start from 1, not 0**. When users say "slide 5", they mean the 5th slide, never array position `[4]`.

---

## Speaker Notes

**Not added by default** — only when explicitly requested.

With speaker notes you can reduce slide text to minimum, focus on impactful visuals — notes carry complete script.

### Format

**Multi-file**: In `index.html`'s `<head>`:

```html
<script type="application/json" id="speaker-notes">
[
  "Script for slide 1...",
  "Script for slide 2...",
  "..."
]
</script>
```

**Single-file**: Same location.

### Notes Writing Points

- **Complete**: Not an outline, but the actual words to be spoken
- **Conversational**: Like everyday speech, not written language
- **Corresponding**: Array item N corresponds to slide N
- **Length**: 200-400 words ideal
- **Emotional arc**: Mark emphasis, pauses, stress points

---

## Slide Design Patterns

### 1. Establish a System (Required)

After exploring design context, **verbally state the system you'll use first**:

```markdown
Deck system:
- Background color: max 2 (90% white + 10% dark section dividers)
- Typography: display uses Instrument Serif, body uses Geist Sans
- Rhythm: section dividers use full-bleed color + white text; regular slides use white background
- Images: hero slides use full-bleed photos; data slides use charts

I'll build to this system — tell me if there's an issue.
```

Confirm with user before proceeding.

### 2. Common Slide Layouts

- **Title slide**: Solid color background + large title + subtitle + author/date
- **Section divider**: Colored background + chapter number + chapter title
- **Content slide**: White background + heading + 1-3 bullet points
- **Data slide**: Heading + large chart/number + brief description
- **Image slide**: Full-bleed photo + small caption at bottom
- **Quote slide**: White space + large quote + attribution
- **Two-column**: Left-right comparison (vs / before-after / problem-solution)

Max 4-5 layouts in one deck.

### 3. Scale (Worth Repeating)

- Body minimum **24px**, ideal 28-36px
- Heading **60-120px**
- Hero text **180-240px**
- Slides viewed from 10 meters away — text must be large enough

### 4. Visual Rhythm

Decks need **intentional variety**:

- Color rhythm: mostly white background + occasional colored section dividers + occasional dark segments
- Density rhythm: a few text-heavy pages + a few image-heavy pages + a few quote white-space pages
- Font size rhythm: normal headings + occasional giant hero text

**Don't make every slide look the same** — that's a PPT template, not design.

### 5. Breathing Space (Must Read for Data-dense Pages)

**Easiest beginner trap**: stuffing all possible information into one page.

Information density ≠ effective information delivery. Academic/presentation decks especially need restraint:

- Lists/matrix pages: don't make N elements all the same size. Use **hierarchy** — the 5 being discussed today enlarged as leads; remaining 16 reduced as background hints.
- Large number pages: number itself is visual lead. Surrounding caption max 3 lines, otherwise audience eyes jump around.
- Quote pages: leave whitespace between quote and attribution.

Use two self-audit checks: "Is data the lead?" and "Is text crammed?" — revise until the whitespace makes you slightly uncomfortable.

---

## Print to PDF

**Multi-file**: `deck_index.html` already handles `beforeprint` event, outputs PDF page by page.

**Single-file**: `deck_stage.js` handles similarly.

Print styles already written — no additional `@media print` CSS needed.

---

## Export to PPTX / PDF (Self-service Scripts)

HTML-first is primary. Two general-purpose scripts in `scripts/`, **usable for any multi-file deck**:

### `export_deck_pdf.mjs` — Export Vector PDF (Multi-file Architecture)

```bash
node scripts/export_deck_pdf.mjs --slides <slides-dir> --out deck.pdf
```

**Features**:
- Text **remains vector** (copyable, searchable)
- Visual 100% fidelity (Playwright Chromium renders then prints)
- **No changes needed to HTML**
- Each slide uses independent `page.pdf()`, then merged with `pdf-lib`

**Dependencies**: `npm install playwright pdf-lib`

**Limitation**: PDF text cannot be edited — go back to HTML to make changes.

### `export_deck_stage_pdf.mjs` — Single-file deck-stage Architecture Specific ⚠️

**When to use**: Deck is single HTML file + `<deck-stage>` web component wrapping N `<section>` elements (Path B architecture). The "one `page.pdf()` per HTML" approach from `export_deck_pdf.mjs` doesn't work — use this dedicated script.

```bash
node scripts/export_deck_stage_pdf.mjs --html deck.html --out deck.pdf
```

**Why `export_deck_pdf.mjs` cannot be reused** (real trap from 2026-04-20):

1. **Shadow DOM beats `!important`**: deck-stage's shadow CSS has `::slotted(section) { display: none }` (only the active one shows `display: block`). Even `@media print { deck-stage > section { display: block !important } }` in light DOM can't override it — `page.pdf()` triggers print media, Chromium's final rendering only shows the active slide. Result: **entire PDF is only 1 page**.

2. **Looping goto each page still only outputs 1 page**: Intuitive fix "navigate to each `#slide-N` then `page.pdf({pageRanges:'1'})`" also fails — print CSS outside shadow DOM gets overridden, final rendering is always the first section list element. 17 iterations get 17 copies of P01 cover.

3. **Absolute child elements run to next page**: Sections with `position: static` will have absolute-positioned `cover-footer`/`slide-footer` positioned relative to initial containing block — absolute footers may be pushed to next page when sections are forced to 1080px height for print.

**Fix strategy** (already implemented in script):

```js
// After opening HTML, use page.evaluate to pull sections out of deck-stage slot,
// mount them directly under body in a regular div, inline style ensures position:relative + fixed dimensions
await page.evaluate(() => {
  const stage = document.querySelector('deck-stage');
  const sections = Array.from(stage.querySelectorAll(':scope > section'));
  document.head.appendChild(Object.assign(document.createElement('style'), {
    textContent: `
      @page { size: 1920px 1080px; margin: 0; }
      html, body { margin: 0 !important; padding: 0 !important; }
      deck-stage { display: none !important; }
    `,
  }));
  const container = document.createElement('div');
  sections.forEach(s => {
    s.style.cssText = 'width:1920px!important;height:1080px!important;display:block!important;position:relative!important;overflow:hidden!important;page-break-after:always!important;break-after:page!important;background:#F7F4EF;margin:0!important;padding:0!important;';
    container.appendChild(s);
  });
  // Last page disables page-break to avoid blank trailing page
  sections[sections.length - 1].style.pageBreakAfter = 'auto';
  sections[sections.length - 1].style.breakAfter = 'auto';
  document.body.appendChild(container);
});

await page.pdf({ width: '1920px', height: '1080px', printBackground: true, preferCSSPageSize: true });
```

**Why this works**:
- Pulls sections from shadow DOM slot to regular div in light DOM — bypasses `::slotted(section) { display: none }` rule
- Inline `position: relative` lets absolute children position relative to section, no overflow
- `page-break-after: always` makes browser print each section as independent page
- Last child doesn't break page, avoids trailing blank page

**Note when validating with `mdls -name kMDItemNumberOfPages`**: macOS Spotlight metadata is cached. After rewriting PDF, run `mdimport file.pdf` to force refresh, otherwise old page count is shown. Use `pdfinfo` or `pdftoppm` for true count.

---

### `export_deck_pptx.mjs` — Export Editable PPTX

```bash
# Only mode: text boxes are natively editable (fonts fall back to system fonts)
node scripts/export_deck_pptx.mjs --slides <dir> --out deck.pptx
```

How it works: `html2pptx` reads computedStyle element-by-element and translates DOM into PowerPoint objects (text frame / shape / picture). Text becomes real text boxes, directly editable with double-click in PPT.

**Hard constraints** (HTML must satisfy these, otherwise page is skipped — detailed in `references/editable-pptx.md`):
- All text must be in `<p>`/`<h1>`-`<h6>`/`<ul>`/`<ol>` (prohibit bare text divs)
- `<p>`/`<h*>` tags cannot have background/border/shadow (put on outer div)
- Don't use `::before`/`::after` to insert decorative text (pseudo-elements can't be extracted)
- Inline elements (span/em/strong) cannot have margin
- No CSS gradients (not renderable)
- Divs don't use `background-image` (use `<img>`)

Script has built-in **auto-preprocessor** — wraps "bare text in leaf divs" into `<p>` (preserving classes). Solves most common violation (bare text). Other violations (border on p, margin on span) still need HTML source compliance.

**Font fallback caveat**:
- Playwright uses webfonts to measure text-box dimensions; PowerPoint/Keynote renders with system fonts
- When different, **overflow or misalignment** results — visually check each page
- Recommend installing fonts used in HTML on target machine, or fall back to `system-ui`

**For visual-priority scenarios, don't take this path** → use `export_deck_pdf.mjs` for PDF. PDF is 100% visually faithful, vector, cross-platform, text searchable — the true destination for visual-priority decks.

### Make HTML Export-friendly From the Start

```html
<!-- ❌ Not good -->
<div class="title">Key Finding</div>

<!-- ✅ Good (p wrapper, class inherited) -->
<p class="title">Key Finding</p>

<!-- ❌ Not good (border on p) -->
<p class="stat" style="border-left: 3px solid red;">41%</p>

<!-- ✅ Good (border on outer div) -->
<div class="stat-wrap" style="border-left: 3px solid red;">
  <p class="stat">41%</p>
</div>
```

### When to Choose Which

| Scenario | Recommended |
|------|------|
| Giving to host / archiving | **PDF** (universal, high-fidelity, text searchable) |
| Sending to collaborators to fine-tune text | **PPTX editable** (accept font fallback) |
| Live presentation, no content changes | **PDF** (vector fidelity, cross-platform) |
| HTML is preferred presentation medium | Play directly in browser, export is backup |

## Deep Path for Editable PPTX (Long-term Projects Only)

If your deck will be maintained long-term, repeatedly modified, team-collaborated — **write HTML per html2pptx constraints from the beginning**, so `export_deck_pptx.mjs` can directly pass everything. See `references/editable-pptx.md` (4 hard constraints + HTML template + common error quick reference + fallback process).

---

## Common Issues

**Multi-file: Page in iframe can't open / white screen**
→ Check if `file` path in `MANIFEST` is correct relative to `index.html`. Use browser DevTools to see if iframe's src is accessible.

**Multi-file: Styles on one page conflict with another**
→ Impossible (iframe isolation). If it feels like a conflict, that's cache — Cmd+Shift+R hard refresh.

**Single-file: Multiple slides rendering stacked**
→ CSS specificity problem. See "Single-file Architecture CSS Trap" section above.

**Single-file: Scaling looks wrong**
→ Check all slides are direct `<section>` children under `<deck-stage>`. No wrapping `<div>` in between.

**Single-file: Want to jump to a specific slide**
→ Add hash to URL: `index.html#slide-5` jumps to slide 5.

**Both architectures: Text position inconsistent on different screens**
→ Use fixed dimensions (1920×1080) and `px` units — no `vw`/`vh` or `%`. Handle scaling uniformly.

---

## Validation Checklist (Required Before Finishing Deck)

1. [ ] Open `index.html` directly in browser, check first page has no broken images, fonts loaded
2. [ ] Press → key to navigate every page — no blank pages, no layout misalignment
3. [ ] Press P for print preview, each page exactly one A4 (or 1920×1080) with no cropping
4. [ ] Randomly choose 3 pages Cmd+Shift+R hard refresh, localStorage memory works correctly
5. [ ] Playwright batch screenshots (multi-page: traverse `slides/*.html`; single-file: use goTo), visually review all pages
6. [ ] Search for `TODO` / `placeholder` remnants, confirm all cleaned up
