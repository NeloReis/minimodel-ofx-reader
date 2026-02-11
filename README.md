# Multi-Agent Starter Template

A GitHub Copilot agent system with 5 specialized agents, file-based handoffs, and opinionated defaults for building web applications.

---

## What This Is

A template repository with custom Copilot agents that follow a structured pipeline:

```
Architect → Designer → Implementer → Validator → Debugger
```

Each agent writes its output to a file in `docs/`. The next agent reads that file. No context is lost between handoffs.

---

## Agents

| Agent | Role | Writes |
|-------|------|--------|
| **Architect** | Research, planning, specs | `docs/pcd.md` |
| **Designer** | CSS, HTML structure, visual design | `docs/design-spec.md` |
| **Implementer** | Working code from specs | Source files |
| **Validator** | Testing with evidence | `docs/test-report.md` |
| **Debugger** | Minimal bug fixes | `docs/debug-report.md` |

---

## How To Use

### 1. Use This Template

Click "Use this template" on GitHub → create your repo → clone it.

### 2. Open in VS Code

Open the repo in VS Code with GitHub Copilot enabled.

### 3. Start with Architect

Select the **Architect** agent in Copilot Chat. Describe what you want to build:

```
I need a single-page app that converts CSV files to JSON.
It should handle files of any size, show a preview, and let users download the result.
```

### 4. Follow the Handoffs

After each agent finishes, a handoff button appears. Click it to move to the next agent. Each handoff pre-fills a prompt telling the next agent where to find its input files.

---

## File Structure

```
.github/
├── agents/                  # Agent definitions
│   ├── architect.agent.md
│   ├── designer.agent.md
│   ├── implementer.agent.md
│   ├── validator.agent.md
│   └── debugger.agent.md
└── copilot-instructions.md  # Shared rules for all agents

docs/
├── design-system.md         # CSS foundation (reference, read-only)
├── coding-patterns.md       # JS patterns (reference, read-only)
├── pcd.md                   # ← Generated per project
├── design-spec.md           # ← Generated per project
├── test-report.md           # ← Generated per project
└── debug-report.md          # ← Generated per project

WORKFLOW.md                  # Detailed workflow guide
README.md                    # This file
```

---

## Defaults (Opinionated)

These are the defaults. Users can override any of them during the Architect phase.

- **Code:** Vanilla JS, no frameworks, no build tools
- **Style:** Dark theme with glow effects (customizable accent color)
- **HTML:** Semantic HTML5, single-file for simple projects
- **Safety:** `escapeHtml()` for all user data, try-catch on all async, no silent failures
- **Variables:** `const`/`let` only, never `var`

---

## Key Design Decisions

### File-based handoffs
Each agent writes to `docs/`. The next agent reads from `docs/`. This eliminates the #1 failure mode of multi-agent systems: context loss between agents.

### Reference files vs. inline instructions
CSS patterns live in `docs/design-system.md`. JS patterns live in `docs/coding-patterns.md`. Agents link to these files instead of embedding hundreds of lines in their instructions. This keeps agents short and focused.

### Handoff buttons with pre-filled prompts
Each handoff button tells the next agent exactly which file to read. No guessing, no "check the conversation history."

### Evidence-based testing
Validator must provide exact steps, expected result, actual result, and evidence for every test. "Tests pass" without proof is forbidden.

---

## Customizing

### Change the design system
Edit `docs/design-system.md` — the Designer agent reads this as its CSS foundation.

### Change coding patterns
Edit `docs/coding-patterns.md` — the Implementer agent reads this for utility functions.

### Change shared rules
Edit `.github/copilot-instructions.md` — these rules apply to all agents.

### Add/remove agents
Add/remove `.agent.md` files in `.github/agents/`. Update handoff references in other agents to match.

---

## Documentation

- [WORKFLOW.md](WORKFLOW.md) — Detailed pipeline and workflow guide
- [.github/copilot-instructions.md](.github/copilot-instructions.md) — Shared rules
- [docs/design-system.md](docs/design-system.md) — CSS design system reference
- [docs/coding-patterns.md](docs/coding-patterns.md) — Defensive JS patterns

---

## License

MIT
