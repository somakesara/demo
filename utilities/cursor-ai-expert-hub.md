# Cursor AI Expert Hub

> A practical reference for Cursor power users. Covers rules, context, every agent surface, and the workflows that tie them together.
>
> **Snapshot:** June 12, 2026 — Cursor 3.7. Tool-specific stats and version-pinned features will drift; verify at [cursor.com/changelog](https://cursor.com/changelog) before quoting them externally.
>
> **Out of scope:** Enterprise SSO, billing, VS Code parity details, deployment specifics.

---

## Table of Contents

1. [Rules (.cursor/rules/*.mdc)](#1-rules-cursorrulesmdc)
2. [Memory & Project Context](#2-memory--project-context)
3. [Surfaces: Cmd+K vs Composer vs Tab](#3-surfaces-cmdk-vs-composer-vs-tab)
4. [Background & Parallel Agents](#4-background--parallel-agents)
5. [Bugbot (PR Review)](#5-bugbot-pr-review)
6. [Canvas & Design Mode](#6-canvas--design-mode)
7. [Cursor SDK & Custom Tools](#7-cursor-sdk--custom-tools)
8. [Mobile & CLI](#8-mobile--cli)
9. [MCP Servers](#9-mcp-servers)
10. [Context Management](#10-context-management)
11. [Prompt Engineering](#11-prompt-engineering)
12. [End-to-End Workflows](#12-end-to-end-workflows)
13. [Keyboard Shortcuts](#13-keyboard-shortcuts)
14. [Team Setup & Closing Principles](#14-team-setup--closing-principles)

---

## 1. Rules (.cursor/rules/*.mdc)

Modern Cursor rules live in `.cursor/rules/` as `.mdc` files. The legacy single-file `.cursorrules` at the repo root is **deprecated** and is **silently ignored by Composer/Agent mode** — migrate if you use agentic workflows.

> ### ⚠ The Token Tax
> Always-apply rules load on **every** request — chat, Composer, Agent. Bloated `alwaysApply` rules are the #1 way users sabotage themselves in Cursor. **Keep always-apply rules under 200 words.** Push everything else into scoped or agent-requested rules. Cursor 3.7's Context Usage Report (see §10) makes this measurable — use it.

### The Four Activation Modes

| Mode | Frontmatter | When it fires |
|---|---|---|
| **Always Apply** | `alwaysApply: true` | Every interaction |
| **Auto Attached** | `globs: ["..."]` | When matching files are in context |
| **Agent Requested** | `description: "..."` | Agent decides based on description match |
| **Manual** | none of the above | Only when you `@rule-name` it |

### Always-Apply Base Rule

```markdown
---
description: Base coding standards
alwaysApply: true
---

# Project Rules

## Stack
- TypeScript strict mode
- React 18 + Tailwind CSS
- Vitest for testing

## Code style
- Named exports only
- No default exports
- Prefer const over let
- Always handle errors explicitly
```

### Auto-Attached Scoped Rule

```markdown
---
description: API route conventions
globs: ["src/app/api/**/*.ts"]
---

# API Route Rules

- Validate input with Zod
- Return typed responses
- Use HTTP status codes correctly
- Log errors with structured format
- Never expose stack traces to client
```

### Agent-Requested Rule

```markdown
---
description: Apply when writing or modifying database migrations
---

# Migration Rules

- One concern per migration
- Always reversible (write both up and down)
- Use transactions for multi-statement migrations
- Add indexes in separate migrations from schema changes
```

### Recommended Rule Set (5–8 files)

```
.cursor/rules/
  base.mdc              always-apply, <200 words
  typescript.mdc        globs: ["**/*.{ts,tsx}"]
  react.mdc             globs: ["**/*.tsx"]
  api.mdc               globs: ["src/app/api/**"]
  tests.mdc             globs: ["**/*.{test,spec}.*"]
  database.mdc          agent-requested, for migrations/queries
  security.mdc          always-apply, OWASP basics
```

### AGENTS.md (Cross-Tool Standard)

If your team also uses Claude Code, Aider, Copilot, or Codex, you can adopt `AGENTS.md` at the repo root as a shared baseline. Cursor reads it.

**Important nuance:** `AGENTS.md` is a **fallback**, not an equivalent. When both `AGENTS.md` and `.cursor/rules/` exist, `.cursor/rules/` takes precedence in Cursor, and `AGENTS.md` doesn't get activation modes. Dual-stack pattern: put cross-tool basics in `AGENTS.md`, put Cursor-specific scoping in `.cursor/rules/`.

---

## 2. Memory & Project Context

### What replaced Notepads

Notepads are gone. Replacements:
- **`.cursor/rules/`** — persistent, scoped, frontmatter-driven
- **`AGENTS.md`** — cross-tool baseline (see §1)
- **@-mentioned markdown docs** — keep architecture/spec docs in your repo and `@mention` them
- **Skills** (`.cursor/skills/`) — task-scoped instructions (parallel to the SKILL.md pattern in other tools)

### Recommended Repo Layout

```
docs/
  ARCHITECTURE.md       @docs/ARCHITECTURE.md in prompts
  API_CONTRACTS.md
  TECH_DEBT.md
  PATTERNS.md           "how we do X here" examples
.cursor/
  rules/                scoped activation rules
  skills/               task-scoped instruction sets (optional)
  mcp.json              MCP server config
AGENTS.md               cross-tool baseline (optional)
.cursorignore           exclude from indexing
```

### Architecture Doc Template

```markdown
# Project Architecture

## Stack
- Frontend: Next.js 14 (App Router)
- Backend: tRPC + Prisma
- DB: PostgreSQL (Supabase)
- Auth: NextAuth.js
- Styling: Tailwind CSS + shadcn/ui

## Key Patterns
- Server Components by default; Client Components only when needed
- All DB access through services in `src/services/`
- Zod schemas in `src/schemas/`, shared FE/BE

## Directory Structure
src/
  app/          Next.js routes
  components/   shared UI
  services/     business logic
  schemas/      Zod validation
  db/           Prisma client + queries
```

### What Goes Where

| Information type | Best location |
|---|---|
| Always-on standards (<200 words) | `.cursor/rules/base.mdc` |
| File-type specific rules | `.cursor/rules/*.mdc` with globs |
| Architecture decisions | `docs/ARCHITECTURE.md` + `@mention` |
| Cross-tool basics | `AGENTS.md` at repo root |
| One-off session context | Inline `@file` in chat |
| Task-scoped procedures | `.cursor/skills/<task>/SKILL.md` |

---

## 3. Surfaces: Cmd+K vs Composer vs Tab

Three distinct surfaces, often confused. They use different models, fire in different contexts, and serve different needs.

| Surface | Shortcut | Model | Best for |
|---|---|---|---|
| **Inline Edit** | `Cmd+K` | Edit-specialist | Surgical single-spot edits in selected code |
| **Composer** | `Cmd+I` | Composer 2.5 or selected frontier | Multi-file work, agentic loops, terminal access |
| **Tab** | `Tab` to accept | Sonic (in-house, low-latency) | Inline autocomplete as you type |

### Composer Modes (Ask vs Agent)

Composer toggles between two modes (cycle with `Shift+Tab` inside Composer):

- **Ask** — read-only Q&A about code, planning, no edits
- **Agent** — multi-file edits, runs terminal commands, iterates on its own output

Use **Ask** first to plan, then switch to **Agent** to execute. This separation reduces wasted work from misunderstood requirements.

### Composer 2.5

Cursor's in-house model, optimized for long-horizon multi-file edits. Best for:
- File-tree scale refactors
- Coordinated changes across many files
- Routine agentic work where you'd otherwise burn frontier-model credits

Frontier models (Claude Sonnet 4.7/Opus 4.8, GPT-5.5, Gemini 3.1 Pro, Grok 4.3) are still selectable per task via the model picker.

### Tab (Sonic)

Tab is operationally separate from Composer/Agent — a lightweight in-house model trained on edit patterns, using a vector index of your codebase. Key points:

- **Unlimited on Pro tier** — only Composer/Agent burns credits
- **Rules do not affect Tab** — Tab quality scales with index quality, not rule quality
- Narrow `.cursorignore` if Tab feels noisy
- Accept word-by-word with `Cmd+→`

### Which surface to use

| Task | Surface |
|---|---|
| "Rename this variable everywhere" | Cmd+K (or refactor command) |
| "Add error handling to this function" | Cmd+K |
| "Explain this module" | Composer Ask |
| "Build a CRUD feature with tests" | Composer Agent |
| "Refactor auth across 12 files" | Composer Agent + Composer 2.5 |
| Routine typing of known patterns | Let Tab handle it |

---

## 4. Background & Parallel Agents

Cloud-hosted agents that run in isolated VMs. Work happens off your machine; results report back asynchronously.

### Background Agents

- Run in sandboxed cloud VMs with terminal, browser, and desktop access
- Can work across multiple repos in parallel
- Triggered from IDE, web ([cursor.com/agents](https://cursor.com/agents)), CLI, or mobile
- Pro tier and above; metered separately

### Parallel Agents

Run multiple agents simultaneously on the same task using git worktrees for isolation. Two common patterns:

```
Pattern 1 — Decomposition:
  Agent 1: Build pricing page
  Agent 2: Write tests for pricing logic
  Agent 3: Refactor auth to support trial flow

Pattern 2 — Best-of-N:
  Spawn N agents on the SAME task
  Review diffs side by side in the Agents Window
  Merge the cleanest; discard the rest
```

Use `/multitask` to dispatch async subagents from inside Composer instead of queueing.

### Loop Mode

`/loop` runs a long-lived local agent on a recurring schedule until a goal is met or you stop it. Use for: continuous integration of small tasks, monitoring-then-fixing flows, repeated audits.

### Background vs Foreground

| Task | Background | Foreground |
|---|---|---|
| Big refactor (20+ min) | ✓ | |
| Quick feature you'll review immediately | | ✓ |
| Multi-repo coordinated change | ✓ | |
| Interactive debugging | | ✓ |
| Overnight batch work | ✓ | |
| Recurring scheduled task | ✓ (via `/loop`) | |

---

## 5. Bugbot (PR Review)

Cursor's automated PR reviewer on GitHub and GitLab.

- Powered by Composer 2.5
- Reviews PRs for bugs and production-breaking issues
- **Intentionally ignores style/formatting** — use a linter for that
- Configurable per-repo (which models can run, ignored paths, auto-fix on/off)

> *Performance metrics (speed, cost, bug-find rate) are vendor-reported and shift release-to-release. Check the [Cursor changelog](https://cursor.com/changelog) for current numbers if you're benchmarking.*

### Local Pre-Push Review (Cursor 3.7+)

```
/review                 choose which agents to run
/review-bugbot          run only Bugbot
/review-security        run only Security Review
```

If you `/review` locally and push a PR with the same diff, Bugbot recognizes it and skips re-reviewing.

### Configuration

```yaml
# .cursor/bugbot.yml (or via cursor.com/dashboard)

review_scope: incremental    # only new changes since last review
model_block_list: []         # restrict to approved models
auto_fix: true               # spawn agent to fix flagged issues
ignore_paths:
  - "**/*.test.ts"
  - "scripts/legacy/**"
```

### Autofix

When autofix is enabled, Bugbot spawns a Background Agent (see §4) that diagnoses the flagged issue, writes a fix, and either commits to the same PR or opens a follow-up. Treat its output like a junior PR — review before merging.

---

## 6. Canvas & Design Mode

Canvases are interactive artifacts agents create — dashboards, reports, internal tools, custom interfaces. Distinct from regular file edits.

### What Canvases Are For

- Live dashboards showing data from your codebase or services
- Internal tools your team can use directly (forms, viewers, runners)
- Visual reports (e.g., the Context Usage Report in §10)
- Shareable artifacts via Cursor Dashboard (Pro, Teams, Enterprise)

Canvas buttons can be embedded that trigger specific prompts on click — useful for repeatable team workflows.

### Design Mode

For UI-heavy work in canvases or live browser previews, Design Mode lets you **point at, draw on, or annotate UI elements directly** to guide agent edits, instead of describing changes in text.

Workflow:
1. Open canvas or browser preview
2. Activate Design Mode
3. Click/draw on the element you want changed
4. Add a short text annotation ("more contrast," "align right")
5. Agent edits the underlying code

This is the biggest UX shift of 2026 for frontend devs. Strongest when paired with a design system already in your codebase — the agent has anchors to work from.

---

## 7. Cursor SDK & Custom Tools

For teams building their own agents on top of Cursor's infrastructure.

### What the SDK Provides

```bash
npm install @cursor/sdk
# or
pip install cursor-sdk
```

- **Programmatic agent launch** — dispatch agents from your own scripts/services
- **Custom tools** — define your own tools agents can call, beyond MCP
- **Custom stores** — persistent state stores agents read/write
- **Auto-review hooks** — wire Bugbot-like review into your own pipelines
- **Run metadata** — `requestId` and other fields for tracing/observability

### When to Reach for the SDK

| Use case | SDK fits |
|---|---|
| Trigger agents from CI/CD on specific events | ✓ |
| Build internal "fix-this-class-of-bug" automation | ✓ |
| Custom dashboards showing agent activity | ✓ |
| Cross-repo orchestration with company-specific logic | ✓ |
| Daily coding work | ✗ — use the IDE |

If your team has 10+ engineers and recurring patterns of agent work, the SDK is where you turn that pattern into infrastructure.

---

## 8. Mobile & CLI

Cursor is no longer a desktop-only product.

### Mobile (iOS + Android)

- Dispatch and monitor Background Agents from your phone
- Review diffs, approve PRs, chat with agents
- Results sync back to your IDE when you return

Common pattern: dispatch a Background Agent before a meeting, glance at the diff during a break, approve or refine when you're back at the desk.

### CLI (Headless)

A command-line interface for running Cursor agents without the IDE. Useful for:
- CI/CD integration (run an agent as a build step)
- Server-side scripting (cron-triggered audits)
- SSH-from-anywhere coding when the IDE isn't available
- Programmatic workflows alongside the SDK

CLI and SDK overlap; SDK gives you a programming interface, CLI gives you a shell interface.

---

## 9. MCP Servers

MCP (Model Context Protocol) connects Cursor's agent to external tools — databases, APIs, browsers, ticketing systems.

> Cursor caps the number of tools available per session to keep model attention focused. The exact limit varies by version and model — trim unused servers if the agent seems unfocused or starts ignoring tools.

### Setup: `.cursor/mcp.json`

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/path/to/project"]
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": { "GITHUB_TOKEN": "ghp_your_token_here" }
    },
    "postgres": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres", "postgresql://localhost/mydb"]
    },
    "puppeteer": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-puppeteer"]
    }
  }
}
```

### Common MCP Servers

| Server | Source | Use case |
|---|---|---|
| Filesystem | `@modelcontextprotocol/server-filesystem` | File operations |
| GitHub | `@modelcontextprotocol/server-github` | Issues, PRs, repos |
| PostgreSQL | `@modelcontextprotocol/server-postgres` | Direct SQL queries |
| Puppeteer | `@modelcontextprotocol/server-puppeteer` | Browser automation |
| Linear | Hosted URL (official) | Issue tracking |
| Sentry | Hosted URL (official) | Error context |
| Jira | Native Cursor integration | Tickets → PRs |

> Verify package names and configuration against the official MCP server registry — many community "MCP servers" are unofficial wrappers. Most SaaS integrations (Linear, Sentry) are URL-hosted, not npm packages.

### Agent + MCP Combos

- **Postgres MCP** → agent writes, runs, and validates migrations (see §5 for autofix combo)
- **GitHub MCP** → agent creates PRs with descriptions, labels, reviewers
- **Puppeteer MCP** → agent drives browser to run E2E tests and capture screenshots
- **Jira (native)** → end-to-end ticket-to-PR pipeline

---

## 10. Context Management

### @-mention Reference

```
@filename.ts        attach specific file
@src/folder/        attach entire directory
@codebase           semantic search across indexed code
@web                live web search
@docs               search indexed documentation
@git                git history / current diff
@terminal           last terminal output
@rule-name          manually attach a rule
#file               pin file to context bar
```

### `.cursorignore`

```gitignore
# .cursorignore — exclude from index
node_modules/
dist/
.next/
build/
coverage/
*.generated.ts
*.min.js
*.min.css
.env*
prisma/migrations/
```

### Context Loading by Task

| Task | Approach |
|---|---|
| Build new feature | `@docs/ARCHITECTURE.md` + `@src/types/` + related `@folder/` |
| Fix a bug | `@terminal` (error) + `@codebase` search + specific `@file` |
| Code review | `@git` diff + `@src/` patterns |
| Refactor | `@target-file` + `@similar-file` (reference) |
| Write tests | `@source-file` + `@existing.test.ts` |
| Add migration | `@schema.prisma` + `@prisma/migrations/` |

### Context Usage Report (Cursor 3.7+)

Interactive canvas showing token breakdown across system prompt, tool definitions, rules, and skills. Use it to:
- Catch bloated always-apply rules eating into your budget
- See which tools/MCP servers consume the most tokens
- Diagnose why a long session is degrading (you've blown the window)

This is the right tool for keeping the **token tax** from §1 honest.

> **Context compaction caveat:** In long Composer sessions, Cursor compacts older context — and rules can silently drop. For critical tasks, reinforce with explicit `@rule-name` mentions.

### Indexing External Docs

**Settings → Indexing & Docs** (path varies by version) → add URLs:
- `https://nextjs.org/docs`
- `https://www.prisma.io/docs`
- `https://ui.shadcn.com/docs`
- `https://trpc.io/docs`

Reference with `@docs` in chat.

---

## 11. Prompt Engineering

### CRAFT Framework

```
C — Context:  What exists, what's the current state
R — Role:     "You are a senior [X] engineer"
A — Action:   Exact task
F — Format:   Expected output structure
T — Tests:    How to verify it worked
```

**Example:**

```
Context: @src/features/auth/ uses NextAuth with JWT.
         @src/features/user/ shows our service layer pattern.

Role: Senior Next.js engineer who prioritizes type safety.

Action: Add email verification — send token on signup,
        verify on /api/auth/verify?token=

Format: Files follow @src/features/user/.
        Zod schema in src/schemas/auth.ts.
        Service in src/services/auth.ts.

Tests: src/features/auth/auth.test.ts.
       Done when: all tests pass, no TS errors.
```

### Spec-First Prompting

```
Step 1: Composer Ask mode →
        "Write a markdown spec for [feature] — no code yet.
         Include: data model, API endpoints, edge cases, acceptance criteria."

Step 2: Review and refine the spec

Step 3: Composer Agent mode →
        "Implement @spec.md following patterns in @src/.
         Start with types, then service, then API, then tests."
```

The Ask/Agent split in Composer maps perfectly onto spec-first work — see §3.

### Reusable Templates

**Code Review:**
```
Review @git diff for:
1. Bugs and edge cases not handled
2. Performance (N+1, missing indexes, unnecessary re-renders)
3. Security (missing auth, unvalidated input)
4. Missing tests
5. Deviations from @src/ patterns

Format each issue: [SEVERITY] File:line — description + fix
```

**Debugging:**
```
Error from @terminal: [paste]

1. Identify root cause — search @codebase
2. Explain why this happens
3. Propose fix + regression test
```

**Documentation:**
```
Document @src/services/payment.ts:
- JSDoc for every export
- README section for the payment flow
- Update CHANGELOG.md
Follow style in @src/services/user.ts
```

---

## 12. End-to-End Workflows

These workflows span multiple sections — they're here because each crosses surfaces, agents, and review tools that don't belong inside any single feature section.

### Spec → Implementation → Review → Merge

```
1. Composer Ask: write spec, iterate                      §3, §11
2. Composer Agent: generate types from spec               §3
3. Composer Agent: write failing tests (TDD)              §3
4. Composer Agent: implement against tests                §3
5. Review diff in IDE → ask "explain each change"         §3
6. /review-bugbot before pushing                          §5
7. Push → Bugbot reviews on PR → Autofix if enabled       §5, §4
8. Human approval → merge
```

### Ticket → PR (Multi-Repo)

```
1. Jira/Linear MCP pulls ticket context                   §9
2. Composer Ask: break ticket into implementation steps   §3
3. Dispatch Background Agent from mobile/IDE              §4, §8
4. Agent uses GitHub MCP to open PR with description      §9
5. Bugbot auto-reviews                                    §5
6. You review on phone during a break, approve            §8
```

### Best-of-N Parallel Build

```
1. Define narrow task with crisp acceptance criteria
2. /multitask spawns N agents on same task (worktrees)    §4
3. Wait for diffs to land in Agents Window
4. Side-by-side diff review
5. Merge cleanest; discard rest
```

### Frontend with Design Mode

```
1. Composer Agent: scaffold the page using design system  §3
2. Open canvas or browser preview                         §6
3. Activate Design Mode → annotate UI directly            §6
4. Agent edits underlying code from annotations           §6
5. Iterate visually until shipped
```

### Bug Investigation

```
1. Paste error from @terminal                             §10
2. @codebase "find code related to [error]"               §10
3. Composer Ask: "What is the root cause?"                §3
4. Composer Agent: "Fix it and add regression test"       §3
5. Agent runs tests to confirm                            §3
```

---

## 13. Keyboard Shortcuts

> Shortcuts evolve between versions. Verify in **Cursor → Settings → Keyboard Shortcuts** if anything looks wrong on your build.

| Shortcut | Action |
|---|---|
| `Cmd+K` | Inline Edit on selected code |
| `Cmd+I` | Open Composer |
| `Shift+Tab` (in Composer) | Toggle Ask ↔ Agent |
| `Cmd+L` | Open chat sidebar |
| `Cmd+Shift+L` | Add current file to chat |
| `Tab` | Accept Tab suggestion |
| `Cmd+→` | Accept Tab suggestion word-by-word |
| `Esc` | Reject suggestion |
| `Cmd+.` | Toggle AI suggestions |
| `Cmd+Shift+P` | Command Palette (VS Code-shared) |

### Cmd+K Quick Patterns

Select code → `Cmd+K` → type:

```
"convert to async/await"
"add error handling"
"add TypeScript types"
"simplify this"
"add JSDoc"
"make this a reusable hook"
"add loading and error states"
"extract to a separate function"
```

### Slash Commands

```
/review                 review before push (3.7+)
/review-bugbot          run only Bugbot
/review-security        run only Security Review
/multitask              dispatch parallel subagents
/loop                   recurring scheduled agent
```

---

## 14. Team Setup & Closing Principles

### Team Setup Checklist

- [ ] `.cursor/rules/base.mdc` — shared standards, alwaysApply, <200 words
- [ ] `.cursor/rules/*.mdc` — scoped rules per layer (api, components, tests, db)
- [ ] `.cursor/mcp.json` — shared MCP config (commit to repo)
- [ ] `.cursorignore` — exclude build/generated files
- [ ] `AGENTS.md` — cross-tool baseline if you use multiple AI tools
- [ ] `docs/ARCHITECTURE.md` — `@mention`-able architecture reference
- [ ] Bugbot enabled on GitHub/GitLab with team config
- [ ] Shared prompt library in `docs/cursor/prompts.md`
- [ ] Cursor SDK considered if your team has recurring agent automation needs

### Closing Principles

1. **Rules set defaults. @mentions add precision. Agent executes.** Use all three; none alone is enough.
2. **Always-apply rules are a token tax.** Keep them under 200 words. Use scoped or agent-requested for the rest. The Context Usage Report keeps you honest.
3. **Three surfaces, three jobs.** Cmd+K for surgery, Composer for multi-file work, Tab for typing. Stop trying to do all three in Composer.
4. **Orchestration is the leverage.** Parallel agents, best-of-N, mobile dispatch — the senior-engineer skill of 2026 is scoping tasks tight enough to dispatch in parallel, then reviewing critically.
5. **Verify before merging.** Tab, Composer, and Agent all hallucinate. Use Bugbot, run tests, read diffs. Never merge what you haven't read.

---

*Cursor AI Expert Hub — v3.0, June 12, 2026. Verified against Cursor 3.7 docs, changelog, and release notes.*
