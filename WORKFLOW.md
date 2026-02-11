# Agent Workflow Guide

## Quick Start

1. Select the **Architect** agent in Copilot Chat
2. Describe your project requirements
3. Architect asks questions, then creates `docs/pcd.md`
4. Use the handoff button to move to the next agent
5. Each agent reads the previous agent's output files
6. Repeat until Validator declares all tests pass

---

## Pipeline

```
User → Architect → Designer (if UI) → Implementer → Validator → Done
                                                        ↓ bugs
                                                     Debugger → Validator
```

**How handoffs work:** After each agent finishes, a button appears (e.g., "→ Designer"). Clicking it switches to the next agent with a pre-filled prompt that tells it where to find the files it needs.

---

## What Each Agent Does

### Architect
- Asks clarifying questions about your project
- Classifies project type (A–F)
- Designs data structures (JSON schemas) and algorithms (pseudocode)
- Writes the Project Context Document to `docs/pcd.md`
- **Produces NO implementation code**

### Designer (UI projects only)
- Reads `docs/pcd.md` for requirements and UI specs
- Builds on the design system in `docs/design-system.md`
- Creates complete CSS + HTML structure spec
- Saves to `docs/design-spec.md`
- **Produces NO JavaScript**

### Implementer
- Reads `docs/pcd.md` + `docs/design-spec.md`
- Uses defensive patterns from `docs/coding-patterns.md`
- Implements algorithms exactly as specified in PCD
- Uses Designer's CSS unchanged
- Creates the project source files

### Validator
- Reads `docs/pcd.md` for success criteria and edge cases
- Tests every requirement with evidence
- Matches test strategy to project type (see matrix below)
- Saves results to `docs/test-report.md`
- **ALL PASS → Done** / **BUGS FOUND → hands off to Debugger**

### Debugger (only if bugs found)
- Reads `docs/test-report.md` for bug reports
- Reproduces each bug before fixing
- Applies minimal, surgical fixes
- Saves to `docs/debug-report.md`
- Hands back to Validator for re-test

---

## File Map

```
docs/
├── pcd.md            ← Architect writes, everyone reads
├── design-spec.md    ← Designer writes, Implementer reads
├── design-system.md  ← Reference CSS (template file, read-only)
├── coding-patterns.md← Reference JS patterns (template file, read-only)
├── test-report.md    ← Validator writes, Debugger reads
└── debug-report.md   ← Debugger writes, Validator reads
```

---

## Test Strategy by Project Type

| Type | Browser | Unit | API | Responsive |
|------|---------|------|-----|------------|
| A: Single-File HTML | ✅ | — | — | ✅ |
| B: Multi-Page Frontend | ✅ | If exists | If exists | ✅ |
| C: Full-Stack Web App | ✅ | ✅ | ✅ | ✅ |
| D: Backend API | — | ✅ | ✅ | — |
| E: JS Library | — | ✅ | — | — |
| F: CLI Tool | — | ✅ | If API | — |

---

## Common Scenarios

### Single-File HTML App (calculator, file viewer, converter)
```
Architect → Designer → Implementer → Validator
```
One HTML file. Manual browser testing. No API tests.

### JavaScript Library (utility package, SDK)
```
Architect → Implementer → Validator
```
Skip Designer — no UI. Unit tests only.

### Full-Stack Web App (dashboard, task manager)
```
Architect → Designer → Implementer → Validator
```
Frontend + backend. UI + API + integration tests.

### Backend API (REST API, microservice)
```
Architect → Implementer → Validator
```
Skip Designer — no UI. API + unit tests.

---

## Tips

1. **Be specific** with requirements — vague requests lead to more questions
2. **Provide sample data** — real examples beat descriptions
3. **Review the PCD** — verify it matches your intent before proceeding
4. **State constraints upfront** — "single file", "no frameworks", "dark theme"
5. **Answer all questions** — don't let agents guess
