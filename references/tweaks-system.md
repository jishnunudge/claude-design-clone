# Tweaks: Real-Time Design Variant Parameter Tuning

Tweaks are a core capability of this skill — letting users switch between variations and adjust parameters in real time without touching the code.

**Cross-agent environment compatibility**: Some native design-agent environments (like Claude.ai Artifacts) rely on the host's postMessage to write tweak values back to the source file for persistence. This skill uses a **pure frontend localStorage approach** — the effect is identical (state survives refresh), but persistence happens in browser localStorage rather than the source file. This approach works in any agent environment (Claude Code / Codex / Cursor / Trae / etc.).

## When to Add Tweaks

- The user explicitly asks for "parameter tuning" / "switching between multiple versions"
- When the design has multiple variations that need to be compared
- The user hasn't said so, but you judge subjectively that **adding a few insightful tweaks would help the user see the possibility space**

Default recommendation: **Add 2-3 tweaks to every design** (color theme / font size / layout variant) even if the user didn't ask — showing users the space of possibilities is part of the design service.

## Implementation (Pure Frontend Version)

### Basic Structure

```jsx
const TWEAK_DEFAULTS = {
  "primaryColor": "#D97757",
  "fontSize": 16,
  "density": "comfortable",
  "dark": false
};

function useTweaks() {
  const [tweaks, setTweaks] = React.useState(() => {
    try {
      const stored = localStorage.getItem('design-tweaks');
      return stored ? { ...TWEAK_DEFAULTS, ...JSON.parse(stored) } : TWEAK_DEFAULTS;
    } catch {
      return TWEAK_DEFAULTS;
    }
  });

  const update = (patch) => {
    const next = { ...tweaks, ...patch };
    setTweaks(next);
    try {
      localStorage.setItem('design-tweaks', JSON.stringify(next));
    } catch {}
  };

  const reset = () => {
    setTweaks(TWEAK_DEFAULTS);
    try {
      localStorage.removeItem('design-tweaks');
    } catch {}
  };

  return { tweaks, update, reset };
}
```

### Tweaks Panel UI

Floating panel in the bottom-right corner. Collapsible:

```jsx
function TweaksPanel() {
  const { tweaks, update, reset } = useTweaks();
  const [open, setOpen] = React.useState(false);

  return (
    <div style={{
      position: 'fixed',
      bottom: 20,
      right: 20,
      zIndex: 9999,
    }}>
      {open ? (
        <div style={{
          background: 'white',
          border: '1px solid #e5e5e5',
          borderRadius: 12,
          padding: 20,
          boxShadow: '0 10px 40px rgba(0,0,0,0.12)',
          width: 280,
          fontFamily: 'system-ui',
          fontSize: 13,
        }}>
          <div style={{ 
            display: 'flex', 
            justifyContent: 'space-between', 
            alignItems: 'center',
            marginBottom: 16,
          }}>
            <strong>Tweaks</strong>
            <button onClick={() => setOpen(false)} style={{
              border: 'none', background: 'none', cursor: 'pointer', fontSize: 16,
            }}>×</button>
          </div>

          {/* Color */}
          <label style={{ display: 'block', marginBottom: 12 }}>
            <div style={{ marginBottom: 4, color: '#666' }}>Primary Color</div>
            <input 
              type="color" 
              value={tweaks.primaryColor} 
              onChange={e => update({ primaryColor: e.target.value })}
              style={{ width: '100%', height: 32 }}
            />
          </label>

          {/* Font size slider */}
          <label style={{ display: 'block', marginBottom: 12 }}>
            <div style={{ marginBottom: 4, color: '#666' }}>Font Size ({tweaks.fontSize}px)</div>
            <input 
              type="range" 
              min={12} max={24} step={1}
              value={tweaks.fontSize}
              onChange={e => update({ fontSize: +e.target.value })}
              style={{ width: '100%' }}
            />
          </label>

          {/* Density options */}
          <label style={{ display: 'block', marginBottom: 12 }}>
            <div style={{ marginBottom: 4, color: '#666' }}>Density</div>
            <select 
              value={tweaks.density}
              onChange={e => update({ density: e.target.value })}
              style={{ width: '100%', padding: 6 }}
            >
              <option value="compact">Compact</option>
              <option value="comfortable">Comfortable</option>
              <option value="spacious">Spacious</option>
            </select>
          </label>

          {/* Dark mode toggle */}
          <label style={{ 
            display: 'flex', 
            alignItems: 'center',
            gap: 8,
            marginBottom: 16,
          }}>
            <input 
              type="checkbox" 
              checked={tweaks.dark}
              onChange={e => update({ dark: e.target.checked })}
            />
            <span>Dark Mode</span>
          </label>

          <button onClick={reset} style={{
            width: '100%',
            padding: '8px 12px',
            background: '#f5f5f5',
            border: 'none',
            borderRadius: 6,
            cursor: 'pointer',
            fontSize: 12,
          }}>Reset</button>
        </div>
      ) : (
        <button 
          onClick={() => setOpen(true)}
          style={{
            background: '#1A1A1A',
            color: 'white',
            border: 'none',
            borderRadius: 999,
            padding: '10px 16px',
            fontSize: 12,
            cursor: 'pointer',
            boxShadow: '0 4px 12px rgba(0,0,0,0.15)',
          }}
        >⚙ Tweaks</button>
      )}
    </div>
  );
}
```

### Applying Tweaks

Use Tweaks in the main component:

```jsx
function App() {
  const { tweaks } = useTweaks();

  return (
    <div style={{
      '--primary': tweaks.primaryColor,
      '--font-size': `${tweaks.fontSize}px`,
      background: tweaks.dark ? '#0A0A0A' : '#FAFAFA',
      color: tweaks.dark ? '#FAFAFA' : '#1A1A1A',
    }}>
      {/* Your content */}
      <TweaksPanel />
    </div>
  );
}
```

Use the variables in CSS:

```css
button.cta {
  background: var(--primary);
  color: white;
  font-size: var(--font-size);
}
```

## Typical Tweak Options

What tweaks to add for different design types:

### General
- Primary color (color picker)
- Font size (slider 12-24px)
- Typeface (select: display font vs body font)
- Dark mode (toggle)

### Slide Deck
- Theme (light / dark / brand)
- Background style (solid / gradient / image)
- Font contrast (more decorative vs more restrained)
- Information density (minimal / standard / dense)

### Product Prototype
- Layout variant (layout A / B / C)
- Animation speed (0.5x-2x)
- Data volume (mock data count: 5 / 20 / 100)
- State (empty / loading / success / error)

### Animation
- Speed (0.5x-2x)
- Loop (once / loop / ping-pong)
- Easing (linear / easeOut / spring)

### Landing Page
- Hero style (image / gradient / pattern / solid)
- CTA copy (a few variants)
- Structure (single column / two column / sidebar)

## Tweaks Design Principles

### 1. Meaningful options — not busywork

Every tweak must expose **real design choices**. Don't add tweaks that no one would actually switch (e.g., a border-radius 0-50px slider — users will find that every intermediate value looks bad).

Good tweaks expose **discrete, considered variations**:
- "Corner style": No rounding / Subtle rounding / Full rounding (three options)
- Not: "Corner radius": 0-50px slider

### 2. Less is more

A design's Tweaks panel should have **at most 5-6 options**. Any more and it becomes a "configuration page," losing the point of quickly exploring variations.

### 3. Default value is a finished design

Tweaks are the **icing on the cake**. The default values must already constitute a complete, publishable design. What the user sees after closing the Tweaks panel is the deliverable.

### 4. Group options logically

When there are many options, group them:

```
---- Visual ----
Primary Color | Font Size | Dark Mode

---- Layout ----
Density | Sidebar Position

---- Content ----
Data Volume | State
```

## Forward Compatibility with Source-Level Persistent Hosts

If you later want to upload the design to an environment that supports source-level tweaks (like Claude.ai Artifacts), keep the **EDITMODE marker block**:

```jsx
const TWEAK_DEFAULTS = /*EDITMODE-BEGIN*/{
  "primaryColor": "#D97757",
  "fontSize": 16,
  "density": "comfortable",
  "dark": false
}/*EDITMODE-END*/;
```

The marker block has **no effect** in the localStorage approach (it's just a plain comment), but in hosts that support source write-back it will be read and enable source-level persistence. Adding this does no harm to the current environment while maintaining forward compatibility.

## Common Issues

**The Tweaks panel covers the design content**
→ Make it collapsible. Collapsed by default, showing a small button — users expand it when they want it.

**Users have to re-enter settings after switching tweaks**
→ Already using localStorage. If state doesn't persist after a refresh, check whether localStorage is available (private/incognito mode will fail — wrap in a try/catch).

**Multiple HTML pages need to share tweaks**
→ Add the project name to the localStorage key: `design-tweaks-[projectName]`.

**I want tweaks to have linked/dependent relationships**
→ Add logic inside `update`:

```jsx
const update = (patch) => {
  let next = { ...tweaks, ...patch };
  // Linked behavior: switching to dark mode automatically changes text color
  if (patch.dark === true && !patch.textColor) {
    next.textColor = '#F0EEE6';
  }
  setTweaks(next);
  localStorage.setItem(...);
};
```
