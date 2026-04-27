# Design Context: Starting from Existing Context

**This is the single most important thing about this skill.**

Good hi-fi design grows from existing design context. **Building hi-fi from scratch is a last resort and produces generic work.** At the start of every task, ask: is there anything to reference?

## What Is Design Context

In order of priority:

### 1. The User's Design System / UI Kit
The product's existing component library, color tokens, typography rules, icon system. **Ideal situation.**

### 2. The User's Codebase
If provided, read these files:
- `theme.ts` / `colors.ts` / `tokens.css` / `_variables.scss`
- Specific components (Button.tsx, Card.tsx)
- Layout scaffold (App.tsx, MainLayout.tsx)
- Global stylesheets

**Copy exact values**: hex codes, spacing scale, font stack, border radius. Don't work from memory.

### 3. The User's Published Product
If no code available, use Playwright or ask for screenshots:

```bash
# Screenshot a public URL with Playwright
npx playwright screenshot https://example.com screenshot.png --viewport-size=1920,1080
```

### 4. Brand Guidelines / Logo / Existing Assets
Logo files, color specs, marketing materials, slide templates — all are context.

### 5. Competitor References
User says "something like XX website" — ask for URL or screenshots. **Do not** work from a vague training-data impression.

### 6. Known Design Systems (Fallback)
If none of the above are available:
- Apple HIG
- Material Design 3
- Radix Colors (color palettes)
- shadcn/ui (components)
- Tailwind default palette

Tell the user what you're using so they know it's a starting point, not a final draft.

## Process for Obtaining Context

### Step 1: Ask the User

```markdown
1. Do you have an existing design system / UI kit / component library? Where?
2. Do you have brand guidelines, color / typography specs?
3. Can you give me screenshots or a URL of your existing product?
4. Is there a codebase I can read?
```

### Step 2: When User Says "No," Help Them Find Something

```markdown
Let me see if there are any clues:
- Do any previous projects have related design work?
- What colors / fonts does the company's marketing website use?
- What style is your product's logo? Can you share one?
- Are there any products you admire as a reference?
```

### Step 3: Read All Available Context

If a codebase path is provided:
1. **List the file structure first**: find style / theme / component-related files
2. **Read theme / token files**: extract specific hex / px values
3. **Read 2–3 representative components**: note visual vocabulary (hover state, shadow, border, padding)
4. **Read the global stylesheet**: base resets, font loading
5. **If there's a Figma link / screenshot**: look at it, but **trust the code more**

**Do not** glance and work from memory. You need 30+ specific values to have truly lifted the context.

### Step 4: Vocalize the System You'll Use

```markdown
Based on your codebase and product screenshots, here's the design system I've distilled:

**Color**
- Primary: #C27558 (from tokens.css)
- Background: #FDF9F0
- Text: #1A1A1A
- Muted: #6B6B6B

**Typography**
- Display: Instrument Serif (from @font-face in global.css)
- Body: Geist Sans
- Mono: JetBrains Mono

**Spacing** (from your scale system)
- 4, 8, 12, 16, 24, 32, 48, 64

**Shadow pattern**
- `0 1px 2px rgba(0,0,0,0.04)` (subtle card)
- `0 10px 40px rgba(0,0,0,0.1)` (elevated modal)

**Border-radius**
- Small components 4px, cards 12px, buttons 8px

**Component vocabulary**
- Button: filled primary, outlined secondary, ghost tertiary, all with 8px border-radius
- Card: white background, subtle shadow, no border

I'll start building with this system. Does that look right?
```

Confirm with user before starting.

## Designing Without Context (Fallback)

**Output quality drops significantly.** Tell the user clearly:

```markdown
You have no design context, so I can only work from general intuition.
The result will be "looks OK but lacks distinctiveness."
Do you want to continue, or provide references first?
```

If user insists, make decisions in this order:

### 1. Choose an Aesthetic Direction
Pick a clear direction — don't deliver generic:
- brutally minimal / editorial / brutalist / organic / luxury / playful / retro-futuristic / soft pastel

Tell the user which one you chose.

### 2. Choose a Known Design System as Skeleton
- Radix Colors for color (https://www.radix-ui.com/colors)
- shadcn/ui for component vocabulary (https://ui.shadcn.com)
- Tailwind spacing scale (multiples of 4)

### 3. Choose a Distinctive Font Pairing

Don't use Inter/Roboto. Free from Google Fonts:
- Instrument Serif + Geist Sans
- Cormorant Garamond + Inter Tight
- Bricolage Grotesque + Söhne (paid)
- Fraunces + Work Sans (Fraunces is overused by AI)
- JetBrains Mono + Geist Sans (technical feel)

### 4. Every Key Decision Has Reasoning

```html
<!--
Design decisions:
- Primary color: warm terracotta (oklch 0.65 0.18 25) — fits the "editorial" direction  
- Display: Instrument Serif for humanist, literary feel
- Body: Geist Sans for cleanness contrast
- No gradients — committed to minimal, no AI slop
- Spacing: 8px base, golden ratio friendly (8/13/21/34)
-->
```

## Import Strategy (When User Provides a Codebase)

### Small (<50 files)
Read everything.

### Medium (50–500 files)
Focus on:
- `src/components/` or `components/`
- All styles / tokens / theme-related files
- 2–3 representative full-page components (Home.tsx, Dashboard.tsx)

### Large (>500 files)
Ask user to specify focus:
- "Building the settings page" → read existing settings-related files
- "Building a new feature" → read the overall shell + closest reference
- Don't try to read everything

## Working with Figma / Design Files

If user provides a Figma link:
- **Do not** expect to directly "convert Figma to HTML" — requires additional tools
- Figma links are usually not publicly accessible
- Ask user to: export as **screenshots** + tell you specific color/spacing values

If given only a Figma screenshot: I can see the visual but can't extract precise values — for key numbers (hex, px), provide them or export as code.

## One Final Reminder

**The quality ceiling of a project's design is set by the quality of context you receive.**

10 minutes collecting context beats 1 hour building hi-fi from scratch.

**When you have no context, ask for it rather than pushing ahead.**
