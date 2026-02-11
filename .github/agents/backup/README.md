# Multi-Agent Starter Template

A clean, intelligent agent system for GitHub Copilot that eliminates common problems: assumptions, feature creep, inappropriate testing, and incomplete work.

---

## Philosophy

**Ask, Don't Assume**

This template fixes the #1 problem with multi-agent systems: **assumptions**.

Every agent follows one core rule:
> **When in doubt → ASK the user. Never guess.**

This eliminates:
- ❌ Guessing at requirements
- ❌ Adding unrequested features
- ❌ Arbitrary file size limits
- ❌ Inappropriate testing (API tests for static HTML)
- ❌ Incomplete handoffs between agents

---

## Agent System

### 5 Specialized Agents

```
Architect → UI/UX Designer → Implementer → Validator → Debugger
```

#### 1. **Architect** (Planning & Research)
- Analyzes requirements and samples
- Designs data structures (JSON schemas)
- Creates algorithms (pseudocode only)
- Writes Architecture Decision Records (ADRs)
- **NO CODE** - only specs and documentation

#### 2. **UI/UX Designer** (Visual Design & CSS)
- Expert CSS developer
- Creates design systems
- Builds dark/light themes
- Ensures accessibility
- Creates print stylesheets
- **Only activated if project has UI**

#### 3. **Implementer** (Vanilla JavaScript Expert)
- Implements Architect's algorithms exactly
- Uses UI/UX Designer's CSS unchanged
- Writes clean, commented code
- Follows specs precisely
- **Prefers vanilla JS** - frameworks only if requested

#### 4. **Validator** (Context-Aware Testing)
- Tests **intelligently** based on project type
- Manual testing for simple projects
- Automated testing for complex projects
- **NO inappropriate tests** (e.g., API tests for static HTML)
- Verifies success criteria

#### 5. **Debugger** (Surgical Bug Fixes)
- **Only activated when bugs found**
- Reproduces bugs exactly
- Makes minimal, targeted fixes
- Tests for regressions
- **NO refactoring** while debugging

---

## Key Features

### ✅ Anti-Assumption Guardrails
Every agent must ask when uncertain instead of guessing.

### ✅ No Arbitrary Restrictions
No file size limits or scaling assumptions unless user specifies.

### ✅ Context-Aware Testing
Validator chooses appropriate tests based on project type:
- Single-file HTML → Manual browser tests
- Full-stack app → UI + API + integration tests
- JavaScript library → Unit tests only

### ✅ Stay In Your Lane
- Architect doesn't write code
- UI/UX Designer doesn't write JavaScript
- Implementer doesn't change architecture
- Validator doesn't fix bugs
- Debugger doesn't add features

### ✅ Complete Handoffs
Every agent must complete their checklist before handing off.

---

## Quick Start

### Step 1: Use the Template

1. Click "Use this template" on GitHub
2. Create your new repository
3. Clone to your machine

### Step 2: Start with Architect

Open GitHub Copilot chat and select the **Architect** agent.

Send your project requirements:

```
I need to build [description of project].

Requirements:
- [Feature 1]
- [Feature 2]
- [Feature 3]

Constraints:
- [Any specific constraints]

Preferences:
- [Technology preferences if any]
```

### Step 3: Follow the Pipeline

Architect will:
1. Ask clarifying questions
2. Analyze requirements
3. Create specifications
4. Hand off to next agent

Follow the handoff prompts as they appear.

### Step 4: Review Deliverables

Each agent creates:
- **Architect:** Specs, ADRs in `.github/adrs/`
- **UI/UX Designer:** Complete CSS and design docs
- **Implementer:** Working code
- **Validator:** Test results and bug reports
- **Debugger:** Bug fixes (if needed)

---

## Project Type Examples

### Single-File HTML App
```
Architect → UI/UX Designer → Implementer → Validator
```
**Example:** File converter, calculator, data viewer

**Tests:** Manual browser testing, no API tests

---

### JavaScript Library
```
Architect → Implementer → Validator
```
**Example:** Utility functions, helper library

**Tests:** Unit tests, no UI tests

**Skip:** UI/UX Designer (no interface)

---

### Full-Stack Web App
```
Architect → UI/UX Designer → Implementer → Validator
```
**Example:** Task manager, blog platform

**Tests:** UI + API + integration + database

---

## File Structure

```
.github/
├── agents/               # Agent definitions
│   ├── architect.agent.md
│   ├── ui-ux-designer.agent.md
│   ├── implementer.agent.md
│   ├── validator.agent.md
│   └── debugger.agent.md
├── adrs/                 # Architecture Decision Records
│   └── ADR-XXX-title.md
├── instructions/         # Path-specific instructions
└── copilot-instructions.md  # Core rules

docs/                     # Generated documentation
WORKFLOW.md              # Detailed workflow guide
README.md                # This file
```

---

## Core Rules (All Agents)

### 🚨 Rule #1: When In Doubt → ASK

Never:
- Guess at requirements
- Assume user preferences
- Fill in gaps yourself
- Use "best practices" without context

Always:
- Ask specific questions
- Present options with tradeoffs
- Request clarification
- Wait for user decision

### 🚨 Rule #2: No Arbitrary Restrictions

Never add:
- File size limits (unless user specifies)
- "Typically X users" assumptions
- "Usually X KB" assumptions
- Scaling restrictions

Always:
- Support any scale unless told otherwise
- Process files of any size
- No artificial constraints

### 🚨 Rule #3: Complete Before Handoff

Check ALL handoff gate items:
- [ ] All deliverables complete
- [ ] All decisions documented
- [ ] No unknowns or assumptions
- [ ] Next agent has everything needed

### 🚨 Rule #4: Context-Aware Actions

- Architect: NO code, only specs
- Implementer: NO architecture changes
- Validator: Tests appropriate for project type
- Debugger: NO refactoring, only bug fixes

---

## Success Criteria

A successful workflow has:

✅ **Clear Communication**
- Agents ask when unclear
- Handoffs are detailed and complete
- No assumptions made

✅ **Appropriate Testing**
- Tests match project type
- No irrelevant tests run
- Evidence collected for all tests

✅ **Clean Deliverables**
- Code matches specifications exactly
- No extra features added
- Documentation complete

✅ **Working Result**
- Meets all requirements
- All tests pass
- User can use immediately

---

## Troubleshooting

### Agent making assumptions
**Fix:** "STOP. Read core rule: When in doubt, ASK."

### Agent adding features
**Fix:** "Implement ONLY what's in requirements. No additions."

### Inappropriate tests running
**Fix:** "Determine project type. Choose context-appropriate tests."

### Incomplete handoff
**Fix:** "Check your handoff gate. Complete ALL items before handing off."

---

## Documentation

- **[WORKFLOW.md](WORKFLOW.md)** - Detailed workflow and pipeline
- **[.github/copilot-instructions.md](.github/copilot-instructions.md)** - Core rules
- **[.github/agents/](.github/agents/)** - Individual agent specifications

---

## ADR Template

Architecture Decision Records go in `.github/adrs/`:

```markdown
# ADR-XXX: [Decision Title]

**Status:** Accepted
**Date:** YYYY-MM-DD
**Deciders:** [User + Architect]

## Context
[Problem and constraints]

## Decision
[What we decided]

## Alternatives Considered
1. Option A - Why rejected
2. Option B - Why rejected

## Consequences
- Positive: [benefits]
- Negative: [tradeoffs]
- Risks: [with mitigation]
```

---

## Contributing

This is a template repository. Fork it, customize it, improve it.

Key principles to maintain:
1. Ask, don't assume (core rule)
2. Context-aware actions (right tool for the job)
3. Complete handoffs (no incomplete work)
4. Stay in your lane (clear responsibilities)

---

## Why This Template?

**Problem:** Traditional multi-agent systems fail because:
- Agents guess instead of asking
- Inappropriate tests waste time
- Feature creep adds complexity
- Incomplete handoffs cause confusion
- Agents overstep their roles

**Solution:** This template enforces:
- "Ask when uncertain" as core rule
- Context-aware testing decisions
- Strict scope boundaries
- Complete handoff requirements
- Clear role separation

**Result:** 
- Faster development (less debugging unnecessary tests)
- Better output (matches actual requirements)
- Clearer communication (no assumptions)
- Cleaner code (no feature creep)

---

## License

MIT License - Use freely, modify as needed.

---

## Version

**v2.0** - Complete rewrite focusing on:
- Anti-assumption guardrails
- Context-aware testing
- Vanilla JavaScript preference
- Minimal, focused agents
- Clear handoff requirements
