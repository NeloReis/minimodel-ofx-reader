````skill
---
name: Context-Aware Testing Decision Tree
description: Determines exactly which tests to run based on project type. Prevents running irrelevant tests (like API tests on static HTML) and ensures appropriate coverage. Used by Validator and referenced by all agents.
---

# Context-Aware Testing Decision Tree

This skill prevents the most common testing mistake: **running the wrong tests for the project type.**

---

## Step 1: Classify the Project

Read the PCD (Project Context Document) from Architect. Find the project type classification.

If there is no PCD, classify the project yourself using these indicators:

| Type | Name | Key Indicators | Examples |
|------|------|---------------|----------|
| **A** | Single-File HTML App | One `.html` file, inline CSS/JS, no server, no API calls, client-side processing | Calculator, file converter, OFX reader, data viewer |
| **B** | Multi-File Frontend | Multiple HTML/CSS/JS files, no backend, may call external APIs, static hosting | Portfolio, documentation site, landing page |
| **C** | Full-Stack Web App | Frontend + backend server + database (optional), client-server communication | Task manager, blog, admin dashboard, SaaS app |
| **D** | Backend API Only | Server-side code only, REST/GraphQL, no frontend UI | REST API, microservice, data pipeline |
| **E** | JS Library / Package | Pure functions, no UI, meant to be `import`ed, reusable | Utility library, SDK, helper package |
| **F** | CLI Tool | Command-line interface, terminal-based, no web UI | Build tool, code generator, file processor |
| **G** | Desktop/Mobile App | Electron, React Native, or similar framework | Desktop app, mobile app |

**If the project doesn't fit any type → ASK the user.**

---

## Step 2: Look Up Required Tests

Use this matrix to determine exactly which tests to run:

| Test Category | A: Single HTML | B: Multi-File | C: Full-Stack | D: API Only | E: Library | F: CLI |
|--------------|---------------|--------------|---------------|-------------|------------|--------|
| **Manual Browser Testing** | ✅ REQUIRED | ✅ REQUIRED | ✅ REQUIRED | ❌ SKIP | ❌ SKIP | ❌ SKIP |
| **Functional Testing** | ✅ REQUIRED | ✅ REQUIRED | ✅ REQUIRED | ✅ REQUIRED | ✅ REQUIRED | ✅ REQUIRED |
| **Edge Case Testing** | ✅ REQUIRED | ✅ REQUIRED | ✅ REQUIRED | ✅ REQUIRED | ✅ REQUIRED | ✅ REQUIRED |
| **UI State Testing** | ✅ REQUIRED | ✅ REQUIRED | ✅ REQUIRED | ❌ SKIP | ❌ SKIP | ❌ SKIP |
| **Visual Quality Assessment** | ✅ REQUIRED | ✅ REQUIRED | ✅ REQUIRED | ❌ SKIP | ❌ SKIP | ❌ SKIP |
| **Responsive Testing** | ✅ REQUIRED | ✅ REQUIRED | ✅ REQUIRED | ❌ SKIP | ❌ SKIP | ❌ SKIP |
| **Cross-Browser Testing** | ✅ Chrome+Firefox | ✅ Chrome+Firefox | ✅ Chrome+Firefox+Safari | ❌ SKIP | ❌ SKIP | ❌ SKIP |
| **Print/PDF Testing** | ⚠️ If feature exists | ⚠️ If feature exists | ⚠️ If feature exists | ❌ SKIP | ❌ SKIP | ❌ SKIP |
| **API Endpoint Testing** | ❌ SKIP | ⚠️ If calls external API | ✅ REQUIRED | ✅ REQUIRED | ❌ SKIP | ⚠️ If calls API |
| **Unit Testing** | ❌ SKIP | ❌ SKIP | ⚠️ If complex logic | ✅ REQUIRED | ✅ REQUIRED | ✅ REQUIRED |
| **Database Testing** | ❌ SKIP | ❌ SKIP | ✅ If has DB | ✅ If has DB | ❌ SKIP | ❌ SKIP |
| **E2E Framework** | ❌ NEVER | ❌ SKIP | ⚠️ If complex workflows | ❌ SKIP | ❌ SKIP | ❌ SKIP |
| **Command Testing** | ❌ SKIP | ❌ SKIP | ❌ SKIP | ❌ SKIP | ❌ SKIP | ✅ REQUIRED |
| **Accessibility (basic)** | ✅ REQUIRED | ✅ REQUIRED | ✅ REQUIRED | ❌ SKIP | ❌ SKIP | ❌ SKIP |

**Legend:**
- ✅ REQUIRED — Must run. No exceptions.
- ⚠️ Conditional — Run only if the feature/capability exists.
- ❌ SKIP — Do NOT run. Would be irrelevant.
- ❌ NEVER — Actively harmful to run (e.g., E2E framework on a single HTML file).

---

## Step 3: Choose Testing Method

### Manual Browser Testing (Types A, B, C)

**When:** Project has a UI that runs in a browser.

**How:**
1. Open the HTML file or start the dev server
2. Open browser DevTools Console (F12) BEFORE testing
3. Test every feature by clicking through the UI
4. Check console for errors after each action
5. Document results with evidence

**Tools:** Browser + DevTools. Nothing else needed.

### Unit Testing (Types D, E, F)

**When:** Project is a library, API, or CLI with testable functions.

**How:**
1. Write tests for each public function
2. Cover: valid input, invalid input, edge cases, null/undefined
3. Run test suite and collect results
4. Document coverage

**Tools:** Vitest, Jest, or built-in test runner.

### API Testing (Types C, D)

**When:** Project has backend endpoints.

**How:**
1. Test each endpoint: correct method, correct URL
2. Test with valid payload → expect success
3. Test with invalid payload → expect proper error
4. Test authentication if applicable
5. Check response format and status codes

**Tools:** curl, Postman, Supertest, or fetch in test files.

### Command Testing (Type F)

**When:** Project is a CLI tool.

**How:**
1. Run each command with valid arguments
2. Run with missing/invalid arguments → expect helpful error
3. Test --help flag
4. Check exit codes
5. Verify output format

**Tools:** Terminal.

---

## Decision Flowchart

```
START: What type of project is this?
│
├─ Does it have a web UI?
│  ├─ YES → Is it one HTML file?
│  │         ├─ YES → Type A (Manual browser tests ONLY)
│  │         └─ NO → Does it have a backend server?
│  │                  ├─ YES → Type C (Manual + API + DB tests)
│  │                  └─ NO → Type B (Manual browser tests)
│  └─ NO → Is it an API server?
│           ├─ YES → Type D (API + Unit tests)
│           └─ NO → Is it a CLI tool?
│                    ├─ YES → Type F (Command + Unit tests)
│                    └─ NO → Type E (Unit tests)
```

---

## Evidence Requirements by Test Type

| Test Type | Required Evidence |
|-----------|------------------|
| Manual Browser | What was clicked, what appeared, pass/fail |
| Functional | Input used, expected output, actual output |
| Edge Case | Edge case description, input, expected vs actual |
| Visual Quality | Checklist items with pass/fail for each |
| Responsive | Viewport size tested, layout description |
| Cross-Browser | Browser name + version, pass/fail per browser |
| Print/PDF | Print preview description, checklist results |
| API | Request sent, response received, status code |
| Unit | Test name, assertion, result |
| Command | Command run, output, exit code |

---

## Common Mistakes This Skill Prevents

| Mistake | Project Type | Why It's Wrong | Correct Action |
|---------|-------------|---------------|----------------|
| Running API tests | Type A (Single HTML) | There is no API to test | Skip API tests entirely |
| Using Playwright/Cypress | Type A (Single HTML) | Massive overkill for one file | Use manual browser testing |
| Browser testing | Type D (API Only) | There is no UI to test | Use API/unit tests |
| Database testing | Type A (Single HTML) | There is no database | Skip DB tests entirely |
| E2E framework | Type E (Library) | Libraries don't have UIs | Use unit tests |
| Skipping visual QA | Type A/B/C (Any UI) | "It works" ≠ "It looks good" | Run Visual Quality Assessment |
| Skipping error states | Type A/B/C (Any UI) | Error paths are where bugs hide | Test every error state |
| Only Chrome testing | Type A/B/C (Any UI) | Browser differences cause bugs | Test Chrome + Firefox minimum |

---

## Quick Decision Reference

**I have a single HTML file with inline JS:**
→ Type A → Manual browser tests, edge cases, visual QA, responsive. Skip everything else.

**I have a frontend with separate HTML/CSS/JS files:**
→ Type B → Same as Type A but also test navigation between pages.

**I have a React/Vue/vanilla frontend + Node/Python backend:**
→ Type C → Manual browser tests + API endpoint tests + database tests.

**I have a REST API with no UI:**
→ Type D → API endpoint tests + unit tests. No browser testing.

**I have a utility library:**
→ Type E → Unit tests only. No browser, no API, no UI testing.

**I have a CLI tool:**
→ Type F → Command tests + unit tests. Run in terminal, not browser.

````
