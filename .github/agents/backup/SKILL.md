git pushgit push---
name: Context-Aware Testing Decision Tree
description: Helps determine appropriate testing strategies based on project type. Use when validating any project to choose context-appropriate tests and avoid irrelevant testing.
---

# Context-Aware Testing Decision Tree

This skill helps you determine **what tests are appropriate** for the specific project type.

---

## 🎯 Purpose

Prevent common testing mistakes:
- ❌ Running API tests on static HTML files
- ❌ Using E2E frameworks for simple projects
- ❌ Testing features that don't exist
- ❌ Wasting time on irrelevant test types

---

## Step 1: Identify Project Type

Ask yourself: **What type of project is this?**

### Type A: Single-File HTML Application
**Indicators:**
- One HTML file with inline CSS and JavaScript
- No backend server
- No external API calls
- Client-side only processing
- Examples: Calculator, file converter, data viewer, OFX reader

**Verdict:** Static client-side app

---

### Type B: Multi-File Frontend (No Backend)
**Indicators:**
- Multiple HTML/CSS/JS files
- No server-side code
- May call external APIs
- Static file hosting
- Examples: Portfolio site, documentation, landing page

**Verdict:** Static frontend app

---

### Type C: Frontend with Backend API
**Indicators:**
- Frontend code (React, Vue, vanilla JS)
- Backend API (Node.js, Python, etc.)
- Database (optional)
- Client-server communication
- Examples: Task manager, blog platform, dashboard

**Verdict:** Full-stack web app

---

### Type D: Backend API Only
**Indicators:**
- Server-side code only
- REST/GraphQL API
- Database integration
- No frontend UI
- Examples: REST API, microservice, data processor

**Verdict:** Backend service

---

### Type E: JavaScript Library/Package
**Indicators:**
- Pure JavaScript functions
- No UI components
- Meant to be imported/required
- May be browser or Node.js
- Examples: Utility library, helper functions, SDK

**Verdict:** Library/package

---

### Type F: CLI Tool
**Indicators:**
- Command-line interface
- Node.js or other runtime
- No web UI
- Terminal-based interaction
- Examples: Build tool, code generator, file processor

**Verdict:** CLI application

---

## Step 2: Choose Appropriate Tests

### For Type A: Single-File HTML Application

#### ✅ REQUIRED Tests:
1. **Manual Browser Testing**
   - Open in Chrome, Firefox, Safari
   - Test all user interactions
   - Verify display/rendering
   - Check console for errors

2. **Input/Output Testing**
   - Test with sample files/data
   - Verify processing logic
   - Check edge cases from Architect's spec

3. **Print Testing** (if print feature exists)
   - Open print preview
   - Generate PDF
   - Verify layout, no dark backgrounds

4. **Responsive Testing** (if responsive design)
   - Desktop viewport
   - Tablet viewport
   - Mobile viewport

#### ❌ SKIP These Tests:
- API endpoint testing (no API)
- E2E frameworks (Playwright, Cypress - overkill)
- Backend integration tests (no backend)
- Database tests (no database)
- Load testing (single user)
- Server tests (no server)

---

### For Type B: Multi-File Frontend

#### ✅ REQUIRED Tests:
1. **Navigation Testing**
   - All links work
   - Page transitions smooth
   - Back/forward browser buttons

2. **Manual Browser Testing**
   - Multiple browsers
   - Different screen sizes

3. **External API Tests** (if calling external APIs)
   - Mock API responses
   - Handle API errors gracefully

#### ❌ SKIP These Tests:
- Backend API tests (no backend)
- Database tests (no database)
- Server-side logic tests

---

### For Type C: Full-Stack Web App

#### ✅ REQUIRED Tests:
1. **Frontend Manual Testing**
   - UI interactions
   - Browser compatibility
   - Responsive design

2. **API Endpoint Testing**
   - Test each endpoint
   - Verify request/response
   - Check error handling
   - Authentication/authorization

3. **Integration Testing**
   - Frontend + Backend flow
   - Database operations
   - End-to-end workflows

4. **Database Testing**
   - CRUD operations
   - Data integrity
   - Migrations (if applicable)

#### ⚠️ OPTIONAL:
- E2E frameworks (appropriate here if complex workflows)
- Load testing (if performance critical)

---

### For Type D: Backend API Only

#### ✅ REQUIRED Tests:
1. **API Endpoint Testing**
   - All routes/endpoints
   - Request validation
   - Response format
   - Status codes
   - Error handling

2. **Database Testing**
   - CRUD operations
   - Queries correct
   - Transactions work

3. **Integration Testing**
   - External API calls
   - Third-party services

#### ❌ SKIP These Tests:
- UI testing (no UI)
- Browser testing (no browser)
- Responsive testing (no frontend)

---

### For Type E: JavaScript Library

#### ✅ REQUIRED Tests:
1. **Unit Tests**
   - Each function individually
   - Input validation
   - Output verification
   - Edge cases

2. **Integration Tests**
   - Functions work together
   - Module composition

3. **Browser/Node Testing** (if applicable)
   - Test in target environment
   - Check compatibility

#### ❌ SKIP These Tests:
- UI testing (no UI)
- API tests (not an API)
- Database tests (no database)

---

### For Type F: CLI Tool

#### ✅ REQUIRED Tests:
1. **Command Testing**
   - All commands work
   - Flags/options work
   - Help text displays

2. **Input/Output Testing**
   - Various input formats
   - Output verification
   - File operations

3. **Error Handling**
   - Invalid commands
   - Missing arguments
   - Permission errors

#### ❌ SKIP These Tests:
- Browser testing (not browser-based)
- UI testing (CLI only)
- API tests (unless calling APIs)

---

## Step 3: Testing Method Selection

### When to Use Manual Testing
**Use for:**
- ✅ Single-file HTML apps
- ✅ Simple multi-page sites
- ✅ Quick validation
- ✅ Visual/UX verification
- ✅ Print layout checks

**How:**
- Open in browser(s)
- Click through interactions
- Test with real data
- Take screenshots as evidence

---

### When to Use Automated Tests
**Use for:**
- ✅ JavaScript libraries (unit tests)
- ✅ Complex full-stack apps
- ✅ APIs with many endpoints
- ✅ Regression-prone areas

**Tools to consider:**
- Unit tests: Vitest, Jest
- API tests: Supertest, Postman
- E2E: Playwright (only if justified)

---

### When to Use E2E Frameworks
**ONLY use E2E frameworks when:**
- ✅ Complex user workflows
- ✅ Multiple interconnected features
- ✅ High-value regression testing
- ✅ Full-stack app with critical paths

**DON'T use E2E for:**
- ❌ Single-file HTML apps
- ❌ Simple websites
- ❌ Static pages
- ❌ Libraries/APIs

---

## Step 4: Evidence Collection

### Always Collect:
1. **Screenshots** for visual tests
2. **Console logs** for error checking
3. **PDF outputs** for print tests
4. **Test results** for automated tests
5. **Browser versions** tested

### Format:
```markdown
## Test Evidence

### Manual Browser Test
- Browser: Chrome 120
- Result: ✅ Pass
- Evidence: [Screenshot attached]

### Sample File Test
- File: sample.ofx
- Expected: Parse 50 transactions
- Actual: Parsed 50 transactions ✅
- Evidence: Console output shows all data
```

---

## Decision Flow Chart

```
START
  ↓
Does it have a user interface?
  ↓                    ↓
 YES                  NO
  ↓                    ↓
Is it single file?   Is it an API?
  ↓         ↓          ↓         ↓
YES       NO         YES       NO
  ↓         ↓          ↓         ↓
Type A   Type B/C   Type D   Type E/F
  ↓         ↓          ↓         ↓
Manual   Manual +   API      Unit
Browser  E2E (if   Tests    Tests
Tests    complex)
```

---

## Common Mistakes to Avoid

### ❌ Mistake 1: Testing Non-Existent Features
**Bad:** "Running API tests" on static HTML
**Fix:** Identify project type first

### ❌ Mistake 2: Over-Testing Simple Projects
**Bad:** E2E framework for calculator app
**Fix:** Use manual testing for simple projects

### ❌ Mistake 3: Under-Testing Complex Projects
**Bad:** Only manual tests for full-stack app
**Fix:** Add API and integration tests

### ❌ Mistake 4: No Evidence
**Bad:** "Tests pass" (no proof)
**Fix:** Collect screenshots, logs, results

### ❌ Mistake 5: Wrong Test Environment
**Bad:** Browser testing for CLI tool
**Fix:** Test in target environment

---

## Quick Reference Table

| Project Type | Manual | Unit | API | E2E | Print |
|--------------|--------|------|-----|-----|-------|
| Single HTML  | ✅     | ❌   | ❌  | ❌  | ⚠️    |
| Frontend     | ✅     | ⚠️   | ⚠️  | ❌  | ⚠️    |
| Full-Stack   | ✅     | ✅   | ✅  | ⚠️  | ⚠️    |
| Backend API  | ❌     | ✅   | ✅  | ❌  | ❌    |
| Library      | ❌     | ✅   | ❌  | ❌  | ❌    |
| CLI          | ❌     | ✅   | ⚠️  | ❌  | ❌    |

**Legend:**
- ✅ Required
- ⚠️ If applicable
- ❌ Not applicable

---

## Final Checklist

Before reporting test results:

- [ ] Project type correctly identified
- [ ] Only relevant tests executed
- [ ] No irrelevant tests run
- [ ] Evidence collected for all tests
- [ ] Browser versions documented (if applicable)
- [ ] Sample files tested (if applicable)
- [ ] Edge cases from Architect's spec verified
- [ ] Success criteria from Architect checked
