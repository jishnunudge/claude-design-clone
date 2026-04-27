# Animations: Timeline Animation Engine

Read this when making animations / motion design HTML. Covers principles, usage, and common patterns.

## Core Pattern: Stage + Sprite

Our animation system (`assets/animations.jsx`) provides a timeline-driven engine:

- **`<Stage>`**: Container for the entire animation, automatically provides auto-scale (fit viewport) + scrubber + play/pause/loop controls
- **`<Sprite start end>`**: Time segment. A Sprite only displays between `start` and `end` time. Internally can read its own local progress `t` (0→1) via the `useSprite()` hook
- **`useTime()`**: Reads the current global time (in seconds)
- **`Easing.easeInOut` / `Easing.easeOut` / ...**:  Easing functions
- **`interpolate(t, from, to, easing?)`**: Interpolates based on t

This pattern is inspired by Remotion/After Effects, but lightweight and zero-dependency.

## Getting Started

```html
<script type="text/babel" src="animations.jsx"></script>
<script type="text/babel">
  const { Stage, Sprite, useTime, useSprite, Easing, interpolate } = window.Animations;

  function Title() {
    const { t } = useSprite();  // local progress 0→1
    const opacity = interpolate(t, [0, 1], [0, 1], Easing.easeOut);
    const y = interpolate(t, [0, 1], [40, 0], Easing.easeOut);
    return (
      <h1 style={{ 
        opacity, 
        transform: `translateY(${y}px)`,
        fontSize: 120,
        fontWeight: 900,
      }}>
        Hello.
      </h1>
    );
  }

  function Scene() {
    return (
      <Stage duration={10}>  {/* 10-second animation */}
        <Sprite start={0} end={3}>
          <Title />
        </Sprite>
        <Sprite start={2} end={5}>
          <SubTitle />
        </Sprite>
        {/* ... */}
      </Stage>
    );
  }

  const root = ReactDOM.createRoot(document.getElementById('root'));
  root.render(<Scene />);
</script>
```

## Common Animation Patterns

### 1. Fade In / Fade Out

```jsx
function FadeIn({ children }) {
  const { t } = useSprite();
  const opacity = interpolate(t, [0, 0.3], [0, 1], Easing.easeOut);
  return <div style={{ opacity }}>{children}</div>;
}
```

**Note on range**: `[0, 0.3]` means the fade-in completes in the first 30% of the sprite's duration, then holds at opacity=1.

### 2. Slide In

```jsx
function SlideIn({ children, from = 'left' }) {
  const { t } = useSprite();
  const progress = interpolate(t, [0, 0.4], [0, 1], Easing.easeOut);
  const offset = (1 - progress) * 100;
  const directions = {
    left: `translateX(-${offset}px)`,
    right: `translateX(${offset}px)`,
    top: `translateY(-${offset}px)`,
    bottom: `translateY(${offset}px)`,
  };
  return (
    <div style={{
      transform: directions[from],
      opacity: progress,
    }}>
      {children}
    </div>
  );
}
```

### 3. Character-by-character Typewriter

```jsx
function Typewriter({ text }) {
  const { t } = useSprite();
  const charCount = Math.floor(text.length * Math.min(t * 2, 1));
  return <span>{text.slice(0, charCount)}</span>;
}
```

### 4. Number Count-up

```jsx
function CountUp({ from = 0, to = 100, duration = 0.6 }) {
  const { t } = useSprite();
  const progress = interpolate(t, [0, duration], [0, 1], Easing.easeOut);
  const value = Math.floor(from + (to - from) * progress);
  return <span>{value.toLocaleString()}</span>;
}
```

### 5. Segmented Explanation (Typical Tutorial Animation)

```jsx
function Scene() {
  return (
    <Stage duration={20}>
      {/* Phase 1: Show the problem */}
      <Sprite start={0} end={4}>
        <Problem />
      </Sprite>

      {/* Phase 2: Show the approach */}
      <Sprite start={4} end={10}>
        <Approach />
      </Sprite>

      {/* Phase 3: Show the result */}
      <Sprite start={10} end={16}>
        <Result />
      </Sprite>

      {/* Caption displayed throughout */}
      <Sprite start={0} end={20}>
        <Caption />
      </Sprite>
    </Stage>
  );
}
```

## Easing Functions

Preset easing curves:

| Easing | Characteristic | Used for |
|--------|------|------|
| `linear` | Constant speed | Scrolling text, continuous animation |
| `easeIn` | Slow → Fast | Exit / disappear |
| `easeOut` | Fast → Slow | Entrance / appear |
| `easeInOut` | Slow → Fast → Slow | Position changes |
| **`expoOut`** ⭐ | **Exponential ease-out** | **Anthropic-grade main easing** (physical weight feel) |
| **`overshoot`** ⭐ | **Elastic bounce** | **Toggle / button pop / emphasis interaction** |
| `spring` | Spring physics | Interaction feedback, geometry homing |
| `anticipation` | Reverse first then forward | Emphasizing actions |

**Default main easing is `expoOut`** (not `easeOut`) — see `animation-best-practices.md` §2.
Entrance uses `expoOut`, exit uses `easeIn`, toggle uses `overshoot` — the basic rules of Anthropic-grade animation.

## Rhythm and Duration Guide

### Micro-interactions (0.1-0.3 seconds)
- Button hover
- Card expand
- Tooltip appear

### UI Transitions (0.3-0.8 seconds)
- Page transitions
- Modal appear
- List item join

### Narrative Animation (2-10 seconds per segment)
- One phase of concept explanation
- Data chart reveal
- Scene transition

### Single narrative animation segment should not exceed 10 seconds
Human attention is limited. 10 seconds to explain one thing, then move to the next.

## Design Thinking Order for Animations

### 1. Content/story first, animation second

**Wrong**: First want to make fancy animation, then stuff content into it
**Right**: First figure out what information to convey, then use animation to serve that information

Animation is **signal**, not **decoration**. A fade-in emphasizes "this is important, please look" — if everything fades in, the signal becomes ineffective.

### 2. Write the timeline by scene

```
0:00 - 0:03   Problem appears (fade in)
0:03 - 0:06   Problem magnified/expanded (zoom+pan)
0:06 - 0:09   Solution appears (slide in from right)
0:09 - 0:12   Solution expanded with explanation (typewriter)
0:12 - 0:15   Result demo (counter up + chart reveal)
0:15 - 0:18   Summary sentence (static, read for 3 seconds)
0:18 - 0:20   CTA or fade out
```

Write the timeline first, then write the components.

### 3. Assets first

Images/icons/fonts needed for the animation should be prepared **first**. Don't go hunting for assets halfway through — it breaks the flow.

## Common Issues

**Animation stutters**
→ Mainly layout thrashing. Use `transform` and `opacity`; don't change `top`/`left`/`width`/`height`/`margin`. The browser GPU-accelerates `transform`.

**Animation is too fast to see**
→ A person needs 100-150ms to read a character, 300-500ms for a word. If you're telling a story with text, leave at least 3 seconds per sentence.

**Animation is too slow, audience is bored**
→ Interesting visual changes should be dense. Static frames that exceed 5 seconds become boring.

**Multiple animations interfering with each other**
→ Use CSS's `will-change: transform` to tell the browser in advance that this element will move, reducing reflow.

**Recording to video**
→ Use the skill's built-in toolchain (one command outputs three formats): see `video-export.md`
- `scripts/render-video.js` — HTML → 25fps MP4 (Playwright + ffmpeg)
- `scripts/convert-formats.sh` — 25fps MP4 → 60fps MP4 + optimized GIF
- Want more precise frame rendering? Make render(t) a pure function — see `animation-pitfalls.md` item 5

## Integration with Video Tools

This skill creates **HTML animations** (running in the browser). If the final output needs to be video material:

- **Short animation/concept demo**: Make HTML animation using these methods → screen record
- **Long video/narrative**: This skill focuses on HTML animation; for long video use AI video generation skills or professional video software
- **Motion graphics**: Professional After Effects/Motion Canvas is more appropriate

## On Libraries Like Popmotion

If you genuinely need physics animation (spring, decay, keyframes with precise timing) that our engine can't handle, you can fall back to Popmotion:

```html
<script src="https://unpkg.com/popmotion@11.0.5/dist/popmotion.min.js"></script>
```

But **try our engine first**. It's sufficient for 90% of cases.
