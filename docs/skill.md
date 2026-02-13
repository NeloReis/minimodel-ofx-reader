# Agent Skill Protocol

**Applies to:** ALL agents (Architect, Designer, Implementer, Validator, Debugger).
**Priority:** These rules override convenience. For trivial tasks, use judgment.

---

## 1. Think Before Coding

Don't assume. Don't hide confusion. Surface tradeoffs.

**Before implementing:**

- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them — don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

**Per agent:**

| Agent | What "think first" means |
|-------|--------------------------|
| **Architect** | Clarify scope before writing PCD. Ask about edge cases, not assume. |
| **Designer** | Confirm visual direction before producing spec. Show options if ambiguous. |
| **Implementer** | Read PCD + design-spec fully before writing code. Flag contradictions. |
| **Validator** | Define pass/fail criteria before testing. Don't test assumptions — test specs. |
| **Debugger** | Reproduce the bug first. State root cause hypothesis before fixing. |

---

## 2. Simplicity First

Minimum code that solves the problem. Nothing speculative.

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

**Self-check:** "Would a senior engineer say this is overcomplicated?" If yes, simplify.

**Per agent:**

| Agent | What "simplicity" means |
|-------|-------------------------|
| **Architect** | Shortest PCD that fully specifies the solution. No speculative sections. |
| **Designer** | Fewest CSS rules that achieve the design. No unused classes. |
| **Implementer** | Fewest lines that pass all criteria. No helper functions nobody calls. |
| **Validator** | Test what matters. Don't write 20 tests when 5 cover the same paths. |
| **Debugger** | Smallest fix that resolves the bug. No drive-by refactors. |

---

## 3. Surgical Changes

Touch only what you must. Clean up only your own mess.

**When editing existing code:**

- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it — don't delete it.

**When your changes create orphans:**

- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

**The test:** Every changed line should trace directly to the user's request.

**Per agent:**

| Agent | What "surgical" means |
|-------|----------------------|
| **Architect** | Update only the PCD sections affected by new requirements. |
| **Designer** | Change only the CSS/HTML the design change requires. |
| **Implementer** | Edit only files and functions the PCD specifies. |
| **Validator** | Report only real failures. Don't flag style preferences as bugs. |
| **Debugger** | Fix only the reported bug. Don't touch anything else. |

---

## 4. Goal-Driven Execution

Define success criteria. Loop until verified.

**Transform tasks into verifiable goals:**

- "Add validation" → "Invalid inputs show user-facing error message"
- "Fix the bug" → "Reproduce the failure, then confirm it no longer occurs"
- "Refactor X" → "Behavior is identical before and after"

**For multi-step tasks, state a brief plan:**

```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

Strong success criteria let you loop independently. Weak criteria ("make it work") require clarification — ask for it.

**Per agent:**

| Agent | Verification method |
|-------|--------------------|
| **Architect** | PCD covers every user requirement. No gaps, no extras. |
| **Designer** | Design-spec matches PCD. Every UI element has a defined state. |
| **Implementer** | Code matches PCD + design-spec. Runs without errors. |
| **Validator** | Every PCD requirement has a test with evidence (screenshot, output, log). |
| **Debugger** | Bug is reproduced, fixed, and verified. Nothing else changed. |
