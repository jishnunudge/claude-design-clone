# Video Export: Exporting HTML Animations to MP4/GIF

## When to Export

**Export when**:
- Animation runs completely and has been visually verified (Playwright screenshots confirm correct state at each time point)
- User has watched it in browser at least once and confirmed the effect looks good
- **Do not** export while animation bugs are unresolved — changes after video export are costly

**Trigger phrases**:
- "Can this be exported as a video?" / "Convert to MP4" / "Make it a GIF" / "60fps"

## Output Formats

Deliver three formats at once and let the user choose:

| Format | Spec | Best for | Typical size (30s) |
|---|---|---|---|
| MP4 25fps | 1920×1080 · H.264 · CRF 18 | WeChat Official Accounts, Video channels, YouTube | 1–2 MB |
| MP4 60fps | 1920×1080 · minterpolate frame interpolation · H.264 · CRF 18 | High frame-rate showcase, Bilibili, portfolios | 1.5–3 MB |
| GIF | 960×540 · 15fps · palette optimized | Twitter/X, README, Slack previews | 2–4 MB |

## Toolchain

Two scripts in `scripts/`:

### 1. `render-video.js` — HTML → MP4

Records 25fps MP4 base version. Requires global playwright.

```bash
NODE_PATH=$(npm root -g) node /path/to/claude-design/scripts/render-video.js <html-file>
```

Optional parameters:
- `--duration=30` animation duration (seconds)
- `--width=1920 --height=1080` resolution
- `--trim=2.2` seconds to trim from start (removes reload + font load time)
- `--fontwait=1.5` font loading wait (seconds), increase for many fonts

Output: same directory as HTML, same name with `.mp4` extension.

### 2. `add-music.sh` — MP4 + BGM → MP4

Mixes background music into a silent MP4, selecting from the built-in BGM library by mood or a custom audio file. Auto-matches duration with fade in/out.

```bash
bash add-music.sh <input.mp4> [--mood=<name>] [--music=<path>] [--out=<path>]
```

**Built-in BGM library** (in `assets/bgm-<mood>.mp3`):

| `--mood=` | Style | Best for |
|-----------|-------|---------|
| `tech` (default) | Apple keynote style, minimal synth + piano | Product launches, AI tools, skill promos |
| `ad` | Upbeat modern electronic, build + drop | Social media ads, product teasers, promos |
| `educational` | Warm, light guitar / electric piano | Science content, tutorials, course previews |
| `educational-alt` | Same category, alternate track | Same as above |
| `tutorial` | Lo-fi ambient, nearly unobtrusive | Software demos, coding tutorials, long demos |
| `tutorial-alt` | Same category, alternate track | Same as above |

**Behavior**:
- Music trimmed to match video duration
- 0.3s fade in + 1s fade out
- Video stream `-c:v copy` (no re-encoding), audio AAC 192k
- `--music=<path>` takes priority over `--mood`
- Wrong mood name lists available options — won't fail silently

**Typical pipeline**:
```bash
node render-video.js animation.html                        # record screen
bash convert-formats.sh animation.mp4                      # derive 60fps + GIF
bash add-music.sh animation-60fps.mp4                      # add default tech BGM
# Or for different scenarios:
bash add-music.sh tutorial-demo.mp4 --mood=tutorial
bash add-music.sh product-promo.mp4 --mood=ad --out=promo-final.mp4
```

### 3. `convert-formats.sh` — MP4 → 60fps MP4 + GIF

```bash
bash /path/to/claude-design/scripts/convert-formats.sh <input.mp4> [gif_width] [--minterpolate]
```

Output (same directory as input):
- `<name>-60fps.mp4` — default uses `fps=60` frame duplication (broad compat); add `--minterpolate` for high-quality interpolation
- `<name>.gif` — palette-optimized GIF (default 960px wide, adjustable)

**60fps Mode Selection**:

| Mode | Command | Compatibility | Use case |
|---|---|---|---|
| Frame duplication (default) | `convert-formats.sh in.mp4` | QuickTime/Safari/Chrome/VLC all work | General delivery, platform uploads, social media |
| minterpolate interpolation | `convert-formats.sh in.mp4 --minterpolate` | macOS QuickTime/Safari may reject | Bilibili and venues requiring true interpolation — **test locally before delivery** |

Default is frame duplication because minterpolate's H.264 elementary stream has a known compat bug causing "macOS QuickTime can't open it" failures. See `animation-pitfalls.md` §14.

`gif_width` parameter: 960 (default, general use) / 1280 (sharper, larger) / 600 (optimized for Twitter/X)

## Full Process (Standard Recommendation)

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

- Frame rate fixed at 25fps; 60fps can't be recorded directly (Chromium headless compositor limit)
- Recording starts from context creation; use `trim` to cut loading time at start
- Default format is webm; needs ffmpeg to convert to H.264 MP4

`render-video.js` handles all of the above.

### ffmpeg minterpolate Parameters

Current config: `minterpolate=fps=60:mi_mode=mci:mc_mode=aobmc:me_mode=bidir:vsbmc=1`

- `mi_mode=mci` — motion compensation interpolation
- `mc_mode=aobmc` — adaptive overlapped block motion compensation
- `me_mode=bidir` — bidirectional motion estimation
- `vsbmc=1` — variable size block motion compensation

Works well for CSS **transform animations** (translate/scale/rotate). May produce slight ghosting on **pure fades** — fall back to simple frame duplication if user objects:

```bash
ffmpeg -i input.mp4 -r 60 -c:v libx264 ... output.mp4
```

### Why GIF Palette Requires Two Passes

GIF is limited to 256 colors. Single-pass compresses all colors into a generic 256-color palette, causing blurring for subtle schemes like beige + orange.

Two passes:
1. `palettegen=stats_mode=diff` — scans entire clip to generate an optimal palette for this specific animation
2. `paletteuse=dither=bayer:bayer_scale=5:diff_mode=rectangle` — encodes with this palette; rectangle diff only updates changed areas, reducing file size

`dither=bayer` is smoother than `none` for fade transitions but slightly larger file.

## Pre-flight Check (Before Export)

- [ ] HTML fully run in browser with no console errors
- [ ] Frame 0 is the complete initial state (not blank loading screen)
- [ ] Final frame is a stable end state (not cut off mid-way)
- [ ] Fonts / images / emoji all render correctly (see `animation-pitfalls.md`)
- [ ] Duration parameter matches actual animation duration in HTML
- [ ] `window.__recording` in HTML forces loop=false (required for hand-written Stages; built-in with `assets/animations.jsx`)
- [ ] Ending Sprite has `fadeOut={0}` (video final frame should not fade out)
- [ ] Includes "Created by Huashu-Design" watermark (animation scenarios only; prefix "Unofficial · " for third-party brand work. See SKILL.md § "Skill Promotion Watermark")

## Standard Delivery Note Format

```
**Complete Delivery**

| File | Format | Spec | Size |
|---|---|---|---|
| foo.mp4 | MP4 | 1920×1080 · 25fps · H.264 | X MB |
| foo-60fps.mp4 | MP4 | 1920×1080 · 60fps (motion interpolation) · H.264 | X MB |
| foo.gif | GIF | 960×540 · 15fps · palette optimized | X MB |

**Notes**
- 60fps uses minterpolate for motion estimation interpolation — works great for transform animations
- GIF uses palette optimization; a 30s animation compresses to ~3MB

Let me know if you need a different size or frame rate.
```

## Common Follow-up User Requests

| User says | How to handle |
|---|---|
| "Too large" | MP4: increase CRF to 23–28; GIF: reduce resolution to 600 or fps to 10 |
| "GIF is too blurry" | Increase `gif_width` to 1280; or suggest MP4 instead |
| "I need vertical 9:16" | Change HTML source with `--width=1080 --height=1920` and re-record |
| "Add a watermark" | ffmpeg: add `-vf "drawtext=..."` or `overlay=` a PNG |
| "I need transparent background" | MP4 doesn't support alpha; use WebM VP9 + alpha or APNG |
| "I need lossless" | Set CRF to 0 + preset veryslow (file ~10x larger) |
