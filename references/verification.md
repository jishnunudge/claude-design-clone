# Verification: Output Validation Process

Some native design-agent environments (e.g. Claude.ai Artifacts) have a built-in `fork_verifier_agent` that checks screenshots via iframe. Most environments (Claude Code / Codex / Cursor / Trae / etc.) don't — use Playwright manually to cover the same scenarios.

## Verification Checklist

After each HTML output, run through this checklist:

### 1. Browser Render Check (Required)

Can the HTML be opened? On macOS:

```bash
open -a "Google Chrome" "/path/to/your/design.html"
```

Or use Playwright for a screenshot (next section).

### 2. Console Error Check

Most common issue: white screen from JS errors. Run with Playwright:

```bash
python ~/.claude/skills/claude-design/scripts/verify.py path/to/design.html
```

This script:
1. Opens HTML in headless Chromium
2. Saves screenshot to the project directory
3. Captures console errors
4. Reports status

See `scripts/verify.py` for details.

### 3. Multi-Viewport Check

For responsive designs:

```bash
python verify.py design.html --viewports 1920x1080,1440x900,768x1024,375x667
```

### 4. Interaction Check

Tweaks, animations, button toggles — static screenshots won't catch these. **Have the user open it in their browser and click through**, or use Playwright to record:

```python
page.video.record('interaction.mp4')
```

### 5. Slide-by-Slide Check

For deck-type HTML:

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

### Wait for animations before screenshotting

```python
page.wait_for_timeout(2000)  # wait 2s for animations to settle
page.screenshot(...)
```

## Sharing Screenshots with the User

### Open locally

```bash
open screenshot.png
```

### Upload for shareable link

For remote collaborators (Slack / Lark / WeChat), use an image hosting tool or MCP:

```bash
python ~/Documents/writing/tools/upload_image.py screenshot.png
```

Returns a permanent ImgBB link.

## When Verification Fails

### White screen

Always a console error. Check:

1. Integrity hash on the React + Babel script tag (see `react-setup.md`)
2. Naming conflicts like `const styles = {...}`
3. Cross-file components exported to `window`
4. JSX syntax errors (babel.min.js won't report errors — switch to unminified babel.js)

### Janky animation

- Record in Chrome DevTools Performance tab
- Look for layout thrashing (frequent reflows)
- Prefer `transform` and `opacity` (GPU-accelerated)

### Wrong font

- Check whether `@font-face` URL is accessible
- Check fallback fonts
- CJK fonts load slowly: show fallback first, switch once loaded

### Layout misalignment

- Check `box-sizing: border-box` applied globally
- Check `* { margin: 0; padding: 0 }` reset
- Open gridlines in Chrome DevTools to inspect layout

## Verification = The Designer's Second Set of Eyes

**Always review it yourself.** Common AI-generated issues:

- Looks correct but has interaction bugs
- Static screenshot fine but misaligns on scroll
- Looks great wide but breaks narrow
- Dark mode not tested
- Components don't respond after Tweaks toggled

**1 minute of verification saves 1 hour of rework.**

## Common Verification Script Commands

```bash
# Basic: open + screenshot + catch errors
python verify.py design.html

# Multiple viewports
python verify.py design.html --viewports 1920x1080,375x667

# Multiple slides
python verify.py deck.html --slides 10

# Output to specific directory
python verify.py design.html --output ./screenshots/

# headless=false, open in real browser
python verify.py design.html --show
```
