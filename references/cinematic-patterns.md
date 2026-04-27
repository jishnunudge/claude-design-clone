# Cinematic Patterns · Best Practices for Workflow Demo Animations

> 5 key patterns for upgrading from "PowerPoint animation" to "launch-event cinematic."
> Distilled from the two cinematic demos (Nuwa workflow + Darwin workflow) in the 2026-04 "Let's Talk Skills" deck. Empirically reproducible.

---

## 0 · What Problem Does This Document Solve

When you need to build a "demo animation for a workflow" (typical scenarios: skill workflows, product onboarding, API call flows, agent task execution), there are two common approaches:

| Paradigm | What It Looks Like | Outcome |
|---|---|---|
| **PowerPoint animation** (bad) | Step 1 fades in → Step 2 fades in → Step 3 fades in, 4 boxes arranged on screen | Audience thinks "it's just a PPT with fade effects" — no wow moment |
| **Cinematic** (good) | Scene-based; only one thing in focus at a time; scenes transition via dissolve / focus pull / morph | Audience thinks "this is a product launch clip" — they want to screenshot and share it |

The difference is **not the animation technology** — it's the **narrative paradigm**. This document explains how to upgrade from the former to the latter.

---

## 1 · Five Core Patterns

### Pattern A · Dashboard + Cinematic Overlay Dual-Layer Structure

**Problem**: A standalone cinematic defaults to a black screen with a single ▶ button. If the user scrolls past without clicking, they see nothing.

**Solution**:
```
DEFAULT state (always visible): Complete static workflow dashboard
  └── Audience immediately understands how this skill / workflow runs

POINT ▶ triggers (overlay rises): 22-second cinematic
  └── Auto-fades back to DEFAULT when done
```

**Implementation notes**:
- `.dash` is visible by default; `.cinema` starts as `opacity: 0; pointer-events: none`
- `.play-cta` is a small gold button in the bottom-right (not a large central overlay)
- Click → `cinema.classList.add('show')` + `dash.classList.add('hide')`
- Run once via `requestAnimationFrame` (not a loop); on finish `endCinematic()` reverses state

**Anti-pattern**: Default = large ▶ overlay covering everything, leaving the page blank until clicked.

---

### Pattern B · Scene-based, NOT Step-based

**Problem**: Breaking the animation into "step 1 shows → step 2 shows → ..." is PowerPoint thinking.

**Solution**: Break into 5 scenes. Each scene is an **independent shot** — the entire screen focuses on only one thing:

| Scene Type | Responsibility | Duration |
|---|---|---|
| 1 · Invoke | User input trigger (terminal typewriter) | 3–4s |
| 2 · Process | Core workflow visualization (unique visual language) | 5–6s |
| 3 · Result/Insight | Key takeaway distilled and visualized | 4–5s |
| 4 · Output | Actual deliverable shown (file / diff / number) | 3–4s |
| 5 · Hero Reveal | Closing hero moment (large text + value proposition) | 4–5s |

**Total runtime ≈ 22 seconds** — this is the battle-tested golden length:
- Shorter than 18s: Product managers haven't settled in before it's over
- Longer than 25s: They lose patience
- 22s is just enough to "hook → develop → resolve → leave an impression"

**Implementation notes**:
- `T = { DURATION: 22.0, s1_in: [0, 0.7], s2_in: [3.8, 4.6], ... }` — a global timeline object
- A single `requestAnimationFrame(render)` drives opacity / transform calculations for all scenes
- Do not use setTimeout chains (brittle, hard to debug)
- Easing must use `expoOut` / `easeOut` / cubic-bezier — **linear is prohibited**

---

### Pattern C · Every Demo Must Have Its Own Visual Language

**Problem**: After finishing the first cinematic, the second one lazily reuses the same template (same orbit + pentagon + typewriter + large hero text) with only the copy changed.

**Consequence**: The audience notices the two skills "look identical," which implicitly says "these two skills are interchangeable."

**Solution**: Each workflow has a different core metaphor, so the visual language must differ.

**Comparison case**:

| Dimension | Nuwa (Persona Distiller) | Darwin (Skill Optimizer) |
|---|---|---|
| Core metaphor | Collect → distill → write | Loop → evaluate → ratchet |
| Visual motion | Float / radiate / pentagon | Cycle / ascend / contrast |
| Scene 2 | 3D Orbit · 8 archive items floating in a perspective ellipse | Spin Loop · token traversing a 6-node ring 5 times |
| Scene 3 | Pentagon · 5 tokens radiating from center | v1 vs v5 · side-by-side diff (red version vs gold version) |
| Scene 4 | SKILL.md typewriter | Hill-Climb · full-screen curve drawing |
| Scene 5 hero | "21 minutes" large serif italic text | Spinning gear ⚙ + "KEPT +1.1" gold tag |

**Validation test**: Cover the copy and look only at the visuals — can you tell which demo is which? If not, it's a lazy shortcut.

---

### Pattern D · Use AI-Generated Real Assets, Not Emoji or Hand-Drawn SVG

**Problem**: 3D orbits / galleries need asset fragments floating around. Emoji (📚🎤) look cheap and have no brand identity; SVG hand-drawn book spines never look like real books.

**Solution**: Use `huashu-gpt-image` to generate a 4×2 grid image (8 thematically relevant objects · white background · 60px breathing space · unified style), then use `extract_grid.py --mode bbox` to cut out 8 individual transparent PNGs.

**Prompt essentials** (detailed prompt patterns in the `huashu-gpt-image` skill):
- IP anchor ("1960s Caltech archive aesthetic" / "Hearthstone-style consistent treatment")
- White background (makes cutting transparent backgrounds easy; grey is atmospheric but harder to cut)
- 4×2 not 5×5 (avoids the last-row compression bug)
- Persona finishing ("You are a Wired magazine curator preparing an exhibition photo")

**Anti-pattern**: Using emoji as icons, using CSS silhouettes instead of product imagery.

---

### Pattern E · BGM + SFX Dual-Track

**Problem**: Animation without sound makes the audience subconsciously feel "this thing is a cheap demo."

**Solution**: Long BGM + 11 SFX cues.

**Universal SFX cue recipe** (applicable to workflow demos):

| Time | SFX | Trigger Scenario |
|---|---|---|
| 0.10s | whoosh | Terminal rises from below |
| 3.0s | enter | Typewriter completes, Enter pressed |
| 4.0s | slide-in | Scene 2 elements enter |
| 5–9s × 5 times | sparkle | Key process nodes (each generation / each token / each data point) |
| 14s | click | Switch to output scene |
| 17.8s | logo-reveal | Hero reveal moment |
| typewriter | type | Triggers every 2 characters (don't make it too dense) |

**Frequency separation**: BGM volume 0.32 (low-freq foundation), SFX volume 0.55 (mid-high freq punch), sparkle 0.7 (needs to stand out), logo-reveal 0.85 (the strongest hero moment).

**User controls**:
- Must have a ▶ to start the overlay (browser autoplay restrictions)
- Small mute button in the top-right (user can toggle silence at any time)
- Do not make it "blares the moment you scroll to this slide"

---

## 2 · Static Dashboard Design Notes

The dashboard is Layer 1 of the dual-layer structure — a product manager who never clicks ▶ should still understand the skill.

**Layout**: 3-column grid (or 1 large + 2 small); each panel answers one question:

| Panel Type | What It Answers | Example |
|---|---|---|
| **Pipeline / Flow Diagram** | "What is the workflow of this skill?" | Nuwa 4-stage pipeline · Darwin autoresearch loop |
| **Snapshot / State** | "What does real output data look like?" | Darwin 8-dimension rubric snapshot |
| **Trajectory / Evolution** | "How does it change across multiple runs?" | Darwin 5-generation hill-climb curve |
| **Examples / Gallery** | "What has it produced?" | Nuwa 21 personas gallery |
| **Strip · Example I/O** | "Input what → output what" | Nuwa example strip: `› nuwa distill feynman → feynman.skill (21 min)` |

**Key constraints**:
- Information density must be sufficient (each panel carries differentiated content)
- But don't stuff it with data slop (every number must mean something)
- Color scheme consistent with the cinematic (same palette family, so transitions aren't jarring)

---

## 3 · Debugging and Development Tools

Any long animation must have three dev tools — without them, debugging becomes chaos.

### Tool 1 · `?seek=N` — Freeze at Second N

```js
const seek = parseFloat(params.get('seek'));
if (!isNaN(seek)) {
  started = true; muted = true;
  frozenT = seek;  // render() uses this t instead of elapsed
  cinema.classList.add('show'); dash.classList.add('hide');
}

// Inside render():
let t = frozenT !== null ? frozenT : (elapsed % T.DURATION);
```

Usage: `http://.../slide.html?seek=12` jumps directly to the frame at second 12 — no waiting for playback.

### Tool 2 · `?autoplay=1` — Skip the ▶ Overlay

Useful for Playwright automated screenshot tests and for force-starting when embedded in an iframe.

### Tool 3 · Manual REPLAY Button

Small button in the top-right; users / developers can replay as many times as needed. CSS:

```css
.replay{position:absolute;top:18px;right:18px;background:rgba(212,165,116,0.1);
  border:1px solid rgba(212,165,116,0.3);color:#D4A574;
  font-family:monospace;font-size:10px;letter-spacing:.28em;text-transform:uppercase;
  padding:6px 12px;border-radius:1px;cursor:pointer;backdrop-filter:blur(6px);z-index:6}
```

---

## 4 · iframe Embedding Pitfalls (When the Cinematic Lives Inside a Deck)

### Pitfall 1 · Parent Window Click Zones Intercept Buttons Inside the iframe

If the deck `index.html` adds "left and right 22vw transparent click zones for page turns," they **overlap the ▶ play button inside the iframe** — the user clicks the button and it becomes "next page."

**Fix**: Add `top: 12vh; bottom: 25vh` to the click zones, leaving the top and bottom 25% unblocked, so both the central ▶ and the bottom-right ▶ inside the iframe can be reached.

### Pitfall 2 · iframe Focus Steal Drops Keyboard Events

After the user clicks inside the iframe, focus is inside it, and the parent window no longer receives ←/→ keyboard events.

**Fix**:
```js
iframe.addEventListener('load', () => {
  // Inject a keyboard forwarder
  const doc = iframe.contentDocument;
  doc.addEventListener('keydown', (e) => {
    window.dispatchEvent(new KeyboardEvent('keydown', { key: e.key, ... }));
  });
  // Pull focus back to parent window after click
  doc.addEventListener('click', () => setTimeout(() => window.focus(), 0));
});
```

### Pitfall 3 · file:// vs https:// Behavior Differences

A cinematic that works locally under file:// may break after deployment because:
- Under file://, iframe contentDocument is same-origin
- Under https:// it is also same-origin (if same host), but audio autoplay restrictions are stricter

**Fix**:
- Before deploying, test with `python3 -m http.server` on a local HTTP server
- BGM must only call `bgm.play()` after the user clicks ▶ — do not autoplay on page load

---

## 5 · Anti-Pattern Quick Reference

| ❌ Anti-Pattern | ✅ Correct Pattern |
|---|---|
| Default = black screen ▶ overlay | Default = static dashboard; ▶ is supplementary |
| 4 steps side-by-side, fading in on the same screen | 5 scenes, full-screen transitions, each focuses on one thing |
| Reuse template with different copy for different demos | Each demo has an independent visual language (can distinguish when copy is hidden) |
| emoji / hand-drawn SVG as assets | gpt-image-2 large image + extract_grid cutouts |
| No BGM, no SFX | BGM + 11 SFX cues, dual-track |
| setTimeout chains for scheduling | requestAnimationFrame + global timeline T object |
| Linear animation | Expo / cubic-bezier easing |
| No dev tools | `?seek=N` + `?autoplay=1` + REPLAY button |
| Buttons inside iframe swallowed by parent click zone | Add top/bottom margins to click zones to make room for buttons |

---

## 6 · Time Budget

Following this pattern set, one complete cinematic demo (including dashboard):

| Task | Time |
|---|---|
| Design 5-scene narrative + visual language | 30 min (requires care — determines independence) |
| Dashboard static layout + content | 1 hour |
| Cinematic 5 scenes implementation | 1.5 hours |
| Audio cue timing + replay button | 30 min |
| Playwright screenshot validation at 5 key moments | 15 min |
| **Total per demo** | **3–4 hours** |

The second demo reuses the framework but **the visual language must be independent** — approximately 2–3 hours.
