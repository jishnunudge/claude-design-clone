# Verification: Output Validation Process

Some native design-agent environments (such as Claude.ai Artifacts) have a built-in `fork_verifier_agent` that spawns a subagent to check screenshots via iframe. Most agent environments (Claude Code / Codex / Cursor / Trae / etc.) don't have this built-in capability — using Playwright manually covers the same verification scenarios.

## Verification Checklist

After each HTML output, run through this checklist:

### 1. Browser Render Check (Required)

The most basic check: **can the HTML be opened?** On macOS:

```bash
open -a "Google Chrome" "/path/to/your/design.html"
```

Or use Playwright for a screenshot (next section).

### 2. Console Error Check

The most common issue in HTML files is a white screen caused by JS errors. Run through it with Playwright:

```bash
python ~/.claude/skills/claude-design/scripts/verify.py path/to/design.html
```

This script will:
1. Open the HTML in headless Chromium
2. Save a screenshot to the project directory
3. Capture console errors
4. Report status

See `scripts/verify.py` for details.

### 3. Multi-Viewport Check

For responsive designs, capture multiple viewports:

```bash
python verify.py design.html --viewports 1920x1080,1440x900,768x1024,375x667
```

### 4. Interaction Check

Tweaks, animations, button toggles — static screenshots won't reveal these. **It's recommended to have the user open it in their browser and click through**, or use Playwright to record:

```python
page.video.record('interaction.mp4')
```

### 5. Slide-by-Slide Check

For deck-type HTML, screenshot each slide:

```bash
python verify.py deck.html --slides 10  # screenshot the first 10 slides
```

Generates `deck-slide-01.png`, `deck-slide-02.png`... for quick review.

## Playwright Setup

Required on first use:

```bash
# If not already installed
npm install -g playwright
npx playwright install chromium

# Or the Python version
pip install playwright
playwright install chromium
```

If the user already has Playwright installed globally, use it directly.

## Screenshot Best Practices

### Full-page screenshot

```python
page.screenshot(path='full.png', full_page=True)
```

### Viewport screenshot

```python
page.screenshot(path='viewport.png')  # defaults to visible area only
```

### Screenshot a specific element

```python
element = page.query_selector('.hero-section')
element.screenshot(path='hero.png')
```

### High-DPI screenshot

```python
page = browser.new_page(device_scale_factor=2)  # retina
```

### Wait for animations to finish before screenshotting

```python
page.wait_for_timeout(2000)  # wait 2s for animations to settle
page.screenshot(...)
```

## Sharing Screenshots with the User

### Open a local screenshot directly

```bash
open screenshot.png
```

The user will view it in their own Preview / Figma / VSCode / browser.

### Upload to an image host for a shareable link

If remote collaborators need to view it (e.g. Slack / Lark / WeChat), have the user use their own image hosting tool or MCP to upload:

```bash
python ~/Documents/writing/tools/upload_image.py screenshot.png
```

Returns a permanent ImgBB link that can be pasted anywhere.

## When Verification Fails

### White screen

There will always be a console error. Check:

1. Whether the integrity hash on the React + Babel script tag is correct (see `react-setup.md`)
2. Whether there's a naming conflict like `const styles = {...}`
3. Whether cross-file components are exported to `window`
4. JSX syntax errors (babel.min.js won't report errors — switch to unminified babel.js)

### Janky animation

- Record a segment in Chrome DevTools Performance tab
- Look for layout thrashing (frequent reflows)
- Prefer `transform` and `opacity` for animations (GPU-accelerated)

### Wrong font

- Check whether the `@font-face` URL is accessible
- Check fallback fonts
- CJK fonts load slowly: show the fallback first, then switch once loaded

### Layout misalignment

- Check whether `box-sizing: border-box` is applied globally
- Check `* { margin: 0; padding: 0 }` reset
- Open gridlines in Chrome DevTools to see the actual layout

## Verification = The Designer's Second Set of Eyes

**Always review it yourself.** When AI writes code, common issues include:

- Looks correct but has interaction bugs
- Static screenshot looks fine but misaligns on scroll
- Looks great on wide screens but breaks on narrow ones
- Forgot to test dark mode
- Some components don't respond after Tweaks are toggled

**1 minute of verification can save 1 hour of rework.**

## Common Verification Script Commands

```bash
# Basic: open + screenshot + catch errors
python verify.py design.html

# Multiple viewports
python verify.py design.html --viewports 1920x1080,375x667

# Multiple slides
python verify.py deck.html --slides 10

# Output to a specific directory
python verify.py design.html --output ./screenshots/

# headless=false, open in a real browser for you to see
python verify.py design.html --show
```
