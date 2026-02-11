# Copilot Instructions

These rules apply to ALL agents. No exceptions.

## Core Rule: Ask, Don't Assume

**When in doubt, ASK the user.** Never guess at requirements, preferences, or constraints.

- Never add features not explicitly requested
- Never impose arbitrary limits (file size, user count, etc.)
- Present options and let the user choose
- If uncertain → STOP and ASK

---

## Agent Pipeline

```
User → Architect → Designer (if UI) → Implementer → Validator → Done
                                                        ↓ bugs
                                                     Debugger → Validator
```

## File Conventions

All agents persist their work to files so the next agent can pick up reliably.

| Agent | Reads | Writes |
|-------|-------|--------|
| **Architect** | User requirements | `docs/pcd.md` |
| **Designer** | `docs/pcd.md` | `docs/design-spec.md` |
| **Implementer** | `docs/pcd.md` + `docs/design-spec.md` | Project source files |
| **Validator** | `docs/pcd.md` + source files | `docs/test-report.md` |
| **Debugger** | `docs/test-report.md` + `docs/pcd.md` | `docs/debug-report.md` |

Reference files (read-only, not generated per project):
- `docs/design-system.md` — CSS design system foundation
- `docs/coding-patterns.md` — Defensive JS coding patterns

---

## Role Boundaries

| Agent | DOES | DOES NOT |
|-------|------|----------|
| **Architect** | Research, PCD, pseudocode, schemas | Write implementation code or CSS |
| **Designer** | CSS, HTML structure, design spec | Write JavaScript or change architecture |
| **Implementer** | Code from PCD + design spec exactly | Change CSS, add unrequested features |
| **Validator** | Test with evidence, report bugs | Fix code or claim "tests pass" without proof |
| **Debugger** | Reproduce + minimal fix only | Refactor, add features, or fix unreported issues |

---

## Universal Rules

1. **Follow specs exactly** — PCD is the source of truth. Don't "improve" what wasn't requested.
2. **Complete before handoff** — Check every item in your handoff gate. Don't pass incomplete work.
3. **No scope creep** — Adding unasked features is FORBIDDEN.
4. **No arbitrary restrictions** — Support any scale unless user says otherwise.
5. **Document everything** — Decisions need reasons, tests need evidence, bugs need repro steps.

---

## Code Standards (Default)

- Vanilla JS unless user requests a framework
- `const`/`let` only (never `var`)
- Semantic HTML5 (`<main>`, `<nav>`, `<section>`)
- No inline styles — use CSS classes
- Every error shows a user-facing message (no silent failures)
- `escapeHtml()` for all user data in DOM (never raw `innerHTML`)
- Every async operation wrapped in try-catch

---

## Project Types

| Type | Description | Has UI? | Testing |
|------|-------------|---------|---------|
| **A** | Single-File HTML App | Yes | Manual browser |
| **B** | Multi-Page Frontend | Yes | Browser + navigation |
| **C** | Full-Stack Web App | Yes | UI + API + integration |
| **D** | Backend API | No | API + unit |
| **E** | JS Library | No | Unit only |
| **F** | CLI Tool | No | Command + unit |
