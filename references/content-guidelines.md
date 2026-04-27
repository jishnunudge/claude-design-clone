# Content Guidelines: Anti-AI Slop, Content Standards, Scale Specs

The traps most easily fallen into in AI design. This is a "what not to do" checklist — more important than "what to do" — because AI slop is the default. If you don't actively avoid it, it will happen.

## AI Slop Complete Blacklist

### Visual Traps

**❌ Aggressive gradient backgrounds**
- Purple → pink → blue full-screen gradients (the signature look of AI-generated web pages)
- Rainbow gradients in any direction
- Mesh gradients filling the background
- ✅ If you want to use gradients: subtle, monochromatic, used intentionally as accents (e.g., button hover)

**❌ Rounded cards + left border accent color**
```css
/* This is the signature of an AI-flavored card */
.card {
  border-radius: 12px;
  border-left: 4px solid #3b82f6;
  padding: 16px;
}
```
This card style is rampant in AI-generated dashboards. Want to create emphasis? Use more design-considered approaches: background color contrast, font weight/size contrast, plain dividers, or simply skip the card containers altogether.

**❌ Emoji decoration**
Unless the brand itself uses emoji (like Notion, Slack), do not put emoji in the UI. **Especially avoid**:
- 🚀 ⚡️ ✨ 🎯 💡 before headings
- ✅ in feature lists
- → inside CTA buttons (standalone arrows are OK; emoji arrows are not)

No icons? Use a real icon library (Lucide/Heroicons/Phosphor) or use a placeholder.

**❌ SVG imagery**
Do not attempt to use SVG to draw: people, scenes, devices, objects, or abstract art. AI-drawn SVG imagery is immediately recognizable as AI — it looks naive and cheap. **A gray rectangle with a text label "Illustration placeholder 1200×800" is 100× better than a clumsy SVG hero illustration**.

The only acceptable SVG uses:
- Real icons (16×16 to 32×32 level)
- Geometric shapes as decorative elements
- Data viz charts

**❌ Excessive iconography**
Not every heading / feature / section needs an icon. Overusing icons makes the interface look like a toy. Less is more.

**❌ "Data slop"**
Made-up stats used as decoration:
- "10,000+ happy customers" (you don't even know if that's true)
- "99.9% uptime" (don't write it without real data)
- Decorative "metric cards" composed of icons + numbers + phrases
- Fake data dressed up in mock tables

If there's no real data, leave a placeholder or ask the user.

**❌ "Quote slop"**
Made-up user testimonials or celebrity quotes used to decorate the page. Leave a placeholder and ask the user for real quotes.

### Typography Traps

**❌ Avoid these overused fonts**:
- Inter (the default for AI-generated web pages)
- Roboto
- Arial / Helvetica
- Pure system font stacks
- Fraunces (AI discovered this and overused it)
- Space Grotesk (currently AI's favorite)

**✅ Use distinctive display + body pairings**. Inspiration directions:
- Serif display + sans-serif body (editorial feel)
- Mono display + sans body (technical feel)
- Heavy display + light body (contrast)
- Variable font for weight animation in the hero

Font resources:
- Underused gems on Google Fonts (Instrument Serif, Cormorant, Bricolage Grotesque, JetBrains Mono)
- Open-source font sites (sibling fonts to Fraunces, Adobe Fonts)
- Do not invent font names out of thin air

### Color Traps

**❌ Inventing colors from scratch**
Do not design an entirely unfamiliar color set from zero. It usually turns out inharmonious.

**✅ Strategy**:
1. Has brand colors → use them; fill missing color tokens using oklch interpolation
2. No brand colors but has a reference → sample colors from the reference product screenshot
3. Starting completely from zero → pick a known color system (Radix Colors / Tailwind default palette / Anthropic brand) rather than dialing your own

**oklch color definition** is the most modern approach:
```css
:root {
  --primary: oklch(0.65 0.18 25);      /* warm terracotta */
  --primary-light: oklch(0.85 0.08 25); /* lighter in the same hue */
  --primary-dark: oklch(0.45 0.20 25);  /* darker in the same hue */
}
```
oklch ensures hue doesn't drift when adjusting lightness — more reliable than hsl.

**❌ Casually adding an inverted dark mode**
Dark mode is not a simple color inversion. A good dark mode requires re-tuning saturation, contrast, and accent colors. If you're not willing to do that, don't add a dark mode.

### Layout Traps

**❌ Bento grid overuse**
Every AI-generated landing page wants a bento layout. Unless your information structure genuinely suits bento, use a different layout.

**❌ Big hero + 3-column features + testimonials + CTA**
This landing page template has been done to death. If you want to innovate, actually innovate.

**❌ Every card in a card grid looks identical**
Asymmetric, varying-size cards — some with images, some text-only, some spanning columns — that's what real designers produce.

## Content Standards

### 1. Don't add filler content

Every element must earn its place. Empty space is a design problem — solve it with **composition** (contrast, rhythm, whitespace), **not** by filling it with content.

**Questions to identify filler**:
- If you remove this piece of content, does the design get worse? If the answer is "no," remove it.
- What real problem does this element solve? If the answer is "making the page feel less empty," delete it.
- Is there real data supporting this stat/quote/feature? If not, don't make it up.

"One thousand no's for every yes."

### 2. Ask before adding material

You think adding another section / page / block would make it better? Ask the user first — don't add it unilaterally.

Reasons:
- The user knows their audience better than you do
- Adding content has a cost; the user may not want it
- Adding content unilaterally violates the "junior designer reporting to a senior" relationship

### 3. Create a system up front

After exploring the design context, **verbally state the system you intend to use** and let the user confirm:

```markdown
My design system:
- Color: #1A1A1A primary + #F0EEE6 background + #D97757 accent (from your brand)
- Typography: Instrument Serif for display + Geist Sans for body
- Rhythm: section titles use full-bleed colored background + white text; regular sections use white background
- Imagery: hero uses full-bleed photo; feature sections use placeholder until you provide real images
- Maximum 2 background colors to avoid visual noise

Confirm this direction and I'll begin.
```

Begin only after the user confirms. This check-in prevents "finishing half the work only to discover the direction was wrong."

## Scale Specs

### Slides (1920×1080)

- Body text minimum **24px**, ideal 28-36px
- Headings 60-120px
- Section title 80-160px
- Hero headline can go 180-240px
- Never use text smaller than 24px on slides

### Print Documents

- Body text minimum **10pt** (≈13.3px), ideal 11-12pt
- Headings 18-36pt
- Captions 8-9pt

### Web and Mobile

- Body text minimum **14px** (16px for elderly-friendly)
- Mobile body text **16px** (prevents iOS auto-zoom)
- Hit targets (tappable elements) minimum **44×44px**
- Line height 1.5-1.7 (Chinese text 1.7-1.8)

### Contrast

- Body text vs. background **at least 4.5:1** (WCAG AA)
- Large text vs. background **at least 3:1**
- Check with Chrome DevTools accessibility panel

## CSS Power Moves

**Advanced CSS features** are a designer's best friend — use them boldly:

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
- Want to add rounded corners + border-left accent to a card? → Don't — find another approach
- Want to draw a hero illustration in SVG? → Don't — use a placeholder
- Want to add a decorative quote? → Ask the user if they have a real quote first
- Want to add a row of icon features? → Ask if icons are needed — they probably aren't
- Using Inter? → Switch to something more distinctive
- Using a purple gradient? → Switch to a color choice grounded in the brand

**When you feel "adding this would look better" — that is usually a sign of AI slop**. Start with the simplest version and only add when the user asks.
