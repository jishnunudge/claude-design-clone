# Animation Pitfalls: Bugs Encountered in HTML Animation and How to Avoid Them

Most common bugs when making animations and how to avoid them. Every rule comes from a real failure case.

Read through this before writing an animation — it can save you a full iteration cycle.

## 1. Layered Layout — `position: relative` is a Default Obligation

**The trap**: A sentence-wrap element wraps 3 bracket-layer elements (`position: absolute`). Without `position: relative` on sentence-wrap, the absolute bracket uses `.canvas` as its coordinate system and floats 200px off the bottom of the screen.

**Rules**:
- Any container holding `position: absolute` children **must** explicitly have `position: relative`
- Even without visual "offset" needed, write `position: relative` as a coordinate system anchor
- If writing `.parent { ... }` and its children include `.child { position: absolute }`, instinctively add relative to the parent

**Quick check**: Every time `position: absolute` appears, count up through ancestors to ensure the nearest positioned ancestor is the coordinate system *you want*.

## 2. Character Traps — Don't Rely on Rare Unicode

**The trap**: Wanted to use `␣` (U+2423 OPEN BOX) to visualize "space token." Noto Serif SC / Cormorant Garamond don't have this glyph, rendering as blank/tofu — audience sees nothing.

**Rules**:
- **Every character in an animation must exist in your chosen font**
- Common rare character blacklist: `␣ ␀ ␐ ␋ ␨ ↩ ⏎ ⌘ ⌥ ⌃ ⇧ ␦ ␖ ␛`
- To express meta-characters like "space / return / tab", use **CSS-constructed semantic boxes**:
  ```html
  <span class="space-key">Space</span>
  ```
  ```css
  .space-key {
    display: inline-flex;
    padding: 4px 14px;
    border: 1.5px solid var(--accent);
    border-radius: 4px;
    font-family: monospace;
    font-size: 0.3em;
    letter-spacing: 0.2em;
    text-transform: uppercase;
  }
  ```
- Emoji must also be validated: some emoji in fonts other than Noto Emoji will fall back to gray boxes. Use `emoji` font-family or SVG

## 3. Data-driven Grid/Flex Templates

**The trap**: Code has `const N = 6` tokens, but CSS hardcodes `grid-template-columns: 80px repeat(5, 1fr)`. The 6th token has no column — entire matrix is misaligned.

**Rules**:
- When count comes from a JS array (`TOKENS.length`), CSS template should also be data-driven
- Option A: Inject via CSS variable from JS
  ```js
  el.style.setProperty('--cols', N);
  ```
  ```css
  .grid { grid-template-columns: 80px repeat(var(--cols), 1fr); }
  ```
- Option B: Use `grid-auto-flow: column` and let the browser auto-extend
- **Prohibit the combination of "fixed number + JS constant"** — changing N won't sync CSS

## 4. Transition Discontinuity — Scene Switches Must Be Continuous

**The trap**: zoom1 (13-19s) → zoom2 (19.2-23s). Main sentence already hidden, zoom1 fade out (0.6s) + zoom2 fade in (0.6s) + stagger delay (0.2s+) = approximately 1 second of pure blank screen. Audience thinks the animation froze.

**Rules**:
- When switching scenes consecutively, fade out and fade in must **cross-overlap**, not wait for previous to fully disappear before starting next
  ```js
  // Bad:
  if (t >= 19) hideZoom('zoom1');      // 19.0s out
  if (t >= 19.4) showZoom('zoom2');    // 19.4s in → 0.4s blank in between

  // Good:
  if (t >= 18.6) hideZoom('zoom1');    // start fade out 0.4s early
  if (t >= 18.6) showZoom('zoom2');    // fade in simultaneously (cross-fade)
  ```
- Or use an "anchor element" (e.g., the main sentence) as a visual link between scenes — it briefly reappears during the zoom switch
- Carefully calculate CSS transition duration to avoid next transition triggering before current one finishes

## 5. Pure Render Principle — Animation State Should Be Seekable

**The trap**: Using `setTimeout` + `fireOnce(key, fn)` to chain animation state triggers. Works fine for normal playback, but during frame-by-frame recording / seeking to any time point, previously-executed setTimeouts cannot "go back in time."

**Rules**:
- `render(t)` function should ideally be a **pure function**: given t, outputs the unique DOM state
- If side effects are necessary (like class toggling), use a `fired` set with explicit reset:
  ```js
  const fired = new Set();
  function fireOnce(key, fn) { if (!fired.has(key)) { fired.add(key); fn(); } }
  function reset() { fired.clear(); /* clear all .show classes */ }
  ```
- Expose `window.__seek(t)` for Playwright / debugging:
  ```js
  window.__seek = (t) => { reset(); render(t); };
  ```
- Animation-related setTimeouts should not span >1 second, otherwise seeking backward will cause chaos

## 6. Measuring Before Fonts Load = Measuring Wrong

**The trap**: On DOMContentLoaded, immediately calling `charRect(idx)` to measure bracket positions. Fonts haven't loaded yet, each character width is the fallback font width, positions are all wrong. Once fonts load (~500ms later), the bracket's `left: Xpx` still holds the old value — permanently offset.

**Rules**:
- Any layout code depending on DOM measurement (`getBoundingClientRect`, `offsetWidth`) **must** be wrapped in `document.fonts.ready.then()`
  ```js
  document.fonts.ready.then(() => {
    requestAnimationFrame(() => {
      buildBrackets(...);  // fonts are ready now, measurement is accurate
      tick();              // animation starts
    });
  });
  ```
- Extra `requestAnimationFrame` gives the browser one frame to commit layout
- If using Google Fonts CDN, `<link rel="preconnect">` speeds up first load

## 7. Recording Preparation — Prepare Handles for Video Export

**The trap**: Playwright `recordVideo` defaults to 25fps, starts recording from context creation. The first 2 seconds of page loading and font loading get recorded. Delivered video has 2 seconds of blank/white flash at the start.

**Rules**:
- Provide `render-video.js` tooling to handle: warmup navigate → reload to restart animation → wait for duration → ffmpeg trim head + convert to H.264 MP4
- **Frame 0** of the animation must be the complete initial state with layout already in place (not blank or loading)
- Want 60fps? Use ffmpeg `minterpolate` post-processing — don't rely on browser source frame rate
- Want GIF? Two-stage palette (`palettegen` + `paletteuse`), can compress a 30s 1080p animation to 3MB

See `video-export.md` for complete script invocation.

## 8. Batch Export — tmp Directory Must Include PID to Prevent Concurrent Conflicts

**The trap**: Using `render-video.js` with 3 processes recording 3 HTMLs in parallel. Because TMP_DIR is only named with `Date.now()`, 3 processes starting at the same millisecond share the same tmp directory. The first process to finish cleans up tmp, the other two get `ENOENT` when reading the directory — all crash.

**Rules**:
- Any temporary directories multiple processes may share must include **PID or random suffix** in the name:
  ```js
  const TMP_DIR = path.join(DIR, '.video-tmp-' + Date.now() + '-' + process.pid);
  ```
- If you actually want parallel multi-file processing, use shell's `&` + `wait` rather than forking within a single node script
- When batch-recording multiple HTMLs, conservative approach: run **serially** (2 can run in parallel, 3+ should queue up)

## 9. Progress Bar / Replay Button in Recording — Chrome Elements Polluting the Video

**The trap**: Animation HTML has `.progress` progress bar, `.replay` replay button, `.counter` timestamp — convenient for human playback debugging. When rendered to MP4 for delivery, these elements appear at the bottom of the video, like developer tools got screenshotted into it.

**Rules**:
- Manage "chrome elements" for human use (progress bar / replay button / footer / masthead / counter / phase labels) separately from the video content body
- **Establish a class name convention** `.no-record`: any element with this class is automatically hidden by the recording script
- The script side (`render-video.js`) by default injects CSS to hide common chrome class names:
  ```
  .progress .counter .phases .replay .masthead .footer .no-record [data-role="chrome"]
  ```
- Use Playwright's `addInitScript` for injection (takes effect before each navigate, works reliably on reload too)
- When you want to see the original HTML (with chrome), add `--keep-chrome` flag

## 10. Animation Repeating at Recording Start — Warmup Frame Leakage

**The trap**: Old `render-video.js` flow: `goto → wait fonts 1.5s → reload → wait duration`. Recording starts from context creation, animation already played through part of it during warmup, then reload restarts from 0. Result: the video's first few seconds show "animation mid-way + cut + animation starting from 0" — strong repeat feeling.

**Rules**:
- **Warmup and Record must use separate contexts**:
  - Warmup context (no `recordVideo` option): only responsible for loading url, waiting for fonts, then close
  - Record context (with `recordVideo`): fresh state start, animation recorded from t=0
- ffmpeg `-ss trim` can only clip the small amount of Playwright startup latency (~0.3s) — **cannot** be used to hide warmup frames; the source must be clean
- Closing the recording context = webm file written to disk, this is a Playwright constraint
- Related code pattern:
  ```js
  // Phase 1: warmup (throwaway)
  const warmupCtx = await browser.newContext({ viewport });
  const warmupPage = await warmupCtx.newPage();
  await warmupPage.goto(url, { waitUntil: 'networkidle' });
  await warmupPage.waitForTimeout(1200);
  await warmupCtx.close();

  // Phase 2: record (fresh)
  const recordCtx = await browser.newContext({ viewport, recordVideo });
  const page = await recordCtx.newPage();
  await page.goto(url, { waitUntil: 'networkidle' });
  await page.waitForTimeout(DURATION * 1000);
  await page.close();
  await recordCtx.close();
  ```

## 11. Don't Draw "Fake Chrome" On-screen — Decorative Player UI Conflicts with Real Chrome

**The trap**: The animation uses a `Stage` component that already comes with scrubber + timecode + pause button (these belong to `.no-record` chrome and are automatically hidden on export). I then drew a "magazine page number style decorative progress bar" at the bottom reading "`00:60 ──── CLAUDE-DESIGN / ANATOMY`". **Result**: Users see two progress bars — one is the Stage controller, one is the decorative one. Visually they completely clash — deemed a bug.

**Rules**:

- Stage already provides: scrubber + timecode + pause/replay buttons. **Do not draw** additional progress indicators, current timecode, copyright attribution bars, or chapter counters inside the canvas — they either conflict with chrome, or are filler slop (violating the "earn its place" principle).
- "Page number feel," "magazine feel," "bottom attribution bar" — these **decorative desires** are high-frequency fillers automatically added by AI. Every occurrence warrants vigilance — does it genuinely convey irreplaceable information? Or is it just filling blank space?
- If you firmly believe a certain bottom strip must exist (e.g., the animation's theme is literally about player UI), it must be **narratively necessary** and **visually distinct from the Stage scrubber** (different position, different form, different tone).

**Element attribution test** (every element drawn into the canvas must answer):

| What it belongs to | Treatment |
|------------|------|
| Narrative content of a specific scene | OK, keep it |
| Global chrome (for control/debugging) | Add `.no-record` class, hide on export |
| **Belongs to neither any scene nor chrome** | **Delete.** Ownerless element — inevitably filler slop |

**Self-check (3 seconds before delivery)**:

- Is there anything in the frame that "looks like video player UI" (horizontal progress bar, timecode, control button shapes)?
- If so, does removing it damage the narrative? If not damaged, delete.
- Does the same type of information (progress/time/attribution) appear twice? Consolidate to one place in chrome.

**Counter-examples**: Drawing `00:42 ──── PROJECT NAME` at the bottom, drawing "CH 03 / 06" chapter count in the lower right corner, drawing version number "v0.3.1" along the edge — all are fake chrome filler.

## 12. Recording Blank Prefix + Recording Start Offset — `__ready` × tick × lastTick Triple Trap

**Trap (A · blank prefix)**: A 60-second animation exported as MP4 has 2-3 seconds of blank screen at the beginning. `ffmpeg --trim=0.3` can't clip it away.

**Trap (B · start offset, real incident 2026-04-20)**: Exporting a 24-second video, user perception is "video doesn't start playing until 19 seconds." The animation actually started recording from t=5, recorded to t=24 then looped back to t=0, then recorded 5 more seconds to end — so the last 5 seconds of the video are the true beginning.

**Root cause** (shared by both traps):

Playwright `recordVideo` starts writing WebM from the moment `newContext()` is called, during which Babel/React/font loading takes L seconds (2-6s). The recording script waits for `window.__ready = true` as the anchor for "animation starts here" — it must strictly pair with animation `time = 0`. Two common mistakes:

| Mistake | Symptom |
|------|------|
| `__ready` is set during `useEffect` or synchronous setup phase (before tick's first frame) | Recording script thinks animation started, but WebM is still recording blank page → **blank prefix** |
| tick's `lastTick = performance.now()` initialized at **script top level** | L seconds of font loading gets counted into first frame `dt`, `time` instantly jumps to L → entire recording is delayed L seconds → **start offset** |

**✅ Correct complete starter tick template** (hand-written animation must use this skeleton):

```js
// ━━━━━━ state ━━━━━━
let time = 0;
let playing = false;   // ❗ don't play by default, wait for fonts ready
let lastTick = null;   // ❗ sentinel — first frame dt is forced to 0 (don't use performance.now())
const fired = new Set();

// ━━━━━━ tick ━━━━━━
function tick(now) {
  if (lastTick === null) {
    lastTick = now;
    window.__ready = true;   // ✅ pair: "recording start" and "animation t=0" in the same frame
    render(0);               // render once more to ensure DOM is ready (fonts are ready at this point)
    requestAnimationFrame(tick);
    return;
  }
  const dt = (now - lastTick) / 1000;   // dt only starts advancing after first frame
  lastTick = now;

  if (playing) {
    let t = time + dt;
    if (t >= DURATION) {
      t = window.__recording ? DURATION - 0.001 : 0;  // don't loop during recording, keep 0.001s to preserve last frame
      if (!window.__recording) fired.clear();
    }
    time = t;
    render(time);
  }
  requestAnimationFrame(tick);
}

// ━━━━━━ boot ━━━━━━
// Don't immediately rAF at top level — wait for fonts to load before starting
document.fonts.ready.then(() => {
  render(0);                 // first draw the initial frame (fonts are ready)
  playing = true;
  requestAnimationFrame(tick);  // first tick will pair __ready + t=0
});

// ━━━━━━ seek interface (for render-video defensive correction) ━━━━━━
window.__seek = (t) => { fired.clear(); time = t; lastTick = null; render(t); };
```

**Why this template is correct**:

| Step | Why it must be this way |
|------|-------------|
| `lastTick = null` + first frame `return` | Avoids L seconds from "script load to tick first execution" being counted into animation time |
| `playing = false` default | During font loading, even if `tick` runs, time doesn't advance — avoids rendering offset |
| `__ready` set in tick's first frame | Recording script starts timing from this moment, corresponding frame is animation's true t=0 |
| Only start tick inside `document.fonts.ready.then(...)` | Avoids fallback font width measurements, avoids first-frame font jump |
| `window.__seek` exists | Allows `render-video.js` to actively correct — second line of defense |

**Corresponding defenses on the recording script side**:
1. `addInitScript` injects `window.__recording = true` (before page goto)
2. `waitForFunction(() => window.__ready === true)`, record this moment's offset as ffmpeg trim
3. **Additionally**: After `__ready`, actively `page.evaluate(() => window.__seek && window.__seek(0))`, forcibly zeroing any time offset the HTML may have — second line of defense, for dealing with HTML that doesn't strictly follow the starter template

**Verification method**: After exporting MP4
```bash
ffmpeg -i video.mp4 -ss 0 -vframes 1 frame-0.png
ffmpeg -i video.mp4 -ss $DURATION-0.1 -vframes 1 frame-end.png
```
First frame must be animation t=0 initial state (not mid-animation, not black); last frame must be animation final state (not some moment in the second loop).

**Reference implementation**: The Stage component in `assets/animations.jsx` and `scripts/render-video.js` are both implemented per this protocol. Hand-written HTML must use the starter tick template — every line guards against a specific bug.

## 13. Disable Loop During Recording — `window.__recording` Signal

**The trap**: Animation Stage defaults to `loop=true` (convenient for browser viewing). `render-video.js` waits an extra 300ms buffer after recording duration before stopping. During this 300ms, Stage enters the next loop cycle. ffmpeg `-t DURATION` clips at that point, and the last 0.5-1s falls into the next loop — the video ending suddenly jumps back to the first frame (Scene 1), audience thinks the video has a bug.

**Root cause**: No "I'm recording" handshake protocol between the recording script and HTML. HTML doesn't know it's being recorded and continues to loop.

**Rules**:

1. **Recording script**: Inject `window.__recording = true` in `addInitScript` (before page goto):
   ```js
   await recordCtx.addInitScript(() => { window.__recording = true; });
   ```

2. **Stage component**: Recognize this signal, force loop=false:
   ```js
   const effectiveLoop = (typeof window !== 'undefined' && window.__recording) ? false : loop;
   // ...
   if (next >= duration) return effectiveLoop ? 0 : duration - 0.001;
   //                                                       ↑ keep 0.001 to prevent Sprite end=duration from being closed
   ```

3. **Ending Sprite's fadeOut**: Should be set to `fadeOut={0}` in recording mode, otherwise the video end will fade to transparent/dark. For hand-written HTML, it's recommended that ending Sprites always use `fadeOut={0}`.

**Reference implementation**: Stage in `assets/animations.jsx` and `scripts/render-video.js` both have the handshake built in. Hand-written Stage must implement `__recording` detection — otherwise recording will always hit this trap.

**Verification**: After exporting MP4, `ffmpeg -ss 19.8 -i video.mp4 -frames:v 1 end.png`, check whether the last 0.2 seconds is still the expected final frame, without suddenly cutting to another scene.

## 14. 60fps Video Defaults to Frame Duplication — minterpolate Has Poor Compatibility

**The trap**: `convert-formats.sh` using `minterpolate=fps=60:mi_mode=mci...` generates 60fps MP4 that cannot be opened in macOS QuickTime / some Safari versions (black screen or outright rejected). VLC / Chrome can open it.

**Root cause**: H.264 elementary stream output by minterpolate contains SEI / SPS fields that some players have trouble parsing.

**Rules**:

- Default 60fps uses simple `fps=60` filter (frame duplication) — wide compatibility (QuickTime/Safari/Chrome/VLC all work)
- High-quality interpolation uses `--minterpolate` flag explicitly — but **must be tested on target players** before delivery
- The value of the 60fps label is **upload platform algorithm recognition** (Bilibili / YouTube mark 60fps for priority streaming); actual perceived smoothness improvement for CSS animation is minimal
- Add `-profile:v high -level 4.0` to improve H.264 general compatibility

**`convert-formats.sh` has already defaulted to compatibility mode**. If you need high-quality frame interpolation, add `--minterpolate` flag:
```bash
bash convert-formats.sh input.mp4 --minterpolate
```

## 15. `file://` + External `.jsx` CORS Trap — Single-file Delivery Must Inline the Engine

**The trap**: Animation HTML uses `<script type="text/babel" src="animations.jsx"></script>` to externally load the engine. Opening locally (via `file://` protocol) → Babel Standalone makes XHR request for `.jsx` → Chrome reports `Cross origin requests are only supported for protocol schemes: http, https, chrome, chrome-extension...` → entire page black screen. This doesn't trigger `pageerror`, only a console error — easy to misdiagnose as "animation not triggering."

Starting an HTTP server also may not help — when the machine has a global proxy, `localhost` also goes through the proxy, returning 502 / connection failed.

**Rules**:

- **Single-file delivery (HTML that works by double-clicking)** → `animations.jsx` must be **inlined** into `<script type="text/babel">...</script>` tag, do not use `src="animations.jsx"`
- **Multi-file projects (demonstrated with HTTP server)** → external loading is fine, but explicitly document `python3 -m http.server 8000` command upon delivery
- Decision criterion: Is what's being delivered to users "an HTML file" or "a project directory with a server"? Former uses inline
- Stage component / animations.jsx is often 200+ lines — pasting into HTML `<script>` block is completely acceptable; don't worry about file size

**Minimum validation**: Double-click the HTML you generated, **do not** open through any server. If Stage displays the animation first frame correctly, it passes.

## 16. Cross-scene Inverted Color Context — Don't Hardcode Colors for On-screen Elements

**The trap**: In multi-scene animations, elements that **appear across all scenes** like `ChapterLabel` / `SceneNumber` / `Watermark` have `color: '#1A1A1A'` (dark text) hardcoded in the component. The first 4 scenes with light backgrounds are fine, but the 5th black-background scene makes "05" and the watermark completely disappear — no errors, no checks triggered, critical information invisible.

**Rules**:

- **On-screen elements reused across multiple scenes** (chapter labels / scene numbers / timecodes / watermarks / copyright bars) **must not have hardcoded color values**
- Change to one of three approaches:
  1. **`currentColor` inheritance**: Element only writes `color: currentColor`, parent scene container sets `color: computed value`
  2. **invert prop**: Component accepts `<ChapterLabel invert />` for manual light/dark switching
  3. **Auto-calculate based on background**: `color: contrast-color(var(--scene-bg))` (CSS 4 new API, or JS judgment)
- Before delivery, use Playwright to extract **representative frames for each scene** and visually verify "cross-scene elements" are all visible

The insidiousness of this trap is — **there is no bug alert**. Only the human eye or OCR can detect it.

## Quick Self-check Checklist (5 seconds before starting)

- [ ] Every `position: absolute` parent element has `position: relative`?
- [ ] Special characters in animation (`␣` `⌘` `emoji`) all exist in the font?
- [ ] Grid/Flex template count matches JS data length?
- [ ] Scene transitions have cross-fade, no >0.3s pure blank?
- [ ] DOM measurement code is wrapped in `document.fonts.ready.then()`?
- [ ] `render(t)` is pure, or has a clear reset mechanism?
- [ ] Frame 0 is complete initial state, not blank?
- [ ] No "fake chrome" decorations on canvas (progress bar/timecode/bottom attribution bar conflicting with Stage scrubber)?
- [ ] Animation tick first frame synchronously sets `window.__ready = true`? (use animations.jsx built-in; hand-written HTML adds it manually)
- [ ] Stage detects `window.__recording` and forces loop=false? (hand-written HTML must add)
- [ ] Ending Sprite's `fadeOut` set to 0 (video ends on clear frame)?
- [ ] 60fps MP4 defaults to frame duplication mode (compatibility); add `--minterpolate` only for high-quality interpolation?
- [ ] After export, extract frame 0 + last frame to verify they are animation initial/final states?
- [ ] Involves specific brands (Stripe/Anthropic/Lovart/...): completed the "brand asset protocol" (SKILL.md §1.a five steps)? Is there a `brand-spec.md`?
- [ ] Single-file delivery HTML: `animations.jsx` is inlined, not `src="..."`? (External .jsx under file:// causes CORS black screen)
- [ ] Cross-scene elements (chapter labels/watermarks/scene numbers) don't have hardcoded colors? Visible against every scene's background?
