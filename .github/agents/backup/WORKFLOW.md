````markdown
# Agent Workflow Guide

## Quick Start

1. **Start with @Architect** — Send your project requirements
2. **Architect creates PCD** — Research, design, data structures, algorithms
3. **UI/UX Designer creates CSS** — Design system, components, layout (if project has UI)
4. **Implementer builds code** — HTML + vanilla JS following PCD exactly
5. **Validator tests** — Context-aware testing with evidence
6. **Debugger fixes** — Surgical bug fixes (only if bugs found)
7. **Done** — Working, tested deliverable

---

## The Project Context Document (PCD)

The PCD is the **single source of truth** for the entire project. Created by Architect, referenced by all downstream agents.

```
┌───────────────────────────────────────────────────────────┐
│                PROJECT CONTEXT DOCUMENT                    │
├───────────────────────────────────────────────────────────┤
│ Section 1:  Project Overview (name, type, description)    │
│ Section 2:  Must-Have Requirements (numbered list)        │
│ Section 3:  Technology Decisions (with reasons)           │
│ Section 4:  Data Structures (schemas + null handling)     │
│ Section 5:  Algorithms (pseudocode + walk-throughs)       │
│ Section 6:  UI Specifications (layout, components)        │
│ Section 7:  Edge Cases (behavior + user messages)         │
│ Section 8:  Success Criteria (measurable, testable)       │
│ Section 9:  Sample Analysis (if applicable)               │
│ Section 10: ADR Index                                     │
│ Section 11: Agent-Specific Instructions                   │
└───────────────────────────────────────────────────────────┘
```

**Why the PCD matters:** Without the PCD, each agent makes independent assumptions and the result is inconsistent. With the PCD, every agent references the same specs, the same data structures, the same edge cases — and the output is coherent.

---

## Agent Pipeline

```
┌─────────────────────────────────────────────────────────┐
│                                                          │
│   User Request                                           │
│       │                                                  │
│       ▼                                                  │
│   ┌──────────┐                                           │
│   │ Architect│ → Creates PCD (NO CODE)                   │
│   └────┬─────┘                                           │
│        │                                                 │
│        ├─── Has UI? ─── YES ──▶ ┌──────────────┐        │
│        │                        │ UI/UX Designer│        │
│        │ NO                     └───────┬──────┘         │
│        │                                │                │
│        ▼                                ▼                │
│   ┌────────────────────────────────────────┐             │
│   │           Implementer                  │             │
│   │  (Follows PCD + uses CSS from Designer)│             │
│   └──────────────┬─────────────────────────┘             │
│                  │                                       │
│                  ▼                                       │
│            ┌──────────┐                                  │
│            │ Validator│                                   │
│            └────┬─────┘                                  │
│                 │                                        │
│           ┌─────┴─────┐                                  │
│           │           │                                  │
│        ALL PASS    BUGS FOUND                            │
│           │           │                                  │
│           ▼           ▼                                  │
│        ✅ Done    ┌──────────┐                            │
│                  │ Debugger │ → Minimal fixes only       │
│                  └────┬─────┘                            │
│                       │                                  │
│                       └──▶ Back to Validator             │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

---

## Phase Details

### Phase 1: Architect (Every Project)

**Agent:** @Architect
**Creates:** Project Context Document (PCD)

**What happens:**
1. Architect asks clarifying questions about the project
2. Analyzes sample files/data if provided
3. Classifies project type (A through G)
4. Designs data structures with schemas
5. Writes algorithms in pseudocode with walk-throughs
6. Documents decisions as ADRs
7. Defines success criteria and edge cases
8. Produces the complete PCD

**Key outputs:**
- Complete PCD (Sections 1–11)
- ADRs in project directory
- Clear instructions for next agent

**Handoff:** PCD passes to UI/UX Designer (if UI project) or Implementer (if no UI).

---

### Phase 2: UI/UX Designer (Only If Project Has UI)

**Agent:** @UI/UX Designer
**Reads:** PCD Sections 1, 2, 6, 7, 11

**What happens:**
1. Creates design system (CSS custom properties)
2. Selects layout template (sidebar+main, single-page, etc.)
3. Writes complete CSS for all components
4. Adds responsive breakpoints
5. Creates print stylesheet (if applicable)
6. Verifies accessibility (contrast, focus states)

**Key outputs:**
- Complete CSS (ready to paste into `<style>`)
- HTML structure guidance for Implementer
- Component state descriptions

**Skip this phase if:** Project has no UI (API, library, CLI tool).

---

### Phase 3: Implementer (Every Project)

**Agent:** @Implementer
**Reads:** Complete PCD + UI/UX Designer's CSS handoff

**What happens:**
1. Copies utility functions (safeGet, escapeHtml, etc.)
2. Creates state management structure
3. Implements Architect's algorithms from PCD Section 5
4. Uses UI/UX Designer's CSS unchanged
5. Handles all edge cases from PCD Section 7
6. Creates test checklist for Validator

**Key outputs:**
- Complete working implementation
- Zero console errors
- Test checklist with step-by-step procedures

---

### Phase 4: Validator (Every Project)

**Agent:** @Validator
**Reads:** PCD Sections 2, 7, 8 + Implementer's test checklist

**What happens:**
1. Classifies project type → determines which tests to run
2. Executes functional tests (every requirement)
3. Executes edge case tests (every edge case)
4. Executes UI state tests (initial, loading, success, error)
5. Runs visual quality assessment
6. Runs cross-browser and responsive tests
7. Produces test report with evidence

**Key outputs:**
- Test report with pass/fail counts
- Evidence for every test
- Bug reports (if failures found)

**Result:**
- **ALL PASS** → Project complete ✅
- **BUGS FOUND** → Handoff to Debugger

---

### Phase 5: Debugger (Only If Bugs Found)

**Agent:** @Debugger
**Reads:** Validator's test report + bug reports

**What happens:**
1. Reproduces each bug (mandatory before fixing)
2. Identifies root causes
3. Plans minimal fixes
4. Implements fixes with inline comments
5. Verifies each fix individually
6. Runs regression tests

**Key outputs:**
- All bugs fixed
- Root cause documentation
- Regression test results
- File change list

**Result:** Hands back to Validator for re-testing.

---

## Project Type Classification

Architect classifies the project type. This determines testing strategy and technology choices.

| Type | Name | When to Use | Default Tech |
|------|------|-------------|-------------|
| **A** | Single-File HTML App | Calculator, converter, file reader, data viewer | 1 HTML file with inline CSS + JS |
| **B** | Multi-File Frontend | Portfolio, documentation, multi-page site | Multiple HTML/CSS/JS files |
| **C** | Full-Stack Web App | Dashboard, task manager, blog, SaaS | Frontend + backend + database |
| **D** | Backend API Only | REST API, microservice, data pipeline | Node.js/Python server |
| **E** | JS Library | Utility library, SDK, helper package | Pure JS functions |
| **F** | CLI Tool | Code generator, build tool, file processor | Node.js CLI |
| **G** | Desktop/Mobile | Electron app, React Native app | Framework-specific |

---

## What Each Agent Reads from the PCD

| PCD Section | Architect | UI/UX Designer | Implementer | Validator | Debugger |
|------------|-----------|---------------|-------------|-----------|----------|
| 1. Project Overview | Creates ✏️ | Reads 📖 | Reads 📖 | Reads 📖 | Reads 📖 |
| 2. Requirements | Creates ✏️ | Reads 📖 | Reads 📖 | **Tests** ✅ | Reads 📖 |
| 3. Tech Decisions | Creates ✏️ | Reads 📖 | Reads 📖 | — | Reads 📖 |
| 4. Data Structures | Creates ✏️ | — | **Implements** 🔨 | — | Reads 📖 |
| 5. Algorithms | Creates ✏️ | — | **Implements** 🔨 | — | Reads 📖 |
| 6. UI Specs | Creates ✏️ | **Designs** 🎨 | **Implements** 🔨 | **Tests** ✅ | Reads 📖 |
| 7. Edge Cases | Creates ✏️ | Reads 📖 | **Handles** 🔨 | **Tests** ✅ | Reads 📖 |
| 8. Success Criteria | Creates ✏️ | Reads 📖 | Reads 📖 | **Verifies** ✅ | — |
| 9. Sample Analysis | Creates ✏️ | — | Reads 📖 | Reads 📖 | Reads 📖 |
| 10. ADR Index | Creates ✏️ | Reads 📖 | Reads 📖 | — | Reads 📖 |
| 11. Agent Instructions | Creates ✏️ | Reads 📖 | Reads 📖 | Reads 📖 | Reads 📖 |

---

## Common Scenarios

### Scenario A: Single-File HTML App (e.g., OFX Reader)

```
@Architect → @UI/UX Designer → @Implementer → @Validator
```

- Architect: Analyzes OFX format, designs parser algorithm, defines data structures
- UI/UX Designer: Dashboard layout with sidebar, stats cards, data table, dark theme
- Implementer: Single HTML file, 7-section code structure, defensive utilities
- Validator: Manual browser tests, sample file tests, print preview. NO API tests.

### Scenario B: JavaScript Library

```
@Architect → @Implementer → @Validator
```

- **Skip UI/UX Designer** — no user interface
- Architect: Defines function signatures, algorithms, edge cases
- Implementer: Pure JS functions with defensive coding
- Validator: Unit tests only. NO browser or UI tests.

### Scenario C: Full-Stack Web App

```
@Architect → @UI/UX Designer → @Implementer → @Validator
```

- Architect: Frontend components, API endpoints, database schema, state management
- UI/UX Designer: Complete design system, responsive layouts, all components
- Implementer: Frontend + backend + database integration
- Validator: UI tests + API tests + database tests + integration tests

---

## Troubleshooting

| Problem | Cause | Fix |
|---------|-------|-----|
| Agent making assumptions | Not following core rule | Remind: "When in doubt → ASK" |
| Agent adding features | Scope creep | Remind: "PCD requirements ONLY" |
| Wrong tests being run | Project type not identified | Check project type classification |
| Implementer changed CSS | Crossed role boundary | Remind: "Use UI/UX Designer's CSS unchanged" |
| Debugger refactoring | Scope creep | Remind: "Minimal fixes ONLY" |
| Context lost between agents | PCD not referenced | Ensure PCD is created and passed |
| Amateur-looking UI | Visual quality not assessed | Run Visual Quality Assessment checklist |
| Silent failures in code | Missing error handling | Add defensive utilities (safeGet, try-catch) |

---

## Tips for Best Results

1. **Start with clear requirements** — Be specific about what you want
2. **Provide sample data** — Real examples are better than descriptions
3. **Answer Architect's questions** — Don't let agents guess
4. **State constraints upfront** — "Single file", "no frameworks", "dark theme", etc.
5. **Define "done"** — What does success look like?
6. **Review the PCD** — When Architect produces it, verify it matches your intent
7. **Check visual quality** — "It works" is not enough; it should look professional

````
