# Audio Design Rules · huashu-design

> Audio application recipes for all animation demos. Used alongside `sfx-library.md` (asset inventory).
> Battle-tested through: huashu-design hero v1–v9 iterations · Deep Gemini analysis of 3 official Anthropic videos · 8000+ A/B comparisons

---

## Core Principle · Dual-Track Audio (Non-Negotiable)

Animation audio **must be designed in two independent layers** — doing only one layer is not enough:

| Layer | Role | Time Scale | Relationship to Visuals | Frequency Range |
|---|---|---|---|---|
| **SFX (beat layer)** | Marks each visual beat | Short, 0.2–2 seconds | **Tight sync** (frame-level alignment) | **High-freq 800Hz+** |
| **BGM (ambient base)** | Sets emotional tone, fills the soundscape | Continuous, 20–60 seconds | Loose sync (section-level) | **Mid-low freq <4kHz** |

**An animation with only BGM is broken** — the audience subconsciously notices that "things are moving but nothing is responding to the sound," and this is the root cause of the cheap feel.

---

## Gold Standard · Golden Ratios

These values were derived from empirical testing of 3 official Anthropic videos combined with our own v9 final cut comparisons. They are **hardened engineering parameters** — apply them directly:

### Volume
- **BGM volume**: `0.40–0.50` (relative to full scale 1.0)
- **SFX volume**: `1.00`
- **Loudness difference**: BGM peak is **−6 to −8 dB below SFX** (prominence comes from the loudness gap, not the absolute SFX volume)
- **amix parameter**: `normalize=0` (never use normalize=1 — it flattens dynamic range)

### Frequency Separation (P1 Hard Optimization)
Anthropic's secret is not "louder SFX" — it's **frequency layering**:

```bash
[bgm_raw]lowpass=f=4000[bgm]      # BGM capped at mid-low freq <4kHz
[sfx_raw]highpass=f=800[sfx]      # SFX pushed into mid-high freq 800Hz+
[bgm][sfx]amix=inputs=2:duration=first:normalize=0[a]
```

Why: The human ear is most sensitive in the 2–5kHz range (the "presence band"). If SFX all sit in this range while BGM covers the full spectrum, **SFX gets masked by BGM's high-frequency content**. Using highpass to push SFX up + lowpass to pull BGM down gives each its own territory on the spectrum — SFX clarity jumps a full grade.

### Fade
- BGM in: `afade=in:st=0:d=0.3` (0.3s, avoids a hard cut)
- BGM out: `afade=out:st=N-1.5:d=1.5` (1.5s long tail, sense of resolution)
- SFX have their own built-in envelope — no additional fade needed

---

## SFX Cue Design Rules

### Density (SFX count per 10 seconds)
Empirical testing of 3 Anthropic videos yields three density tiers:

| Video | SFX per 10s | Product Personality | Scenario |
|---|---|---|---|
| Artifacts (ref-1) | **~9 per 10s** | Feature-dense, information-rich | Complex tool demo |
| Code Desktop (ref-2) | **0** | Pure atmosphere, meditative | Developer focus state |
| Word (ref-3) | **~4 per 10s** | Balanced, office rhythm | Productivity tool |

**Heuristics**:
- Calm/focused product personality → Low SFX density (0–3 per 10s), BGM-led
- Lively/information-dense product personality → High SFX density (6–9 per 10s), SFX drives the rhythm
- **Don't fill every visual beat** — negative space is more sophisticated than density. **Removing 30–50% of cues makes the remaining ones more dramatic**.

### Cue Selection Priority
Not every visual beat needs a SFX. Use this priority order:

**P0 — Always cue** (omitting creates a jarring absence):
- Typing (terminal / input field)
- Click / selection (user decision moments)
- Focus change (visual protagonist shifts)
- Logo reveal (brand resolution)

**P1 — Recommended**:
- Element enter / exit (modal / card)
- Completion / success feedback
- AI generation start / end
- Major transition (scene cut)

**P2 — Optional** (too many gets noisy):
- hover / focus-in
- Progress tick
- Decorative ambient

### Timestamp Alignment Precision
- **Same-frame alignment** (0ms error): click / focus change / logo landing
- **1–2 frames early** (−33ms): fast whoosh (gives the audience psychological anticipation)
- **1–2 frames late** (+33ms): object landing / impact (matches real-world physics)

---

## BGM Selection Decision Tree

huashu-design skill ships with 6 BGM tracks (`assets/bgm-*.mp3`):

```
What is the animation's personality?
├─ Product launch / technical demo → bgm-tech.mp3 (minimal synth + piano)
├─ Tutorial / tool walkthrough → bgm-tutorial.mp3 (warm, instructional)
├─ Educational / concept explanation → bgm-educational.mp3 (curious, thoughtful)
├─ Marketing / brand promotion → bgm-ad.mp3 (upbeat, promotional)
└─ Need a variation of the same style → bgm-*-alt.mp3 (alternate versions)
```

### When No BGM Is the Right Call
Reference Anthropic Code Desktop (ref-2): **0 SFX + pure Lo-fi BGM** can still feel premium.

**When to choose no BGM**:
- Animation is <10s (BGM never establishes itself)
- Product personality is "focused / meditative"
- The scene already has ambient sound or voiceover
- SFX density is high (avoid auditory overload)

---

## Scene Recipes (Ready to Use)

### Recipe A · Product Launch Hero (huashu-design v9 Formula)
```
Duration: 25 seconds
BGM: bgm-tech.mp3 · 45% · freq <4kHz
SFX density: ~6 per 10s

Cues:
  Terminal typing → type × 4 (0.6s intervals)
  Enter key       → enter
  Cards converge  → card × 4 (staggered 0.2s)
  Selection       → click
  Ripple          → whoosh
  4× focus        → focus × 4
  Logo            → thud (1.5s)

Volume: BGM 0.45 / SFX 1.0 · amix normalize=0
```

### Recipe B · Tool Feature Demo (Reference: Anthropic Code Desktop)
```
Duration: 30–45 seconds
BGM: bgm-tutorial.mp3 · 50%
SFX density: 0–2 per 10s (minimal)

Strategy: Let BGM + voiceover drive; SFX only at decisive moments (file save / command execution complete)
```

### Recipe C · AI Generation Demo
```
Duration: 15–20 seconds
BGM: bgm-tech.mp3 or no BGM
SFX density: ~8 per 10s (high density)

Cues:
  User input         → type + enter
  AI starts processing → magic/ai-process (1.2s loop)
  Generation complete → feedback/complete-done
  Result appears     → magic/sparkle

Highlight: ai-process can loop 2–3 times throughout the entire generation sequence
```

### Recipe D · Pure Atmosphere Long Take (Reference: Artifacts)
```
Duration: 10–15 seconds
BGM: none
SFX: 3–5 carefully chosen standalone cues

Strategy: Each SFX is the protagonist — no BGM "smearing" everything together.
Best for: Single-product slow shots, close-up showcases
```

---

## ffmpeg Synthesis Templates

### Template 1 · Single SFX Layered onto Video
```bash
ffmpeg -y -i video.mp4 -itsoffset 2.5 -i sfx.mp3 \
  -filter_complex "[0:a][1:a]amix=inputs=2:normalize=0[a]" \
  -map 0:v -map "[a]" output.mp4
```

### Template 2 · Multi-SFX Timeline Composition (aligned to cue times)
```bash
ffmpeg -y \
  -i sfx-type.mp3 -i sfx-enter.mp3 -i sfx-click.mp3 -i sfx-thud.mp3 \
  -filter_complex "\
[0:a]adelay=1100|1100[a0];\
[1:a]adelay=3200|3200[a1];\
[2:a]adelay=7000|7000[a2];\
[3:a]adelay=21800|21800[a3];\
[a0][a1][a2][a3]amix=inputs=4:duration=longest:normalize=0[mixed]" \
  -map "[mixed]" -t 25 sfx-track.mp3
```
**Key parameters**:
- `adelay=N|N`: First value is left-channel delay (ms), second is right — write both to ensure stereo alignment
- `normalize=0`: Preserves dynamic range — critical!
- `-t 25`: Truncates to the specified duration

### Template 3 · Video + SFX Track + BGM (with frequency separation)
```bash
ffmpeg -y -i video.mp4 -i sfx-track.mp3 -i bgm.mp3 \
  -filter_complex "\
[2:a]atrim=0:25,afade=in:st=0:d=0.3,afade=out:st=23.5:d=1.5,\
     lowpass=f=4000,volume=0.45[bgm];\
[1:a]highpass=f=800,volume=1.0[sfx];\
[bgm][sfx]amix=inputs=2:duration=first:normalize=0[a]" \
  -map 0:v -map "[a]" -c:v copy -c:a aac -b:a 192k final.mp4
```

---

## Failure Mode Quick Reference

| Symptom | Root Cause | Fix |
|---|---|---|
| SFX inaudible | BGM high-freq content masking it | Add `lowpass=f=4000` to BGM + `highpass=f=800` to SFX |
| SFX painfully loud | SFX absolute volume too high | Drop SFX to 0.7 and BGM to 0.3, preserve the gap |
| BGM and SFX rhythm conflict | Wrong BGM choice (used music with a strong beat) | Switch to an ambient / minimal synth BGM |
| BGM cuts off abruptly at end | No fade out applied | `afade=out:st=N-1.5:d=1.5` |
| SFX blur together | Too many cues + each SFX too long | Keep SFX under 0.5s, cue intervals ≥ 0.2s |
| WeChat mp4 has no audio | WeChat sometimes mutes autoplay | Don't worry — users hear it when they tap; GIFs have no audio by nature |

---

## Syncing with Visuals (Advanced)

### SFX Timbre Must Match the Visual Style
- Warm beige / paper-texture visuals → SFX in **wood / soft** timbres (Morse, paper snap, soft click)
- Cold dark-tech visuals → SFX in **metal / digital** timbres (beep, pulse, glitch)
- Hand-drawn / playful visuals → SFX in **cartoon / exaggerated** timbres (boing, pop, zap)

Our current `apple-gallery-showcase.md` warm beige palette → pairs with `keyboard/type.mp3` (mechanical) + `container/card-snap.mp3` (soft) + `impact/logo-reveal-v2.mp3` (cinematic bass)

### SFX Can Lead the Visual Rhythm
Advanced technique: **Design the SFX timeline first, then adjust the visual animation to align to the SFX** (not the other way around).
Because each SFX cue is a "clock tick," visual animation adapting to the SFX rhythm stays rock-solid — the reverse (SFX chasing visuals) often results in ±1 frame misalignment that feels off.

---

## Quality Checklist (Pre-Publish Self-Review)

- [ ] Loudness gap: SFX peak − BGM peak = −6 to −8 dB?
- [ ] Frequency: BGM lowpass 4kHz + SFX highpass 800Hz?
- [ ] amix normalize=0 (preserves dynamic range)?
- [ ] BGM fade-in 0.3s + fade-out 1.5s?
- [ ] SFX count appropriate for the scene personality?
- [ ] Each SFX frame-aligned with its visual beat (within ±1 frame)?
- [ ] Logo reveal SFX long enough (recommended 1.5s)?
- [ ] Mute BGM and listen: does the SFX track alone have sufficient rhythmic feel?
- [ ] Mute SFX and listen: does BGM alone carry emotional shape?

Each layer should hold up on its own. If it only sounds good when both are combined, the design isn't done.

---

## References

- SFX asset inventory: `sfx-library.md`
- Visual style reference: `apple-gallery-showcase.md`
- Deep audio analysis of 3 Anthropic videos: `/Users/alchain/Documents/写作/01-公众号写作/项目/2026.04-huashu-design发布/参考动画/AUDIO-BEST-PRACTICES.md`
- huashu-design v9 production case: `/Users/alchain/Documents/写作/01-公众号写作/项目/2026.04-huashu-design发布/配图/hero-animation-v9-final.mp4`
