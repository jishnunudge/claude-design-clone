# React + Babel Project Standards

Technical standards for prototyping with HTML + React + Babel. Violating these will break things.

## Pinned Script Tags (use these exact versions)

Place these three script tags in `<head>`, using **fixed versions + integrity hashes**:

```html
<script src="https://unpkg.com/react@18.3.1/umd/react.development.js" integrity="sha384-hD6/rw4ppMLGNu3tX5cjIb+uRZ7UkRJ6BPkLpg4hAu/6onKUg4lLsHAs9EBPT82L" crossorigin="anonymous"></script>
<script src="https://unpkg.com/react-dom@18.3.1/umd/react-dom.development.js" integrity="sha384-u6aeetuaXnQ38mYT8rp6sbXaQe3NL9t+IBXmnYxwkUI2Hw4bsp2Wvmx4yRQF1uAm" crossorigin="anonymous"></script>
<script src="https://unpkg.com/@babel/standalone@7.29.0/babel.min.js" integrity="sha384-m08KidiNqLdpJqLq95G/LEi8Qvjl/xUYll3QILypMoQ65QorJ9Lvtp2RXYGBFj1y" crossorigin="anonymous"></script>
```

**Do not** use unpinned versions like `react@18` or `react@latest` — causes version drift / caching issues.

**Do not** omit `integrity` — last line of defense if CDN is hijacked or tampered with.

## File Structure

```
project-name/
├── index.html               # Main HTML
├── components.jsx           # Component file (loaded with type="text/babel")
├── data.js                  # Data file
└── styles.css               # Additional CSS (optional)
```

Loading order in HTML:

```html
<!-- React + Babel first -->
<script src="https://unpkg.com/react@18.3.1/..."></script>
<script src="https://unpkg.com/react-dom@18.3.1/..."></script>
<script src="https://unpkg.com/@babel/standalone@7.29.0/..."></script>

<!-- Then your component files -->
<script type="text/babel" src="components.jsx"></script>
<script type="text/babel" src="pages.jsx"></script>

<!-- Finally the main entry point -->
<script type="text/babel">
  const root = ReactDOM.createRoot(document.getElementById('root'));
  root.render(<App />);
</script>
```

**Do not** use `type="module"` — conflicts with Babel.

## Three Non-Negotiable Rules

### Rule 1: styles objects must use unique names

**Wrong** (breaks with multiple components):
```jsx
// components.jsx
const styles = { button: {...}, card: {...} };

// pages.jsx  ← same name, overrides the first!
const styles = { container: {...}, header: {...} };
```

**Correct**: use unique prefix per component file.

```jsx
// terminal.jsx
const terminalStyles = { 
  screen: {...}, 
  line: {...} 
};

// sidebar.jsx
const sidebarStyles = { 
  container: {...}, 
  item: {...} 
};
```

**Or use inline styles** (recommended for small components):
```jsx
<div style={{ padding: 16, background: '#111' }}>...</div>
```

**Non-negotiable**. Every `const styles = {...}` must be renamed to something specific — otherwise full-stack errors when multiple components are loaded.

### Rule 2: Scope is not shared — manual export required

**Key concept**: Each `<script type="text/babel">` is compiled independently by Babel; **scopes do not communicate**. `Terminal` defined in `components.jsx` is **undefined by default** in `pages.jsx`.

**Solution**: At end of each component file, export onto `window`:

```jsx
// end of components.jsx
function Terminal(props) { ... }
function Line(props) { ... }
const colors = { green: '#...', red: '#...' };

Object.assign(window, {
  Terminal, Line, colors,
  // list everything you want to use elsewhere
});
```

`pages.jsx` can then use `<Terminal />` directly — JSX looks for `window.Terminal`.

### Rule 3: Do not use scrollIntoView

`scrollIntoView` pushes the entire HTML container upward, breaking the web harness layout. **Never use it**.

Alternatives:
```js
// Scroll to a position within a container
container.scrollTop = targetElement.offsetTop;

// Or use element.scrollTo
container.scrollTo({
  top: targetElement.offsetTop - 100,
  behavior: 'smooth'
});
```

## Calling the Claude API (inside HTML)

Some native design-agent environments (like Claude.ai Artifacts) provide `window.claude.complete` with no config required, but most agent environments (Claude Code / Codex / Cursor / Trae / etc.) **do not** have this locally.

If your HTML prototype needs to call an LLM for a demo, three options:

### Option A: Don't make real calls — use a mock

Recommended for demos. Fake helper returning preset responses:
```jsx
window.claude = {
  async complete(prompt) {
    await new Promise(r => setTimeout(r, 800)); // simulate latency
    return "This is a mock response. Replace with a real API call when deploying.";
  }
};
```

### Option B: Call the Anthropic API for real

Requires an API key; user must enter their own. **Never hard-code a key in HTML**.

```html
<input id="api-key" placeholder="Paste your Anthropic API key" />
<script>
window.claude = {
  async complete(prompt) {
    const key = document.getElementById('api-key').value;
    const res = await fetch('https://api.anthropic.com/v1/messages', {
      method: 'POST',
      headers: {
        'x-api-key': key,
        'anthropic-version': '2023-06-01',
        'content-type': 'application/json',
      },
      body: JSON.stringify({
        model: 'claude-haiku-4-5',
        max_tokens: 1024,
        messages: [{ role: 'user', content: prompt }]
      })
    });
    const data = await res.json();
    return data.content[0].text;
  }
};
</script>
```

**Note**: Direct browser calls to the Anthropic API will hit CORS issues. If preview environment doesn't support CORS bypass, use Option A, or tell the user they need a proxy backend.

### Option C: Use agent's LLM to pre-generate mock data

For local demos only: within current agent session, use agent's LLM capability to pre-generate mock response data, then hard-code into HTML. Runtime has zero API dependency.

## Typical HTML Starter Template

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Your Prototype Name</title>

  <!-- React + Babel pinned -->
  <script src="https://unpkg.com/react@18.3.1/umd/react.development.js" integrity="sha384-hD6/rw4ppMLGNu3tX5cjIb+uRZ7UkRJ6BPkLpg4hAu/6onKUg4lLsHAs9EBPT82L" crossorigin="anonymous"></script>
  <script src="https://unpkg.com/react-dom@18.3.1/umd/react-dom.development.js" integrity="sha384-u6aeetuaXnQ38mYT8rp6sbXaQe3NL9t+IBXmnYxwkUI2Hw4bsp2Wvmx4yRQF1uAm" crossorigin="anonymous"></script>
  <script src="https://unpkg.com/@babel/standalone@7.29.0/babel.min.js" integrity="sha384-m08KidiNqLdpJqLq95G/LEi8Qvjl/xUYll3QILypMoQ65QorJ9Lvtp2RXYGBFj1y" crossorigin="anonymous"></script>

  <style>
    * { box-sizing: border-box; margin: 0; padding: 0; }
    html, body { height: 100%; width: 100%; }
    body { 
      font-family: -apple-system, 'SF Pro Text', sans-serif;
      background: #FAFAFA;
      color: #1A1A1A;
    }
    #root { min-height: 100vh; }
  </style>
</head>
<body>
  <div id="root"></div>

  <!-- Your component files -->
  <script type="text/babel" src="components.jsx"></script>

  <!-- Main entry point -->
  <script type="text/babel">
    const { useState, useEffect } = React;

    function App() {
      return (
        <div style={{padding: 40}}>
          <h1>Hello</h1>
        </div>
      );
    }

    const root = ReactDOM.createRoot(document.getElementById('root'));
    root.render(<App />);
  </script>
</body>
</html>
```

## Common Errors and Fixes

**`styles is not defined` or `Cannot read property 'button' of undefined`**
→ `const styles` defined in one file, overwritten by another. Give each a specific name.

**`Terminal is not defined`**
→ Scope doesn't cross files. Add `Object.assign(window, {Terminal})` at end of the defining file.

**Entire page is white, no errors in console**
→ Likely a JSX syntax error Babel didn't report. Swap `babel.min.js` for unminified `babel.js` — clearer error messages.

**ReactDOM.createRoot is not a function**
→ Wrong version. Confirm react-dom@18.3.1 (not 17 or anything else).

**`Objects are not valid as a React child`**
→ Rendered an object instead of JSX / string. Usually `{someObj}` when you meant `{someObj.name}`.

## How to Split Large Projects into Files

**Single files over 1000 lines** are hard to maintain. Split structure:

```
project/
├── index.html
├── src/
│   ├── primitives.jsx      # Base elements: Button, Card, Badge...
│   ├── components.jsx      # Business components: UserCard, PostList...
│   ├── pages/
│   │   ├── home.jsx        # Home page
│   │   ├── detail.jsx      # Detail page
│   │   └── settings.jsx    # Settings page
│   ├── router.jsx          # Simple router (React state switching)
│   └── app.jsx             # Entry component
└── data.js                 # Mock data
```

Load in order in HTML:
```html
<script type="text/babel" src="src/primitives.jsx"></script>
<script type="text/babel" src="src/components.jsx"></script>
<script type="text/babel" src="src/pages/home.jsx"></script>
<script type="text/babel" src="src/pages/detail.jsx"></script>
<script type="text/babel" src="src/pages/settings.jsx"></script>
<script type="text/babel" src="src/router.jsx"></script>
<script type="text/babel" src="src/app.jsx"></script>
```

**At end of every file**, use `Object.assign(window, {...})` to export anything shared.
