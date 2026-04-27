# Tweaks: Real-Time Design Variant Parameter Tuning

Tweaks are a core capability of this skill — letting users switch variations and adjust parameters in real time without touching code.

**Cross-agent environment compatibility**: Some native design-agent environments (like Claude.ai Artifacts) rely on host's postMessage to write tweak values back to source file for persistence. This skill uses **pure frontend localStorage** — identical effect (state survives refresh), but persistence happens in browser localStorage rather than source file. Works in any agent environment (Claude Code / Codex / Cursor / Trae / etc.).

## When to Add Tweaks

- User explicitly asks for "parameter tuning" / "switching between multiple versions"
- Design has multiple variations needing comparison
- User hasn't asked, but **adding a few insightful tweaks would help user see possibility space**

Default recommendation: **Add 2-3 tweaks to every design** (color theme / font size / layout variant) even without asking — showing users possibility space is part of design service.

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

Floating panel bottom-right, collapsible:

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

Use variables in CSS:

```css
button.cta {
  background: var(--primary);
  color: white;
  font-size: var(--font-size);
}
```

## Typical Tweak Options

### General
- Primary color (color picker)
- Font size (slider 12-24px)
- Typeface (select: display vs body font)
- Dark mode (toggle)

### Slide Deck
- Theme (light / dark / brand)
- Background style (solid / gradient / image)
- Font contrast (more decorative vs more restrained)
- Information density (minimal / standard / dense)

### Product Prototype
- Layout variant (A / B / C)
- Animation speed (0.5x-2x)
- Data volume (mock count: 5 / 20 / 100)
- State (empty / loading / success / error)

### Animation
- Speed (0.5x-2x)
- Loop (once / loop / ping-pong)
- Easing (linear / easeOut / spring)

### Landing Page
- Hero style (image / gradient / pattern / solid)
- CTA copy (variants)
- Structure (single column / two column / sidebar)

## Tweaks Design Principles

### 1. Meaningful options — not busywork

Every tweak must expose **real design choices**. Don't add tweaks no one would actually switch (e.g., border-radius 0-50px slider — every intermediate value looks bad).

Good tweaks expose **discrete, considered variations**:
- "Corner style": No rounding / Subtle rounding / Full rounding (three options)
- Not: "Corner radius": 0-50px slider

### 2. Less is more

Tweaks panel should have **at most 5-6 options**. More becomes "configuration page," losing the point.

### 3. Default value is a finished design

Tweaks are **icing on the cake**. Default values must already constitute complete, publishable design. What user sees after closing the panel is the deliverable.

### 4. Group options logically

```
---- Visual ----
Primary Color | Font Size | Dark Mode

---- Layout ----
Density | Sidebar Position

---- Content ----
Data Volume | State
```

## Forward Compatibility with Source-Level Persistent Hosts

To upload later to environments supporting source-level tweaks (like Claude.ai Artifacts), keep the **EDITMODE marker block**:

```jsx
const TWEAK_DEFAULTS = /*EDITMODE-BEGIN*/{
  "primaryColor": "#D97757",
  "fontSize": 16,
  "density": "comfortable",
  "dark": false
}/*EDITMODE-END*/;
```

Marker has **no effect** in localStorage approach (plain comment), but in hosts supporting source write-back it enables source-level persistence. Adding it does no harm and maintains forward compatibility.

## Common Issues

**Tweaks panel covers design content**
→ Make collapsible. Collapsed by default showing small button — expand when needed.

**Users must re-enter settings after switching tweaks**
→ Already using localStorage. If state doesn't persist after refresh, check localStorage availability (private/incognito mode will fail — wrap in try/catch).

**Multiple HTML pages need shared tweaks**
→ Add project name to localStorage key: `design-tweaks-[projectName]`.

**Want tweaks with linked/dependent relationships**
→ Add logic inside `update`:

```jsx
const update = (patch) => {
  let next = { ...tweaks, ...patch };
  // Linked behavior: dark mode automatically changes text color
  if (patch.dark === true && !patch.textColor) {
    next.textColor = '#F0EEE6';
  }
  setTweaks(next);
  localStorage.setItem(...);
};
```
