# Cinematic Patterns · Best Practices for Workflow Demo Animations

> 5 key patterns for upgrading from "PowerPoint animation" to "launch-event cinematic."
> Distilled from two cinematic demos (Nuwa workflow + Darwin workflow) in the 2026-04 "Let's Talk Skills" deck. Empirically reproducible.

---

## 0 · What Problem Does This Document Solve

When building a "demo animation for a workflow" (skill workflows, product onboarding, API call flows, agent task execution):

| Paradigm | What It Looks Like | Outcome |
|---|---|---|
| **PowerPoint animation** (bad) | Step 1 fades in → Step 2 fades in → ..., 4 boxes on screen | Audience thinks "PPT with fade effects" — no wow moment |
| **Cinematic** (good) | Scene-based; one thing in focus at a time; scenes transition via dissolve / focus pull / morph | Audience thinks "product launch clip" — wants to screenshot and share |

Difference is **not the technology** — it's the **narrative paradigm**.

---

## 1 · Five Core Patterns

### Pattern A · Dashboard + Cinematic Overlay Dual-Layer Structure

**Problem**: Standalone cinematic defaults to black screen with ▶. User scrolls past, sees nothing.

**Solution**:
```
DEFAULT state (always visible): Complete static workflow dashboard
  └── Audience immediately understands how the skill / workflow runs

POINT ▶ triggers (overlay rises): 22-second cinematic
  └── Auto-fades back to DEFAULT when done
```

**Implementation notes**:
- `.dash` visible by default; `.cinema` starts as `opacity: 0; pointer-events: none`
- `.play-cta` is small gold button in bottom-right (not large central overlay)
- Click → `cinema.classList.add('show')` + `dash.classList.add('hide')`
- Run once via `requestAnimationFrame` (not loop); on finish `endCinematic()` reverses state

**Anti-pattern**: Default = large ▶ overlay covering everything, page blank until clicked.

---

### Pattern B · Scene-based, NOT Step-based

**Problem**: "Step 1 shows → step 2 shows → ..." is PowerPoint thinking.

**Solution**: Break into 5 scenes. Each is an **independent shot** — entire screen focuses on one thing:

| Scene Type | Responsibility | Duration |
|---|---|---|
| 1 · Invoke | User input trigger (terminal typewriter) | 3–4s |
| 2 · Process | Core workflow visualization (unique visual language) | 5–6s |
| 3 · Result/Insight | Key takeaway distilled and visualized | 4–5s |
| 4 · Output | Actual deliverable shown (file / diff / number) | 3–4s |
| 5 · Hero Reveal | Closing hero moment (large text + value prop) | 4–5s |

**Total runtime ≈ 22 seconds** — battle-tested golden length:
- <18s: PMs haven't settled in before it's over
- >25s: They lose patience
- 22s is "hook → develop → resolve → leave impression"

**Implementation notes**:
- `T = { DURATION: 22.0, s1_in: [0, 0.7], s2_in: [3.8, 4.6], ... }` — global timeline object
- Single `requestAnimationFrame(render)` drives all opacity / transform calculations
- Never use setTimeout chains (brittle, hard to debug)
- Easing must use `expoOut` / `easeOut` / cubic-bezier — **linear is prohibited**

---

### Pattern C · Every Demo Must Have Its Own Visual Language

**Problem**: Second cinematic lazily reuses same template (orbit + pentagon + typewriter + hero text), only copy changed.

**Consequence**: Audience notices two skills "look identical" — implicitly says they're interchangeable.

**Solution**: Each workflow has a different core metaphor; visual language must differ.

| Dimension | Nuwa (Persona Distiller) | Darwin (Skill Optimizer) |
|---|---|---|
| Core metaphor | Collect → distill → write | Loop → evaluate → ratchet |
| Visual motion | Float / radiate / pentagon | Cycle / ascend / contrast |
| Scene 2 | 3D Orbit · 8 archive items floating in perspective ellipse | Spin Loop · token traversing 6-node ring 5× |
| Scene 3 | Pentagon · 5 tokens radiating from center | v1 vs v5 · side-by-side diff (red vs gold) |
| Scene 4 | SKILL.md typewriter | Hill-Climb · full-screen curve drawing |
| Scene 5 hero | "21 minutes" large serif italic | Spinning gear ⚙ + "KEPT +1.1" gold tag |

**Validation test**: Cover copy, look only at visuals — can you tell which demo is which? If not, it's lazy.

---

### Pattern D · Use AI-Generated Real Assets, Not Emoji or Hand-Drawn SVG

**Problem**: 3D orbits / galleries need asset fragments. Emoji look cheap; SVG hand-drawn books never look real.

**Solution**: Use `huashu-gpt-image` to generate 4×2 grid (8 thematic objects · white bg · 60px breathing space · unified style), then use `extract_grid.py --mode bbox` to cut 8 individual transparent PNGs.

**Prompt essentials** (detailed patterns in `huashu-gpt-image` skill):
- IP anchor ("1960s Caltech archive aesthetic" / "Hearthstone-style consistent treatment")
- White background (easy transparent cut; grey is atmospheric but harder)
- 4×2 not 5×5 (avoids last-row compression bug)
- Persona finishing ("You are a Wired magazine curator preparing an exhibition photo")

**Anti-pattern**: Emoji as icons, CSS silhouettes instead of product imagery.

---

### Pattern E · BGM + SFX Dual-Track

**Problem**: Animation without sound makes audience subconsciously feel "cheap demo."

**Solution**: Long BGM + 11 SFX cues.

**Universal SFX cue recipe** (applicable to workflow demos):

| Time | SFX | Trigger Scenario |
|---|---|---|
| 0.10s | whoosh | Terminal rises from below |
| 3.0s | enter | Typewriter completes, Enter pressed |
| 4.0s | slide-in | Scene 2 elements enter |
| 5–9s × 5 | sparkle | Key process nodes (each generation / token / data point) |
| 14s | click | Switch to output scene |
| 17.8s | logo-reveal | Hero reveal moment |
| typewriter | type | Every 2 characters (don't overdo density) |

**Frequency separation**: BGM 0.32 (low-freq foundation), SFX 0.55 (mid-high punch), sparkle 0.7 (stands out), logo-reveal 0.85 (strongest hero moment).

**User controls**:
- Must have ▶ to start overlay (browser autoplay restrictions)
- Small mute button top-right (toggle silence anytime)
- Do not autoplay on scroll

---

## 2 · Static Dashboard Design Notes

Dashboard is Layer 1 of dual-layer structure — PM who never clicks ▶ should still understand the skill.

**Layout**: 3-column grid (or 1 large + 2 small); each panel answers one question:

| Panel Type | What It Answers | Example |
|---|---|---|
| **Pipeline / Flow Diagram** | "What is the workflow?" | Nuwa 4-stage pipeline · Darwin autoresearch loop |
| **Snapshot / State** | "What does real output data look like?" | Darwin 8-dimension rubric snapshot |
| **Trajectory / Evolution** | "How does it change over runs?" | Darwin 5-generation hill-climb curve |
| **Examples / Gallery** | "What has it produced?" | Nuwa 21 personas gallery |
| **Strip · Example I/O** | "Input what → output what" | `› nuwa distill feynman → feynman.skill (21 min)` |

**Key constraints**:
- Sufficient information density (each panel has differentiated content)
- Don't stuff with data slop — every number must mean something
- Color scheme consistent with cinematic (same palette, transitions aren't jarring)

---

## 3 · Debugging and Development Tools

Any long animation needs three dev tools — without them, debugging is chaos.

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

Usage: `http://.../slide.html?seek=12` jumps to frame at second 12 — no waiting.

### Tool 2 · `?autoplay=1` — Skip ▶ Overlay

Useful for Playwright automated screenshots and force-starting in iframes.

### Tool 3 · Manual REPLAY Button

```css
.replay{position:absolute;top:18px;right:18px;background:rgba(212,165,116,0.1);
  border:1px solid rgba(212,165,116,0.3);color:#D4A574;
  font-family:monospace;font-size:10px;letter-spacing:.28em;text-transform:uppercase;
  padding:6px 12px;border-radius:1px;cursor:pointer;backdrop-filter:blur(6px);z-index:6}
```

---

## 4 · iframe Embedding Pitfalls

### Pitfall 1 · Parent Click Zones Intercept Buttons Inside iframe

If the deck `index.html` adds "left/right 22vw transparent click zones for page turns," they **overlap the ▶ button inside the iframe**.

**Fix**: Add `top: 12vh; bottom: 25vh` to click zones, leaving top/bottom 25% unblocked so both ▶ buttons are reachable.

### Pitfall 2 · iframe Focus Steal Drops Keyboard Events

After clicking inside iframe, parent window no longer receives ←/→ keyboard events.

**Fix**:
```js
iframe.addEventListener('load', () => {
  // Inject keyboard forwarder
  const doc = iframe.contentDocument;
  doc.addEventListener('keydown', (e) => {
    window.dispatchEvent(new KeyboardEvent('keydown', { key: e.key, ... }));
  });
  // Pull focus back to parent after click
  doc.addEventListener('click', () => setTimeout(() => window.focus(), 0));
});
```

### Pitfall 3 · file:// vs https:// Behavior Differences

Works locally under file:// but breaks after deploy because audio autoplay restrictions are stricter under https://.

**Fix**:
- Test with `python3 -m http.server` before deploying
- BGM must only call `bgm.play()` after user clicks ▶ — never autoplay on load

---

## 5 · Anti-Pattern Quick Reference

| ❌ Anti-Pattern | ✅ Correct Pattern |
|---|---|
| Default = black screen ▶ overlay | Default = static dashboard; ▶ is supplementary |
| 4 steps side-by-side, fading in on same screen | 5 scenes, full-screen transitions, each focuses on one thing |
| Reuse template with different copy | Each demo has independent visual language (distinguishable with copy hidden) |
| emoji / hand-drawn SVG as assets | gpt-image-2 large image + extract_grid cutouts |
| No BGM, no SFX | BGM + 11 SFX cues, dual-track |
| setTimeout chains for scheduling | requestAnimationFrame + global timeline T object |
| Linear animation | Expo / cubic-bezier easing |
| No dev tools | `?seek=N` + `?autoplay=1` + REPLAY button |
| Buttons inside iframe swallowed by parent click zone | Add top/bottom margins to click zones |

---

## 6 · Time Budget

One complete cinematic demo (including dashboard):

| Task | Time |
|---|---|
| Design 5-scene narrative + visual language | 30 min (determines independence — requires care) |
| Dashboard static layout + content | 1 hour |
| Cinematic 5 scenes implementation | 1.5 hours |
| Audio cue timing + replay button | 30 min |
| Playwright screenshot validation at 5 key moments | 15 min |
| **Total per demo** | **3–4 hours** |

Second demo reuses framework but **visual language must be independent** — approximately 2–3 hours.
