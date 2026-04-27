# Content Guidelines: Anti-AI Slop, Content Standards, Scale Specs

Traps most easily fallen into in AI design. This is a "what not to do" checklist — more important than "what to do" — because AI slop is the default. Don't actively avoid it and it will happen.

## AI Slop Complete Blacklist

### Visual Traps

**❌ Aggressive gradient backgrounds**
- Purple → pink → blue full-screen gradients (signature of AI-generated web pages)
- Rainbow gradients in any direction
- Mesh gradients filling background
- ✅ Want gradients: subtle, monochromatic, used intentionally as accents (e.g., button hover)

**❌ Rounded cards + left border accent color**
```css
/* This is the signature of an AI-flavored card */
.card {
  border-radius: 12px;
  border-left: 4px solid #3b82f6;
  padding: 16px;
}
```
Rampant in AI-generated dashboards. Want emphasis? Use design-considered approaches: background color contrast, font weight/size contrast, plain dividers, or skip card containers altogether.

**❌ Emoji decoration**
Unless brand uses emoji (like Notion, Slack), don't put emoji in UI. **Especially avoid**:
- 🚀 ⚡️ ✨ 🎯 💡 before headings
- ✅ in feature lists
- → inside CTA buttons (standalone arrows OK; emoji arrows not)

No icons? Use real icon library (Lucide/Heroicons/Phosphor) or placeholder.

**❌ SVG imagery**
Don't use SVG to draw people, scenes, devices, objects, or abstract art. AI-drawn SVG is immediately recognizable — naive and cheap. **Gray rectangle with text label "Illustration placeholder 1200×800" is 100× better than clumsy SVG hero illustration**.

Acceptable SVG uses only:
- Real icons (16×16 to 32×32)
- Geometric shapes as decorative elements
- Data viz charts

**❌ Excessive iconography**
Not every heading / feature / section needs an icon. Overusing icons makes interface look like a toy.

**❌ "Data slop"**
Made-up stats as decoration:
- "10,000+ happy customers" (you don't know if that's true)
- "99.9% uptime" (don't write without real data)
- Decorative metric cards of icons + numbers + phrases
- Fake data in mock tables

No real data → leave placeholder or ask user.

**❌ "Quote slop"**
Made-up testimonials or celebrity quotes. Leave placeholder; ask user for real quotes.

### Typography Traps

**❌ Avoid these overused fonts**:
- Inter (default for AI-generated web pages)
- Roboto
- Arial / Helvetica
- Pure system font stacks
- Fraunces (AI discovered and overused)
- Space Grotesk (currently AI's favorite)

**✅ Use distinctive display + body pairings**:
- Serif display + sans body (editorial)
- Mono display + sans body (technical)
- Heavy display + light body (contrast)
- Variable font for weight animation in hero

Font resources:
- Underused Google Fonts gems (Instrument Serif, Cormorant, Bricolage Grotesque, JetBrains Mono)
- Open-source font sites
- Don't invent font names

### Color Traps

**❌ Inventing colors from scratch**
Designing entirely unfamiliar color sets usually turns out inharmonious.

**✅ Strategy**:
1. Has brand colors → use them; fill missing tokens using oklch interpolation
2. Has reference → sample colors from reference product screenshot
3. Starting from zero → pick known color system (Radix Colors / Tailwind default / Anthropic brand)

**oklch color definition** — most modern approach:
```css
:root {
  --primary: oklch(0.65 0.18 25);      /* warm terracotta */
  --primary-light: oklch(0.85 0.08 25); /* lighter in the same hue */
  --primary-dark: oklch(0.45 0.20 25);  /* darker in the same hue */
}
```
oklch ensures hue doesn't drift when adjusting lightness — more reliable than hsl.

**❌ Casually adding inverted dark mode**
Dark mode is not simple color inversion. Good dark mode requires re-tuning saturation, contrast, and accent colors. If unwilling to do that, don't add dark mode.

### Layout Traps

**❌ Bento grid overuse**
Every AI-generated landing page wants bento layout. Unless information structure genuinely suits bento, use different layout.

**❌ Big hero + 3-column features + testimonials + CTA**
This landing page template is done to death. Want to innovate, actually innovate.

**❌ Identical cards in card grid**
Asymmetric, varying-size cards — some with images, some text-only, some spanning columns — that's what real designers produce.

## Content Standards

### 1. Don't add filler content

Every element must earn its place. Empty space is a design problem — solve with **composition** (contrast, rhythm, whitespace), **not** by filling with content.

**Identifying filler**:
- Remove this content — does design get worse? If "no," remove it.
- What real problem does this element solve? If "making page feel less empty," delete it.
- Is there real data supporting this stat/quote/feature? If not, don't make it up.

"One thousand no's for every yes."

### 2. Ask before adding material

Think adding another section / page / block would improve things? Ask user first — don't add unilaterally.

Reasons:
- User knows their audience better than you
- Adding content has cost; user may not want it
- Unilateral additions violate the "junior designer reporting to senior" relationship

### 3. Create a system up front

After exploring design context, **verbally state system you intend to use** and let user confirm:

```markdown
My design system:
- Color: #1A1A1A primary + #F0EEE6 background + #D97757 accent (from your brand)
- Typography: Instrument Serif for display + Geist Sans for body
- Rhythm: section titles use full-bleed colored background + white text; regular sections use white background
- Imagery: hero uses full-bleed photo; feature sections use placeholder until you provide real images
- Maximum 2 background colors to avoid visual noise

Confirm this direction and I'll begin.
```

Begin only after user confirms. Prevents "finishing half the work only to discover direction was wrong."

## Scale Specs

### Slides (1920×1080)

- Body text minimum **24px**, ideal 28-36px
- Headings 60-120px
- Section title 80-160px
- Hero headline 180-240px
- Never use text smaller than 24px on slides

### Print Documents

- Body text minimum **10pt** (≈13.3px), ideal 11-12pt
- Headings 18-36pt
- Captions 8-9pt

### Web and Mobile

- Body text minimum **14px** (16px for elderly-friendly)
- Mobile body text **16px** (prevents iOS auto-zoom)
- Hit targets minimum **44×44px**
- Line height 1.5-1.7 (Chinese text 1.7-1.8)

### Contrast

- Body text vs. background **at least 4.5:1** (WCAG AA)
- Large text vs. background **at least 3:1**
- Check with Chrome DevTools accessibility panel

## CSS Power Moves

**Advanced CSS features** — use boldly:

### Typography

```css
/* Makes heading line breaks more natural — avoids lone word on last line */
h1, h2, h3 { text-wrap: balance; }

/* Body text wrapping — avoids widows and orphans */
p { text-wrap: pretty; }

/* CJK typography: punctuation compression, line-start/end control */
p { 
  text-spacing-trim: space-all;
  hanging-punctuation: first;
}
```

### Layout

```css
/* CSS Grid + named areas = outstanding readability */
.layout {
  display: grid;
  grid-template-areas:
    "header header"
    "sidebar main"
    "footer footer";
  grid-template-columns: 240px 1fr;
  grid-template-rows: auto 1fr auto;
}

/* Subgrid for aligning card content */
.card { display: grid; grid-template-rows: subgrid; }
```

### Visual Effects

```css
/* Styled scrollbar */
* { scrollbar-width: thin; scrollbar-color: #666 transparent; }

/* Glassmorphism (use with restraint) */
.glass {
  backdrop-filter: blur(20px) saturate(150%);
  background: color-mix(in oklch, white 70%, transparent);
}

/* View Transitions API for smooth page transitions */
@view-transition { navigation: auto; }
```

### Interaction

```css
/* :has() selector makes conditional styling easy */
.card:has(img) { padding-top: 0; } /* cards with images have no top padding */

/* Container queries make components truly responsive */
@container (min-width: 500px) { ... }

/* New color-mix function */
.button:hover {
  background: color-mix(in oklch, var(--primary) 85%, black);
}
```

## Decision Quick Reference: When You're Unsure

- Want to add gradient? → Probably don't
- Want to add emoji? → Don't
- Want rounded corners + border-left accent? → Don't — find another approach
- Want SVG hero illustration? → Don't — use placeholder
- Want decorative quote? → Ask if user has real quote first
- Want row of icon features? → Ask if icons needed — probably aren't
- Using Inter? → Switch to something more distinctive
- Using purple gradient? → Switch to brand-grounded color choice

**Feeling "adding this would look better" — usually a sign of AI slop**. Start simplest; add only when user asks.
