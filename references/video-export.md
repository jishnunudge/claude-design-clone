# Video Export: Exporting HTML Animations to MP4/GIF

After an animated HTML is complete, users often ask "can I export this as a video?" This guide covers the full process.

## When to Export

**Export timing**:
- The animation runs completely and has been visually verified (Playwright screenshots confirm correct state at each time point)
- The user has watched it in a browser at least once and confirmed the effect looks good
- **Do not** export while animation bugs are still unresolved — making changes after export to video is more costly

**Trigger phrases the user might say**:
- "Can this be exported as a video?"
- "Convert to MP4"
- "Make it a GIF"
- "60fps"

## Output Formats

By default, deliver three formats at once and let the user choose:

| Format | Spec | Best for | Typical size (30s) |
|---|---|---|---|
| MP4 25fps | 1920×1080 · H.264 · CRF 18 | WeChat Official Accounts, Video channels, YouTube | 1–2 MB |
| MP4 60fps | 1920×1080 · minterpolate frame interpolation · H.264 · CRF 18 | High frame-rate showcase, Bilibili, portfolios | 1.5–3 MB |
| GIF | 960×540 · 15fps · palette optimized | Twitter/X, README, Slack previews | 2–4 MB |

## Toolchain

Two scripts in `scripts/`:

### 1. `render-video.js` — HTML → MP4

Records a 25fps MP4 base version. Requires global playwright.

```bash
NODE_PATH=$(npm root -g) node /path/to/claude-design/scripts/render-video.js <html-file>
```

Optional parameters:
- `--duration=30` animation duration (seconds)
- `--width=1920 --height=1080` resolution
- `--trim=2.2` seconds to trim from the start (removes reload + font load time)
- `--fontwait=1.5` font loading wait time (seconds), increase when using many fonts

Output: same directory as the HTML, same name with `.mp4` extension.

### 2. `add-music.sh` — MP4 + BGM → MP4

Mixes background music into a silent MP4, selecting from the built-in BGM library by mood, or you can provide your own audio. Auto-matches duration with fade in/out.

```bash
bash add-music.sh <input.mp4> [--mood=<name>] [--music=<path>] [--out=<path>]
```

**Built-in BGM library** (in `assets/bgm-<mood>.mp3`):

| `--mood=` | Style | Best for |
|-----------|-------|---------|
| `tech` (default) | Apple Silicon / Apple keynote style, minimal synth + piano | Product launches, AI tools, skill promos |
| `ad` | Upbeat modern electronic, with build + drop | Social media ads, product teasers, promos |
| `educational` | Warm and bright, light guitar / electric piano, inviting | Science content, tutorials, course previews |
| `educational-alt` | Same category, alternate track | Same as above |
| `tutorial` | Lo-fi ambient, nearly unobtrusive | Software demos, coding tutorials, long demos |
| `tutorial-alt` | Same category, alternate track | Same as above |

**Behavior**:
- Music is trimmed to match the video duration
- 0.3s fade in + 1s fade out (avoids hard cuts)
- Video stream `-c:v copy` without re-encoding, audio AAC 192k
- `--music=<path>` takes priority over `--mood`, you can specify any external audio directly
- Passing a wrong mood name will list all available options — it won't fail silently

**Typical pipeline** (animation export trio + music):
```bash
node render-video.js animation.html                        # record screen
bash convert-formats.sh animation.mp4                      # derive 60fps + GIF
bash add-music.sh animation-60fps.mp4                      # add default tech BGM
# Or for different scenarios:
bash add-music.sh tutorial-demo.mp4 --mood=tutorial
bash add-music.sh product-promo.mp4 --mood=ad --out=promo-final.mp4
```

### 3. `convert-formats.sh` — MP4 → 60fps MP4 + GIF

Generates a 60fps version and a GIF from an existing MP4.

```bash
bash /path/to/claude-design/scripts/convert-formats.sh <input.mp4> [gif_width] [--minterpolate]
```

Output (same directory as input):
- `<name>-60fps.mp4` — by default uses `fps=60` frame duplication (broad compatibility); add `--minterpolate` for high-quality frame interpolation
- `<name>.gif` — palette-optimized GIF (default 960px wide, adjustable)

**60fps Mode Selection**:

| Mode | Command | Compatibility | Use case |
|---|---|---|---|
| Frame duplication (default) | `convert-formats.sh in.mp4` | QuickTime/Safari/Chrome/VLC all work | General delivery, platform uploads, social media |
| minterpolate interpolation | `convert-formats.sh in.mp4 --minterpolate` | macOS QuickTime/Safari may reject | Bilibili and other venues requiring true interpolation — **must test locally with target player before delivery** |

Why was the default changed to frame duplication? minterpolate's H.264 elementary stream output has a known compat bug — when minterpolate was the default, we repeatedly hit "macOS QuickTime can't open it" issues. See `animation-pitfalls.md` §14.

`gif_width` parameter:
- 960 (default) — general social platform use
- 1280 — sharper but larger file
- 600 — optimized for Twitter/X loading

## Full Process (Standard Recommendation)

After the user says "export video":

```bash
cd <project directory>

# Assuming $SKILL points to the root of this skill (adjust per install location)

# 1. Record 25fps base MP4
NODE_PATH=$(npm root -g) node "$SKILL/scripts/render-video.js" my-animation.html

# 2. Derive 60fps MP4 and GIF
bash "$SKILL/scripts/convert-formats.sh" my-animation.mp4

# Output:
# my-animation.mp4         (25fps · 1-2 MB)
# my-animation-60fps.mp4   (60fps · 1.5-3 MB)
# my-animation.gif         (15fps · 2-4 MB)
```

## Technical Details (For Debugging)

### Playwright recordVideo Caveats

- Frame rate is fixed at 25fps; 60fps cannot be recorded directly (Chromium headless compositor limit)
- Recording starts from context creation; must use `trim` to cut the loading time at the start
- Default format is webm; needs ffmpeg to convert to H.264 MP4 for universal playback

`render-video.js` already handles all of the above.

### ffmpeg minterpolate Parameters

Current configuration: `minterpolate=fps=60:mi_mode=mci:mc_mode=aobmc:me_mode=bidir:vsbmc=1`

- `mi_mode=mci` — motion compensation interpolation
- `mc_mode=aobmc` — adaptive overlapped block motion compensation
- `me_mode=bidir` — bidirectional motion estimation
- `vsbmc=1` — variable size block motion compensation

Works well for CSS **transform animations** (translate/scale/rotate).

May produce slight ghosting on **pure fades** — if the user objects, fall back to simple frame duplication:

```bash
ffmpeg -i input.mp4 -r 60 -c:v libx264 ... output.mp4
```

### Why GIF Palette Requires Two Passes

GIF can only use 256 colors. A single-pass GIF compresses the entire animation's colors into a 256-color generic palette, which results in blurring for subtle color schemes like beige + orange.

Two passes:
1. `palettegen=stats_mode=diff` — scans the entire clip first to generate an **optimal palette for this specific animation**
2. `paletteuse=dither=bayer:bayer_scale=5:diff_mode=rectangle` — encodes using this palette; rectangle diff only updates changed areas, significantly reducing file size

`dither=bayer` is smoother than `none` for fade transitions, but results in a slightly larger file.

## Pre-flight Check (Before Export)

30-second self-check before exporting:

- [ ] HTML has been fully run through in the browser with no console errors
- [ ] Frame 0 of the animation is the complete initial state (not a blank loading screen)
- [ ] The final frame of the animation is a stable end state (not cut off mid-way)
- [ ] Fonts / images / emoji all render correctly (see `animation-pitfalls.md`)
- [ ] Duration parameter matches the actual animation duration in the HTML
- [ ] Stage detection `window.__recording` in HTML forces loop=false (required check for hand-written Stages; built-in with `assets/animations.jsx`)
- [ ] Ending Sprite has `fadeOut={0}` (video final frame should not fade out)
- [ ] Includes "Created by Huashu-Design" watermark (required for animation scenarios only; prefix "Unofficial · " for third-party brand work. See SKILL.md § "Skill Promotion Watermark")

## Standard Delivery Note Format

Standard note format to give the user after export is complete:

```
**Complete Delivery**

| File | Format | Spec | Size |
|---|---|---|---|
| foo.mp4 | MP4 | 1920×1080 · 25fps · H.264 | X MB |
| foo-60fps.mp4 | MP4 | 1920×1080 · 60fps (motion interpolation) · H.264 | X MB |
| foo.gif | GIF | 960×540 · 15fps · palette optimized | X MB |

**Notes**
- 60fps uses minterpolate for motion estimation interpolation — works great for transform animations
- GIF uses palette optimization; a 30s animation can be compressed to around 3MB

Let me know if you need a different size or frame rate.
```

## Common Follow-up User Requests

| User says | How to handle |
|---|---|
| "Too large" | MP4: increase CRF to 23–28; GIF: reduce resolution to 600 or fps to 10 |
| "GIF is too blurry" | Increase `gif_width` to 1280; or suggest using MP4 instead (WeChat Moments also supports it) |
| "I need vertical 9:16" | Change the HTML source with `--width=1080 --height=1920` and re-record |
| "Add a watermark" | ffmpeg: add `-vf "drawtext=..."` or `overlay=` a PNG |
| "I need transparent background" | MP4 doesn't support alpha; use WebM VP9 + alpha or APNG |
| "I need lossless" | Set CRF to 0 + preset veryslow (file will be 10x larger) |
