# Defensive Coding Patterns

Reference patterns for the Implementer agent. Copy and adapt these utilities for each project.

---

## Safe Data Access

```javascript
/**
 * Safely get a nested property. Never throws.
 * Usage: safeGet(obj, 'a.b.c', defaultValue)
 */
function safeGet(obj, path, defaultValue = null) {
  if (obj == null) return defaultValue;
  const keys = path.split('.');
  let current = obj;
  for (const key of keys) {
    if (current == null || typeof current !== 'object') return defaultValue;
    current = current[key];
  }
  return current ?? defaultValue;
}

/**
 * Parse a number safely. Never returns NaN.
 */
function safeNumber(value, defaultValue = 0) {
  if (value == null) return defaultValue;
  if (typeof value === 'number') return isNaN(value) ? defaultValue : value;
  const parsed = parseFloat(String(value).replace(/[^\d.-]/g, ''));
  return isNaN(parsed) ? defaultValue : parsed;
}

/**
 * Safe string — never null/undefined, always trimmed.
 */
function safeString(value, defaultValue = '') {
  if (value == null) return defaultValue;
  return String(value).trim() || defaultValue;
}

/**
 * Safe array — always returns an array.
 */
function safeArray(value) {
  if (Array.isArray(value)) return value;
  if (value == null) return [];
  return [value];
}
```

---

## XSS Prevention

```javascript
/**
 * Escape HTML to prevent XSS. Use when inserting user data into DOM.
 */
function escapeHtml(str) {
  if (str == null) return '';
  const div = document.createElement('div');
  div.textContent = String(str);
  return div.innerHTML;
}

/**
 * Create DOM element safely. Preferred over innerHTML with user data.
 */
function createElement(tag, attributes = {}, children = []) {
  const el = document.createElement(tag);
  for (const [key, value] of Object.entries(attributes)) {
    if (key === 'className') el.className = value;
    else if (key === 'textContent') el.textContent = value;
    else if (key.startsWith('on') && typeof value === 'function')
      el.addEventListener(key.slice(2).toLowerCase(), value);
    else el.setAttribute(key, value);
  }
  for (const child of safeArray(children)) {
    if (typeof child === 'string') el.appendChild(document.createTextNode(child));
    else if (child instanceof Node) el.appendChild(child);
  }
  return el;
}
```

---

## State Management

```javascript
// Single state object — all UI reads from this
const AppState = {
  status: 'initial', // 'initial' | 'loading' | 'loaded' | 'error'
  data: null,
  error: null,
  warnings: [],
};

/**
 * Update state and re-render. This is the ONLY way to change the UI.
 */
function updateState(newState) {
  Object.assign(AppState, newState);
  render();
}

/**
 * Main render — dispatches to state-specific renderers.
 */
function render() {
  const container = document.getElementById('app-body');
  if (!container) return;
  container.innerHTML = '';

  switch (AppState.status) {
    case 'initial': renderInitialState(container); break;
    case 'loading': renderLoadingState(container); break;
    case 'loaded':  renderLoadedState(container); break;
    case 'error':   renderErrorState(container); break;
    default:        renderErrorState(container);
  }
}
```

---

## Error Boundaries

```javascript
/**
 * Wrap event handlers to prevent crashes.
 */
function safeHandler(fn) {
  return function(event) {
    try {
      fn.call(this, event);
    } catch (error) {
      console.error('[Event Handler Error]', error);
      showToast('error', 'Error', 'Something went wrong. Please try again.');
    }
  };
}
```

---

## Formatting Utilities

```javascript
function formatCurrency(amount, currency = 'BRL', locale = 'pt-BR') {
  try {
    return new Intl.NumberFormat(locale, { style: 'currency', currency }).format(safeNumber(amount));
  } catch {
    return `${currency} ${safeNumber(amount).toFixed(2)}`;
  }
}

function formatDate(dateValue, locale = 'pt-BR') {
  if (!dateValue) return '—';
  try {
    const date = new Date(dateValue);
    return isNaN(date.getTime()) ? '—' : date.toLocaleDateString(locale);
  } catch {
    return '—';
  }
}
```

---

## Toast Notifications

```javascript
function showToast(type, title, message, duration = 5000) {
  const container = document.querySelector('.toast-container')
    || (() => {
      const c = createElement('div', { className: 'toast-container' });
      document.body.appendChild(c);
      return c;
    })();

  const icons = { success: '✓', error: '✕', warning: '⚠', info: 'ℹ' };
  const toast = createElement('div', { className: `toast ${type}` }, [
    createElement('span', { className: 'toast-icon', textContent: icons[type] || 'ℹ' }),
    createElement('div', { className: 'toast-content' }, [
      createElement('div', { className: 'toast-title', textContent: title }),
      createElement('div', { className: 'toast-message', textContent: message }),
    ]),
    createElement('button', {
      className: 'toast-close', textContent: '✕', 'aria-label': 'Close',
      onClick: () => { toast.style.opacity = '0'; setTimeout(() => toast.remove(), 200); }
    }),
  ]);

  container.appendChild(toast);
  if (duration > 0) setTimeout(() => { if (toast.parentNode) toast.remove(); }, duration);
}
```

---

## Code Organization Template

```javascript
'use strict';

// ═══════════════════════════════════════════════════════════
// SECTION 1: CONFIGURATION & CONSTANTS
// ═══════════════════════════════════════════════════════════
const CONFIG = { /* all configurable values */ };

// ═══════════════════════════════════════════════════════════
// SECTION 2: STATE MANAGEMENT
// ═══════════════════════════════════════════════════════════
// AppState + updateState + render

// ═══════════════════════════════════════════════════════════
// SECTION 3: UTILITY FUNCTIONS
// ═══════════════════════════════════════════════════════════
// safeGet, safeNumber, escapeHtml, createElement, etc.

// ═══════════════════════════════════════════════════════════
// SECTION 4: CORE LOGIC (from PCD algorithms)
// ═══════════════════════════════════════════════════════════
// Each PCD algorithm becomes a function

// ═══════════════════════════════════════════════════════════
// SECTION 5: UI RENDERING (from design spec)
// ═══════════════════════════════════════════════════════════
// renderInitialState, renderLoadingState, renderLoadedState, renderErrorState

// ═══════════════════════════════════════════════════════════
// SECTION 6: EVENT HANDLERS (wrapped in safeHandler)
// ═══════════════════════════════════════════════════════════

// ═══════════════════════════════════════════════════════════
// SECTION 7: INITIALIZATION
// ═══════════════════════════════════════════════════════════
document.addEventListener('DOMContentLoaded', () => {
  try {
    updateState({ status: 'initial' });
  } catch (error) {
    console.error('[init] Failed:', error);
    document.body.innerHTML = '<p>Failed to start. Please refresh.</p>';
  }
});
```

---

## Key Rules

- `const` and `let` only (never `var`)
- Descriptive names (`transactionCount` not `tc`)
- Comments explain WHY, not WHAT
- Every `async` has `try-catch`
- Every DOM access has a null check
- Semantic HTML5 (`main`, `nav`, `section`, `article`)
- No inline styles (use CSS classes)
- No `innerHTML` with user data (use `textContent` or `escapeHtml`)
- No uncaught exceptions
- No silent failures (always show user a message)
