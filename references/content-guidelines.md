# Content Guidelines: Anti-AI Slop, Content Standards, Scale Specs

Traps most easily fallen into in AI design. This is a "what not to do" checklist — more important than "what to do" — because AI slop is the default. Don't actively avoid it and it will happen.

## AI Slop Complete Blacklist

### Visual Traps

**❌ Aggressive gradient backgrounds**
- Purple → pink → blue full-screen gradients (signature of AI-generated web pages)
- Rainbow gradients in any direction
- Mesh gradients filling the background
- ✅ Want gradients: subtle, monochromatic, intentional accent use (e.g., button hover)

**❌ Rounded cards + left border accent color**
```css
/* Signature of an AI-flavored card */
.card {
  border-radius: 12px;
  border-left: 4px solid #3b82f6;
  padding: 16px;
}
```
Rampant in AI-generated dashboards. Want emphasis? Use design-considered approaches: background color contrast, font weight/size contrast, plain dividers, or skip card containers altogether.

**❌ Emoji decoration**
Unless the brand uses emoji (like Notion, Slack), don't put emoji in UI. **Especially avoid**:
- 🚀 ⚡️ ✨ 🎯 💡 before headings
- ✅ in feature lists
- → inside CTA buttons (standalone arrows OK; emoji arrows not)

No icons? Use a real icon library (Lucide/Heroicons/Phosphor) or a placeholder.

**❌ SVG imagery**
Don't use SVG to draw people, scenes, devices, objects, or abstract art. AI-drawn SVG imagery is immediately recognizable — naive and cheap. **A gray rectangle labeled "Illustration placeholder 1200×800" is 100× better than a clumsy SVG hero illustration**.

Acceptable SVG uses only:
- Real icons (16×16 to 32×32)
- Geometric shapes as decorative elements
- Data viz charts

**❌ Excessive iconography**
Not every heading / feature / section needs an icon. Overusing icons makes interfaces look like toys.

**❌ "Data slop"**
Made-up stats as decoration:
- "10,000+ happy customers" (unverified)
- "99.9% uptime" (don't write without real data)
- Decorative "metric cards" of icons + numbers + phrases
- Fake data in mock tables

No real data? Leave a placeholder or ask the user.

**❌ "Quote slop"**
Made-up testimonials or celebrity quotes. Leave a placeholder and ask for real quotes.

### Typography Traps

**❌ Avoid these overused fonts**:
- Inter (default for AI-generated web pages)
- Roboto
- Arial / Helvetica
- Pure system font stacks
- Fraunces (AI discovered and overused it)
- Space Grotesk (currently AI's favorite)

**✅ Use distinctive display + body pairings**:
- Serif display + sans-serif body (editorial)
- Mono display + sans body (technical)
- Heavy display + light body (contrast)
- Variable font for weight animation in hero

Font resources:
- Underused Google Fonts gems: Instrument Serif, Cormorant, Bricolage Grotesque, JetBrains Mono
- Open-source font sites (siblings to Fraunces, Adobe Fonts)
- Don't invent font names

### Color Traps

**❌ Inventing colors from scratch**
Usually turns out inharmonious.

**✅ Strategy**:
1. Has brand colors → use them; fill missing tokens with oklch interpolation
2. Has reference → sample colors from reference product screenshot
3. Zero context → pick known color system (Radix Colors / Tailwind default / Anthropic brand)

**oklch color definition** — most modern approach:
```css
:root {
  --primary: oklch(0.65 0.18 25);      /* warm terracotta */
  --primary-light: oklch(0.85 0.08 25); /* lighter in the same hue */
  --primary-dark: oklch(0.45 0.20 25);  /* darker in the same hue */
}
```
oklch ensures hue doesn't drift when adjusting lightness — more reliable than hsl.

**❌ Casually adding an inverted dark mode**
Dark mode isn't simple color inversion. Good dark mode requires re-tuning saturation, contrast, and accent colors. If you're not doing that, don't add it.

### Layout Traps

**❌ Bento grid overuse**
Every AI-generated landing page wants bento. Unless your information structure genuinely suits it, use a different layout.

**❌ Big hero + 3-column features + testimonials + CTA**
This template is done to death.

**❌ Every card in a grid looks identical**
Real designers produce asymmetric, varying-size cards — some with images, some text-only, some spanning columns.

## Content Standards

### 1. Don't add filler content

Every element must earn its place. Empty space is a design problem — solve with **composition** (contrast, rhythm, whitespace), not by filling with content.

**Questions to identify filler**:
- Remove this content — does design get worse? If no, remove it.
- What real problem does this element solve? If "making the page feel less empty," delete it.
- Is there real data supporting this stat/quote/feature? If not, don't make it up.

"One thousand no's for every yes."

### 2. Ask before adding material

Think another section / page / block would help? Ask the user first — don't add unilaterally.

Reasons:
- User knows their audience better
- Adding content has cost; user may not want it
- Unilateral additions violate the "junior designer reporting to senior" relationship

### 3. Create a system up front

After exploring design context, **verbally state the system** and let user confirm:

```markdown
My design system:
- Color: #1A1A1A primary + #F0EEE6 background + #D97757 accent (from your brand)
- Typography: Instrument Serif for display + Geist Sans for body
- Rhythm: section titles use full-bleed colored background + white text; regular sections use white background
- Imagery: hero uses full-bleed photo; feature sections use placeholder until you provide real images
- Maximum 2 background colors to avoid visual noise

Confirm this direction and I'll begin.
```

Begin only after confirmation. Prevents "finishing half the work only to find the direction was wrong."

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

**Advanced CSS features** — use them boldly:

### Typography

```css
/* Makes heading line breaks more natural — avoids a lone word on the last line */
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

- Want to add a gradient? → Probably don't
- Want to add an emoji? → Don't
- Want rounded corners + border-left accent on a card? → Don't — find another approach
- Want to draw a hero illustration in SVG? → Don't — use a placeholder
- Want a decorative quote? → Ask if user has a real quote first
- Want a row of icon features? → Ask if icons are needed — they probably aren't
- Using Inter? → Switch to something more distinctive
- Using a purple gradient? → Switch to a brand-grounded color choice

**When you feel "adding this would look better" — that's usually AI slop**. Start with the simplest version and only add when the user asks.
