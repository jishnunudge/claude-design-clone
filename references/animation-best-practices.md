# Animation Best Practices · Positive Animation Design Grammar

> Derived from deep analysis of three official Anthropic product animations (Claude Design / Claude Code Desktop / Claude for Word),
> distilling "Anthropic-grade" animation design rules.
>
> Use alongside `animation-pitfalls.md` (anti-patterns checklist) — this file is "**what you should do**",
> pitfalls is "**what not to do**". They are orthogonal — read both.
>
> **Scope**: Covers only **motion logic and expressive style** — no specific brand color values.
> Color decisions go through §1.a core asset protocol (extracted from brand spec) or the "design direction advisor"
> (color schemes for each of the 20 philosophies). This reference covers "**how to move**", not "**what color**".

---

## §0 · Who You Are · Identity and Taste

> Read this before any technical rules. Rules **emerge from identity** — not the other way around.

### §0.1 Identity Anchor

**You are a motion designer who has studied the motion archives of Anthropic / Apple / Pentagram / Field.io.**

When creating animations, you are not tweaking CSS transitions — you are using digital elements to **simulate a physical world**,
convincing the audience's subconscious that "this has weight, inertia, and overflow."

You don't make PowerPoint-style animations. You don't make "fade in fade out" animations. You make animations **that make people believe the screen
is a space they could reach into**.

### §0.2 Core Beliefs (3)

1. **Animation is physics, not animation curves**
   `linear` is a number, `expoOut` is an object. Every pixel deserves to be treated as a "thing."
   Every easing choice answers a physical question: "How heavy is this element? What is its coefficient of friction?"

2. **Time distribution matters more than curve shape**
   Slow-Fast-Boom-Stop is your breathing rhythm. **Evenly-paced animation is a technical demo; rhythmic animation is storytelling.**
   Slowing down at the right moment is more important than using the correct easing at the wrong moment.

3. **Respecting the audience is harder than showing off**
   Pausing 0.5 seconds before a key result is **craft**, not compromise. **Giving the brain reaction time is the highest discipline of an animator.**
   AI by default produces animation with no pauses and maximum information density — that's beginner work. What you do is restraint.

### §0.3 Taste Standards · What Is Beautiful

Each dimension has an **identification method** — use these questions to assess a candidate animation rather than mechanically checking rules.

| Dimension | Identification method (audience reaction) |
|---|---|
| **Physical weight** | When animation ends, elements "**land**" solidly — they don't just "**stop**". Audience subconsciously feels "this has weight" |
| **Respect for the audience** | Before key information appears there is a perceptible pause (≥300ms) — audience has time to "**see**" it before moving on |
| **Negative space** | Ending is hard stop + hold, not fade to black. Final frame is clear, decisive, definitive |
| **Restraint** | Only one "120% refined" moment in the entire piece; the other 80% is just right — **showing off everywhere signals cheapness** |
| **Tactility** | Arcs (not straight lines), irregularity (not setInterval mechanical rhythm), a sense of breathing |
| **Respect** | Showing the tweak process, showing bug fixes — **not hiding the work, not performing "magic"**. AI is a collaborator, not a magician |

### §0.4 Self-check · First-Reaction Method

After completing an animation, **what is the audience's first reaction?** — that is the only metric to optimize.

| Audience reaction | Rating | Diagnosis |
|---|---|---|
| "Looks pretty smooth" | good | Competent but undistinguished; making PowerPoint |
| "That animation flows really well" | good+ | Technically right, but not stunning |
| "This thing actually looks like it's **floating up off a desk**" | great | Touched physical weight |
| "This doesn't look like it was made by AI" | great+ | Reached the Anthropic threshold |
| "I want to **screenshot this** and share it" | great++ | Made the audience want to spread it |

**The difference between great and good is not technical correctness — it's taste judgment**. Technical correctness + right taste = great.
Technical correctness + empty taste = good. Technical errors = didn't make the cut.

### §0.5 The Relationship Between Identity and Rules

The technical rules in §1-§8 are **execution means** of this identity in specific scenarios — not an independent rule checklist.

- When a rule doesn't cover a scenario → return to §0, judge with **identity**, don't guess
- When rules conflict → return to §0, use **taste standards** to judge which matters more
- Want to break a rule → first answer: "Which dimension of beauty in §0.3 does this serve?" If you can answer it, break it; if not, don't

---

## Overview · Animation as Physics — Three-Level Expansion

Root cause of cheap-feeling AI-generated animations: **they behave like "numbers" not "objects"**.
Real objects have mass, inertia, elasticity, and overflow. Anthropic's three films achieve "premium feel" by giving digital elements
**physical world motion rules**.

3 levels:

1. **Narrative rhythm layer**: Slow-Fast-Boom-Stop time distribution
2. **Motion curve layer**: Expo Out / Overshoot / Spring, rejecting linear
3. **Expressive language layer**: Showing process, mouse arcs, Logo morph convergence

---

## 1. Narrative Rhythm · Slow-Fast-Boom-Stop 5-Part Structure

All three Anthropic films follow this structure without exception:

| Segment | Share | Rhythm | Purpose |
|---|---|---|---|
| **S1 Trigger** | ~15% | Slow | Give humans reaction time, establish authenticity |
| **S2 Generate** | ~15% | Medium | Visual wow moment appears |
| **S3 Process** | ~40% | Fast | Show controllability / density / detail |
| **S4 Burst** | ~20% | Boom | Camera pulls back / 3D pop-out / multi-panel surge |
| **S5 Land** | ~10% | Still | Brand Logo + hard stop |

**Duration mapping** (15-second animation):
S1 Trigger 2s · S2 Generate 2s · S3 Process 6s · S4 Burst 3s · S5 Land 2s

**Prohibited**:
- ❌ Uniform rhythm (same information density every second) — audience fatigue
- ❌ Sustained high density — no peak, no memorable moment
- ❌ Fading out at the end — should be a **hard stop**

**Self-check**: Sketch 5 thumbnails on paper, each representing the climactic frame of one segment. If the 5 images don't differ much, rhythm hasn't been achieved.

---

## 2. Easing Philosophy · Reject Linear, Embrace Physics

All motion effects in Anthropic's three films use Bézier curves with a "damped" feel. Default cubic easeOut
(`1-(1-t)³`) is **not sharp enough** — doesn't start fast enough, doesn't stop steadily enough.

### Three Core Easings (built into animations.jsx)

```js
// 1. Expo Out · Fast start, slow brake (most common, default main easing)
// CSS equivalent: cubic-bezier(0.16, 1, 0.3, 1)
Easing.expoOut(t) // = t === 1 ? 1 : 1 - Math.pow(2, -10 * t)

// 2. Overshoot · Elastic toggle/button pop
// CSS equivalent: cubic-bezier(0.34, 1.56, 0.64, 1)
Easing.overshoot(t)

// 3. Spring physics · Geometry homing, natural settling
Easing.spring(t)
```

### Usage Mapping

| Scenario | Easing |
|---|---|
| Card rise-in / panel entrance / Terminal fade / focus overlay | **`expoOut`** (main easing, most used) |
| Toggle switch / button pop / emphasis interaction | `overshoot` |
| Preview geometry homing / physical landing / UI element bounce | `spring` |
| Continuous motion (e.g. mouse trajectory interpolation) | `easeInOut` (preserves symmetry) |

### Counter-intuitive Insight

Most product promo animations are **too fast and too hard**. `linear` makes digital elements feel like machines; `easeOut` is baseline,
but `expoOut` is the technical root of "premium feel" — it gives digital elements a **physical-world sense of weight**.

---

## 3. Motion Language · 8 Common Principles

### 3.1 Background color is never pure black or pure white

None of Anthropic's three films use `#FFFFFF` or `#000000` as primary background. **Neutral colors with color temperature**
(warm or cool) have the material feel of "paper / canvas / desk", reducing mechanical feel.

Specific color decisions go through §1.a core asset protocol (extracted from brand spec) or the "design direction advisor". This reference does not prescribe specific color values — that is **brand decision**, not motion rules.

### 3.2 Easing is never linear

See §2.

### 3.3 Slow-Fast-Boom-Stop Narrative

See §1.

### 3.4 Show "Process" Not "Magic Results"

- Claude Design shows tweaking parameters, dragging sliders (not one-click perfect results)
- Claude Code shows code errors + AI fixes (not first-try success)
- Claude for Word shows Redline red-deletion/green-addition revision process (not delivering a final draft directly)

**Shared subtext**: The product is a **collaborator, pair engineer, senior editor** — not a one-click magician.
Targets professional users' pain points around "controllability" and "authenticity."

**Anti-AI slop**: AI by default makes "magic one-click success" animations. **Do the opposite** — show the process, show tweaks, show bugs and fixes —
that is the source of brand recognition.

### 3.5 Mouse Trajectory Hand-drawn (Arcs + Perlin Noise)

Real human mouse movement is "accelerate → arc → decelerate and correct → click."
AI's direct linear-interpolated mouse trajectory has a **subconscious repulsion effect**.

```js
// Quadratic Bézier curve interpolation (start → control point → end)
function bezierQuadratic(p0, p1, p2, t) {
  const x = (1-t)*(1-t)*p0[0] + 2*(1-t)*t*p1[0] + t*t*p2[0];
  const y = (1-t)*(1-t)*p0[1] + 2*(1-t)*t*p1[1] + t*t*p2[1];
  return [x, y];
}

// Path: start → offset midpoint → end (creates arc)
const path = [[100, 100], [targetX - 200, targetY + 80], [targetX, targetY]];

// Layer very small Perlin Noise (±2px) to create "hand tremor"
const jitterX = (simpleNoise(t * 10) - 0.5) * 4;
const jitterY = (simpleNoise(t * 10 + 100) - 0.5) * 4;
```

### 3.6 Logo "Morph Convergence"

In all three Anthropic films, Logo entrance is **never a simple fade-in** — it **morphs from the preceding visual element**.

**Common pattern**: In the last 1-2 seconds, Morph / Rotate / Converge, letting the entire narrative "collapse" onto the brand mark.

**Low-cost implementation** (without true morph):
Let the preceding element "collapse" into a color block (scale → 0.1, translate toward center),
then the color block "expands" to become the wordmark. Use a 150ms quick cut + motion blur
(`filter: blur(6px)` → `0`) for the transition.

```js
<Sprite start={13} end={14}>
  {/* Collapse: previous element scale 0.1, opacity maintained, filter blur increases */}
  const scale = interpolate(t, [0, 0.5], [1, 0.1], Easing.expoOut);
  const blur = interpolate(t, [0, 0.5], [0, 6]);
</Sprite>
<Sprite start={13.5} end={15}>
  {/* Expand: Logo from color block center scale 0.1 → 1, blur 6 → 0 */}
  const scale = interpolate(t, [0, 0.6], [0.1, 1], Easing.overshoot);
  const blur = interpolate(t, [0, 0.6], [6, 0]);
</Sprite>
```

### 3.7 Serif + Sans-serif Dual Typography

- **Brand / narration**: Serif (conveys "academic / publication / taste")
- **UI / code / data**: Sans-serif + monospace

**A single typeface is always wrong**. Serif gives "taste", sans-serif gives "function."

Specific typeface choices go through brand spec (Display / Body / Mono three-stack in brand-spec.md) or the design direction
advisor's 20 philosophies. This reference does not prescribe specific fonts — that is **brand decision**.

### 3.8 Focus Switch = Background Dim + Foreground Sharpen + Flash Guide

Focus switching is **not just** reducing opacity. Complete recipe:

```js
// Filter combination for non-focused elements
tile.style.filter = `
  brightness(${1 - 0.5 * focusIntensity})
  saturate(${1 - 0.3 * focusIntensity})
  blur(${focusIntensity * 4}px)        // ← Key: blur is what actually makes it "recede"
`;
tile.style.opacity = 0.4 + 0.6 * (1 - focusIntensity);

// After focus completes, do a 150ms Flash highlight at focus position to guide eyes back
focusOverlay.animate([
  { background: 'rgba(255,255,255,0.3)' },
  { background: 'rgba(255,255,255,0)' }
], { duration: 150, easing: 'ease-out' });
```

**Why blur is essential**: With only opacity + brightness, out-of-focus elements remain "sharp" visually,
without a true "receding to background" effect. blur(4-8px) genuinely pushes non-focused elements one depth plane back.

---

## 4. Specific Motion Techniques (Copy-ready Code Snippets)

### 4.1 FLIP / Shared Element Transition

A button "expands" into an input field — **not** the button disappearing and a new panel appearing. Key: **the same DOM element**
transitioning between two states, not two elements cross-fading.

```jsx
// Using Framer Motion layoutId
<motion.div layoutId="design-button">Design</motion.div>
// ↓ After click, same layoutId
<motion.div layoutId="design-button">
  <input placeholder="Describe your design..." />
</motion.div>
```

Native implementation reference: https://aerotwist.com/blog/flip-your-animations/

### 4.2 "Breathing" Expansion (width→height)

Panel expansion is **not** pulling width and height simultaneously — instead:
- First 40% of time: only expand width (keep height small)
- Last 60% of time: width stays, expand height

Simulates the physical feel of "unfold first, then fill with liquid."

```js
const widthT = interpolate(t, [0, 0.4], [0, 1], Easing.expoOut);
const heightT = interpolate(t, [0.3, 1], [0, 1], Easing.expoOut);
style.width = `${widthT * targetW}px`;
style.height = `${heightT * targetH}px`;
```

### 4.3 Staggered Fade-up (30ms stagger)

When table rows, card columns, or list items enter, **each element delays 30ms**, `translateY` returning from 10px to 0.

```js
rows.forEach((row, i) => {
  const localT = Math.max(0, t - i * 0.03);  // 30ms stagger
  row.style.opacity = interpolate(localT, [0, 0.3], [0, 1], Easing.expoOut);
  row.style.transform = `translateY(${
    interpolate(localT, [0, 0.3], [10, 0], Easing.expoOut)
  }px)`;
});
```

### 4.4 Non-linear Breathing · 0.5s Hold Before Key Results

**Hold 0.5 seconds before a key result appears**, giving the audience's brain reaction time.

```jsx
// Typical scenario: AI generation complete → hold 0.5s → result appears
<Sprite start={8} end={8.5}>
  {/* 0.5s pause — nothing moves, let audience stare at loading state */}
  <LoadingState />
</Sprite>
<Sprite start={8.5} end={10}>
  <ResultAppear />
</Sprite>
```

**Counter-example**: AI generation complete immediately cuts to result — audience has no reaction time, information is lost.

### 4.5 Chunk Reveal · Simulate Token Streaming

AI-generated text **should not use `setInterval` single-character popping** — use **chunk reveal**
— appear 2-5 characters at a time, with irregular intervals, simulating real token streaming output.

```js
// Split into chunks, not characters
const chunks = text.split(/(\s+|,\s*|\.\s*|;\s*)/);  // split by word + punctuation
let i = 0;
function reveal() {
  if (i >= chunks.length) return;
  element.textContent += chunks[i++];
  const delay = 40 + Math.random() * 80;  // irregular 40-120ms
  setTimeout(reveal, delay);
}
reveal();
```

### 4.6 Anticipation → Action → Follow-through

3 of Disney's 12 principles. Anthropic uses them explicitly:

- **Anticipation**: Small reverse movement before action (button slightly shrinks then pops out)
- **Action**: The main action itself
- **Follow-through**: Residual motion after action ends (card lightly bounces after landing)

```js
// Complete three-phase card entrance
const anticip = interpolate(t, [0, 0.2], [1, 0.95], Easing.easeIn);     // anticipation
const action  = interpolate(t, [0.2, 0.7], [0.95, 1.05], Easing.expoOut); // main action
const settle  = interpolate(t, [0.7, 1], [1.05, 1], Easing.spring);       // settle
// Final scale = product or sequential application of three phases
```

**Counter-example**: Animation with only Action and no Anticipation + Follow-through feels like "PowerPoint animation."

### 4.7 3D Perspective + translateZ Layering

For the "tilted 3D + floating cards" aesthetic, add perspective to the container and different translateZ to individual elements:

```css
.stage-wrap {
  perspective: 2400px;
  perspective-origin: 50% 30%;  /* viewpoint slightly overhead */
}
.card-grid {
  transform-style: preserve-3d;
  transform: rotateX(8deg) rotateY(-4deg);  /* golden ratio */
}
.card:nth-child(3n) { transform: translateZ(30px); }
.card:nth-child(5n) { transform: translateZ(-20px); }
.card:nth-child(7n) { transform: translateZ(60px); }
```

**Why rotateX 8° / rotateY -4° is the golden ratio**:
- Greater than 10° → elements distort too much, look like they're "falling over"
- Less than 5° → looks like "shearing" rather than "perspective"
- Asymmetric ratio of 8° × -4° simulates the natural angle of "camera looking down from upper-left of desk"

### 4.8 Diagonal Pan · Moving XY Simultaneously

Camera movement is not purely vertical or horizontal — **moves XY simultaneously** to simulate diagonal movement:

```js
const panX = Math.sin(flowT * 0.22) * 40;
const panY = Math.sin(flowT * 0.35) * 30;
stage.style.transform = `
  translate(-50%, -50%)
  rotateX(8deg) rotateY(-4deg)
  translate3d(${panX}px, ${panY}px, 0)
`;
```

**Key**: X and Y frequencies differ (0.22 vs 0.35), preventing Lissajous cycles from becoming regular.

---

## 5. Scene Recipes (Three Narrative Templates)

Choose the one that best fits your product — don't mix and match.

### Recipe A · Apple Keynote Dramatic Style (Claude Design type)

**Suited for**: Major version releases, hero animations, visual impact as priority
**Rhythm**: Slow-Fast-Boom-Stop strong arc
**Easing**: Full `expoOut` + small amount of `overshoot`
**SFX density**: High (~0.4/s), SFX pitch tuned to BGM scale
**BGM**: IDM / minimalist tech electronic, calm + precise
**Convergence**: Camera rapid pull-back → drop → Logo morph → ethereal single note → hard stop

### Recipe B · One-Take Tool Style (Claude Code type)

**Suited for**: Developer tools, productivity apps, flow-state scenarios
**Rhythm**: Continuously stable flow, no distinct peaks
**Easing**: `spring` physics + `expoOut`
**SFX density**: **0** (rely purely on BGM to drive edit rhythm)
**BGM**: Lo-fi Hip-hop / Boom-bap, 85-90 BPM
**Core technique**: Key UI actions land on BGM kick/snare transients — "**music rhythm is the interaction sound effect**"

### Recipe C · Office Productivity Narrative Style (Claude for Word type)

**Suited for**: Enterprise software, document/spreadsheet/calendar type, professional feel as priority
**Rhythm**: Multi-scene hard cuts + Dolly In/Out
**Easing**: `overshoot` (toggle) + `expoOut` (panels)
**SFX density**: Medium (~0.3/s), UI clicks as main
**BGM**: Jazzy Instrumental, minor key, BPM 90-95
**Core highlight**: One scene must be the "whole-film highlight" — 3D pop-out / lifting off the plane

---

## 6. Counter-examples · What Makes AI Slop

| Anti-pattern | Why it's wrong | Correct approach |
|---|---|---|
| `transition: all 0.3s ease` | `ease` is a cousin of linear, all elements at same speed | Use `expoOut` + per-element stagger |
| All entrances are `opacity 0→1` | No sense of movement direction | Pair with `translateY 10→0` + Anticipation |
| Logo fades in | No narrative convergence | Morph / Converge / collapse-expand |
| Mouse moves in straight line | Subconscious machine feel | Bézier arc + Perlin Noise |
| Typing single-char pop (setInterval) | Like old film subtitles | Chunk Reveal, random intervals |
| Key results with no pause | Audience has no reaction time | 0.5s hold before result |
| Focus switch only changes opacity | Non-focused elements remain sharp | opacity + brightness + **blur** |
| Pure black / pure white background | Cyberpunk feel / glare fatigue | Neutral colors with color temperature (follow brand spec) |
| All animations at same speed | No rhythm | Slow-Fast-Boom-Stop |
| Fade out ending | No decisiveness | Hard stop (hold final frame) |

---

## 7. Self-check Checklist (60 seconds before delivery)

- [ ] Narrative structure is Slow-Fast-Boom-Stop, not uniform rhythm?
- [ ] Default easing is `expoOut`, not `easeOut` or `linear`?
- [ ] Toggle / button pop uses `overshoot`?
- [ ] Card / list entrances have 30ms stagger?
- [ ] 0.5s hold before key results?
- [ ] Typing uses Chunk Reveal, not setInterval single character?
- [ ] Focus switch adds blur (not just opacity)?
- [ ] Logo is morph convergence, not fade-in?
- [ ] Background is not pure black / pure white (has color temperature)?
- [ ] Text has serif + sans-serif hierarchy?
- [ ] Ending is hard stop, not fade out?
- [ ] (If mouse present) Mouse trajectory is arc, not straight line?
- [ ] SFX density matches product personality (see Recipes A/B/C)?
- [ ] BGM and SFX have 6-8dB loudness difference? (see `audio-design-rules.md`)

---

## 8. Relationship with Other References

| Reference | Role | Relationship |
|---|---|---|
| `animation-pitfalls.md` | Technical anti-patterns (16 items) | "**What not to do**" · Reverse of this file |
| `animations.md` | Stage/Sprite engine usage | Fundamentals of **how to write** animations |
| `audio-design-rules.md` | Dual-track audio rules | Rules for **audio pairing** with animations |
| `sfx-library.md` | 37 SFX catalog | Sound effect **asset library** |
| `apple-gallery-showcase.md` | Apple gallery display style | Specialized study of a specific motion style |
| **This file** | Positive motion design grammar | "**What you should do**" |

**Invocation order**:
1. Check SKILL.md workflow Step 3's four position questions (decides narrative role and visual temperature)
2. After choosing direction, read this file to determine **motion language** (Recipe A/B/C)
3. When writing code, reference `animations.md` and `animation-pitfalls.md`
4. When exporting video, follow `audio-design-rules.md` + `sfx-library.md`

---

## Appendix · Source Materials

- Anthropic official animation breakdown: `参考动画/BEST-PRACTICES.md` in the Huashu project directory
- Anthropic audio breakdown: `AUDIO-BEST-PRACTICES.md` in the same directory
- 3 reference videos: `ref-{1,2,3}.mp4` + corresponding `gemini-ref-*.md` / `audio-ref-*.md`
- **Strictly filtered**: This reference does not include any specific brand color values, font names, or product names.
  Color/font decisions go through §1.a core asset protocol or the 20 design philosophies.
