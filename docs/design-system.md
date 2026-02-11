# Dark & Glow Design System

Reference CSS design system for the Designer agent. Customize the accent color and intensity levels per project.

---

## CSS Custom Properties

```css
/* ═══════════════════════════════════════════════════════════
   DESIGN SYSTEM: DARK & GLOW
   Customize: Accent color, glow intensity, animation level
   ═══════════════════════════════════════════════════════════ */

/* ────────────────── RESET ────────────────── */

*, *::before, *::after {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

/* ────────────────── CSS VARIABLES ────────────────── */

:root {
  /* ── Dark Backgrounds (never pure black) ── */
  --bg-base: #0d0d0d;
  --bg-primary: #141414;
  --bg-secondary: #1c1c1e;
  --bg-tertiary: #2c2c2e;
  --bg-elevated: #1a1a1c;
  --bg-input: #1a1a1c;
  --bg-sidebar: #111113;
  --bg-header: rgba(20, 20, 22, 0.85);

  /* ── Borders ── */
  --border-subtle: rgba(255, 255, 255, 0.06);
  --border-default: rgba(255, 255, 255, 0.10);
  --border-strong: rgba(255, 255, 255, 0.16);
  --border-focus: var(--accent);

  /* ── Text Colors ── */
  --text-primary: #f5f5f7;
  --text-secondary: #a1a1a6;
  --text-tertiary: #6e6e73;
  --text-inverse: #000000;
  --text-link: var(--accent);
  --text-danger: #ff453a;
  --text-success: #30d158;
  --text-warning: #ffd60a;

  /* ── Accent Colors (generate variants from base accent) ── */
  --accent: #0a7aff;        /* Override per project */
  --accent-rgb: 10, 122, 255;
  --accent-lightest: #e5f0ff;
  --accent-lighter: #4da3ff;
  --accent-light: #2e8fff;
  --accent-dark: #0066dd;
  --accent-darkest: #004499;

  /* ── Semantic Colors ── */
  --color-success: #30d158;
  --color-success-bg: rgba(48, 209, 88, 0.12);
  --color-error: #ff453a;
  --color-error-bg: rgba(255, 69, 58, 0.12);
  --color-warning: #ffd60a;
  --color-warning-bg: rgba(255, 214, 10, 0.12);
  --color-info: #64d2ff;
  --color-info-bg: rgba(100, 210, 255, 0.12);

  /* ── Typography ── */
  --font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', 'Inter', system-ui, sans-serif;
  --font-mono: 'SF Mono', 'Fira Code', 'Cascadia Code', 'Consolas', monospace;
  --font-size-xs: 0.6875rem;
  --font-size-sm: 0.75rem;
  --font-size-base: 0.875rem;
  --font-size-md: 1rem;
  --font-size-lg: 1.25rem;
  --font-size-xl: 1.5rem;
  --font-size-2xl: 2rem;
  --font-size-3xl: 2.5rem;
  --font-weight-normal: 400;
  --font-weight-medium: 500;
  --font-weight-semibold: 600;
  --font-weight-bold: 700;
  --line-height-tight: 1.25;
  --line-height-normal: 1.5;
  --line-height-relaxed: 1.625;

  /* ── Spacing (4px base unit) ── */
  --space-1: 0.25rem;
  --space-2: 0.5rem;
  --space-3: 0.75rem;
  --space-4: 1rem;
  --space-5: 1.25rem;
  --space-6: 1.5rem;
  --space-8: 2rem;
  --space-10: 2.5rem;
  --space-12: 3rem;
  --space-16: 4rem;

  /* ── Border Radius ── */
  --radius-sm: 6px;
  --radius-md: 8px;
  --radius-lg: 12px;
  --radius-xl: 16px;
  --radius-full: 9999px;

  /* ── Shadows ── */
  --shadow-sm: 0 1px 2px rgba(0, 0, 0, 0.3), 0 1px 3px rgba(0, 0, 0, 0.15);
  --shadow-md: 0 4px 6px rgba(0, 0, 0, 0.3), 0 2px 4px rgba(0, 0, 0, 0.2);
  --shadow-lg: 0 10px 25px rgba(0, 0, 0, 0.4), 0 6px 10px rgba(0, 0, 0, 0.2);

  /* ── Glow Effects ── */
  --glow-xs: 0 0 10px rgba(var(--accent-rgb), 0.15);
  --glow-sm: 0 0 20px rgba(var(--accent-rgb), 0.2), 0 0 40px rgba(var(--accent-rgb), 0.08);
  --glow-md: 0 0 30px rgba(var(--accent-rgb), 0.3), 0 0 60px rgba(var(--accent-rgb), 0.12);
  --glow-lg: 0 0 40px rgba(var(--accent-rgb), 0.4), 0 0 80px rgba(var(--accent-rgb), 0.2);

  /* ── Transitions ── */
  --transition-fast: 150ms cubic-bezier(0.4, 0, 0.2, 1);
  --transition-base: 250ms cubic-bezier(0.4, 0, 0.2, 1);
  --transition-slow: 400ms cubic-bezier(0.4, 0, 0.6, 1);

  /* ── Layout ── */
  --sidebar-width: 260px;
  --header-height: 56px;
  --max-content-width: 1400px;
  --container-padding: var(--space-6);
}
```

---

## Base Styles

```css
html {
  font-size: 16px;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

body {
  font-family: var(--font-family);
  font-size: var(--font-size-base);
  line-height: var(--line-height-normal);
  color: var(--text-primary);
  background: linear-gradient(180deg, var(--bg-primary) 0%, var(--bg-base) 100%);
  min-height: 100vh;
}

h1, h2, h3, h4 {
  font-weight: var(--font-weight-semibold);
  line-height: var(--line-height-tight);
  color: var(--text-primary);
}

h1 { font-size: var(--font-size-2xl); }
h2 { font-size: var(--font-size-xl); }
h3 { font-size: var(--font-size-lg); }

p { color: var(--text-secondary); line-height: var(--line-height-relaxed); }

*:focus-visible {
  outline: 2px solid var(--accent);
  outline-offset: 2px;
  border-radius: var(--radius-sm);
}

@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
  }
}

.sr-only {
  position: absolute; width: 1px; height: 1px; padding: 0; margin: -1px;
  overflow: hidden; clip: rect(0, 0, 0, 0); white-space: nowrap; border: 0;
}
```

---

## Layout Templates

### Layout A: Dashboard with Sidebar

Use for Type C apps (dashboards, admin panels, data apps).

```css
.app-layout {
  display: grid;
  grid-template-columns: var(--sidebar-width) 1fr;
  grid-template-rows: var(--header-height) 1fr;
  grid-template-areas: "sidebar header" "sidebar main";
  min-height: 100vh;
}

.sidebar {
  grid-area: sidebar;
  background: var(--bg-sidebar);
  border-right: 1px solid var(--border-subtle);
  position: fixed; top: 0; left: 0; bottom: 0;
  width: var(--sidebar-width);
  padding: var(--space-4) 0;
  overflow-y: auto;
  z-index: 40;
}

.header {
  grid-area: header;
  background: var(--bg-header);
  backdrop-filter: blur(20px);
  border-bottom: 1px solid var(--border-subtle);
  display: flex; align-items: center; justify-content: space-between;
  padding: 0 var(--container-padding);
  position: sticky; top: 0; z-index: 30;
  height: var(--header-height);
  margin-left: var(--sidebar-width);
}

.main-content {
  grid-area: main;
  padding: var(--container-padding);
  margin-left: var(--sidebar-width);
  max-width: calc(var(--max-content-width) + var(--container-padding) * 2);
}

@media (max-width: 768px) {
  .app-layout { grid-template-columns: 1fr; grid-template-areas: "header" "main"; }
  .sidebar { transform: translateX(-100%); transition: transform var(--transition-base); z-index: 50; }
  .sidebar.open { transform: translateX(0); }
  .header, .main-content { margin-left: 0; }
}
```

### Layout B: Single-Page App

Use for Type A apps (calculators, file viewers, converters).

```css
.app-container {
  max-width: 1200px;
  margin: 0 auto;
  padding: var(--space-6);
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}

.app-header {
  display: flex; align-items: center; justify-content: space-between;
  padding-bottom: var(--space-6);
  border-bottom: 1px solid var(--border-subtle);
  margin-bottom: var(--space-8);
}

.app-body { flex: 1; display: flex; flex-direction: column; gap: var(--space-6); }

@media (max-width: 768px) {
  .app-container { padding: var(--space-4); }
  .app-header { flex-direction: column; gap: var(--space-3); text-align: center; }
}
```

---

## Component Patterns

### Cards

```css
.card {
  background: var(--bg-secondary);
  border: 1px solid var(--border-subtle);
  border-radius: var(--radius-lg);
  padding: var(--space-5);
  transition: all var(--transition-base);
}

.card:hover {
  border-color: var(--border-default);
  box-shadow: var(--shadow-md);
}

.card-glow:hover {
  border-color: rgba(var(--accent-rgb), 0.3);
  box-shadow: var(--glow-sm);
}
```

### Buttons

```css
.btn {
  display: inline-flex; align-items: center; justify-content: center; gap: var(--space-2);
  padding: var(--space-2) var(--space-5);
  border-radius: var(--radius-md);
  font-size: var(--font-size-base);
  font-weight: var(--font-weight-medium);
  cursor: pointer; border: none;
  transition: all var(--transition-fast);
  min-height: 44px;
}

.btn-primary {
  background: var(--accent);
  color: var(--text-inverse);
  box-shadow: var(--glow-xs);
}
.btn-primary:hover { filter: brightness(1.15); box-shadow: var(--glow-sm); }

.btn-secondary {
  background: var(--bg-tertiary);
  color: var(--text-primary);
  border: 1px solid var(--border-default);
}
.btn-secondary:hover { background: rgba(var(--accent-rgb), 0.1); border-color: var(--accent); }
```

### Data Tables

```css
.data-table { width: 100%; border-collapse: collapse; font-size: var(--font-size-sm); }
.data-table th {
  text-align: left; padding: var(--space-3) var(--space-4);
  background: var(--bg-tertiary); color: var(--text-secondary);
  font-weight: var(--font-weight-semibold); font-size: var(--font-size-xs);
  text-transform: uppercase; letter-spacing: 0.05em;
  border-bottom: 1px solid var(--border-default);
}
.data-table td {
  padding: var(--space-3) var(--space-4);
  border-bottom: 1px solid var(--border-subtle);
  color: var(--text-primary);
}
.data-table tr:hover td { background: rgba(var(--accent-rgb), 0.04); }
.data-table .number { text-align: right; font-family: var(--font-mono); }
.data-table .positive { color: var(--color-success); }
.data-table .negative { color: var(--color-error); }
```

### Form Inputs

```css
.form-input {
  width: 100%; padding: var(--space-3) var(--space-4);
  background: var(--bg-input); color: var(--text-primary);
  border: 1px solid var(--border-default); border-radius: var(--radius-md);
  font-size: var(--font-size-base); font-family: var(--font-family);
  transition: all var(--transition-fast); min-height: 44px;
}
.form-input:focus { border-color: var(--accent); box-shadow: var(--glow-xs); }
.form-input::placeholder { color: var(--text-tertiary); }
```

### Alerts

```css
.alert {
  display: flex; align-items: flex-start; gap: var(--space-3);
  padding: var(--space-4); border-radius: var(--radius-md);
  border: 1px solid; margin-bottom: var(--space-4);
}
.alert.success { background: var(--color-success-bg); border-color: rgba(48, 209, 88, 0.3); color: var(--color-success); }
.alert.error { background: var(--color-error-bg); border-color: rgba(255, 69, 58, 0.3); color: var(--color-error); }
.alert.warning { background: var(--color-warning-bg); border-color: rgba(255, 214, 10, 0.3); color: var(--color-warning); }
.alert.info { background: var(--color-info-bg); border-color: rgba(100, 210, 255, 0.3); color: var(--color-info); }
```

### Loading & Empty States

```css
.spinner {
  width: 24px; height: 24px;
  border: 3px solid var(--border-default);
  border-top-color: var(--accent);
  border-radius: 50%;
  animation: spin 0.8s linear infinite;
}
.spinner.lg { width: 48px; height: 48px; border-width: 4px; }
@keyframes spin { to { transform: rotate(360deg); } }

.empty-state {
  display: flex; flex-direction: column; align-items: center; justify-content: center;
  padding: var(--space-16) var(--space-8); text-align: center;
}
.empty-state-icon { font-size: 3rem; margin-bottom: var(--space-4); opacity: 0.5; }
.empty-state-title { font-size: var(--font-size-lg); font-weight: var(--font-weight-semibold); margin-bottom: var(--space-2); }
.empty-state-message { color: var(--text-secondary); max-width: 400px; }
```

### Upload Zone

```css
.upload-zone {
  border: 2px dashed var(--border-default); border-radius: var(--radius-lg);
  padding: var(--space-10) var(--space-8);
  text-align: center; cursor: pointer;
  transition: all var(--transition-base);
}
.upload-zone:hover, .upload-zone.dragover {
  border-color: var(--accent);
  background: rgba(var(--accent-rgb), 0.05);
  box-shadow: var(--glow-xs);
}
```

### Toast Notifications

```css
.toast-container {
  position: fixed; top: var(--space-4); right: var(--space-4);
  z-index: 1000; display: flex; flex-direction: column; gap: var(--space-2);
}
.toast {
  display: flex; align-items: center; gap: var(--space-3);
  padding: var(--space-3) var(--space-4);
  background: var(--bg-elevated); border: 1px solid var(--border-default);
  border-radius: var(--radius-md); box-shadow: var(--shadow-lg);
  animation: toast-enter 0.3s ease;
  min-width: 300px; max-width: 450px;
}
.toast.success { border-left: 3px solid var(--color-success); }
.toast.error { border-left: 3px solid var(--color-error); }
.toast.warning { border-left: 3px solid var(--color-warning); }
@keyframes toast-enter { from { opacity: 0; transform: translateX(100%); } to { opacity: 1; transform: translateX(0); } }
```

---

## Print Styles

```css
@media print {
  body { background: white; color: black; }
  .sidebar, .header, .btn, .upload-zone, .toast-container { display: none !important; }
  .main-content, .app-container { margin: 0; padding: 0; max-width: none; }
  .card { border: 1px solid #ddd; box-shadow: none; background: white; break-inside: avoid; }
  .data-table th { background: #f0f0f0; }
  .data-table td, .data-table th { border: 1px solid #ddd; }
  * { color: black !important; text-shadow: none !important; box-shadow: none !important; }
}
```

---

## Customization Questions

Before designing, the Designer should ask the user:

1. **Accent color:** Default blue (#0a7aff) or a brand color? (hex code)
2. **Glow intensity:** Subtle / Medium (default) / Strong
3. **Animation level:** Minimal / Moderate (default) / Rich
4. **Design references:** Any apps or sites to look similar to?
