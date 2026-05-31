---
name: ai-context-handoff
description: >
  Use this skill when the user wants to capture, document, and transfer the current chat session's
  full context to a new AI session. Triggers include: "handoff", "context dump", "save context",
  "switching to new chat", "AI is losing context", "document this session", "create a handoff",
  "context export", "session summary for new AI", or any variation of wanting to preserve
  conversation state before starting fresh. The output is a structured Markdown document
  saved to docs/ai-context-handoff/<Month YYYY>/<timestamp>-<slug>.md that a new AI can
  ingest as its first message to resume work with zero ambiguity.
---

# AI Context Handoff Skill

> **Role:** Senior AI Memory Architect  
> **Purpose:** Capture everything a new AI instance needs to resume work as if it were present from the beginning. No detail is too small. No section is mandatory — empty sections are valid and should remain as stubs to signal they were considered.

---

## What Is This?

AI conversations have a memory limit. After enough back-and-forth, the AI starts forgetting earlier details — the decisions made, the files touched, the things that didn't work, and the things that did. When that happens, or when you simply need to start a fresh chat, you lose all of that accumulated context. You'd have to re-explain everything from scratch, and the new AI would have no idea where you left off.

This skill solves that problem. It creates a structured handoff document — a Markdown file — that captures everything important from the current session in one organised place. Think of it as writing a detailed briefing note for a colleague who is joining mid-project. When the new AI reads it, it knows exactly what was happening, what decisions were made and why, what failed, what's next, and how you like to work.

## How the Handoff Works

The process has two parts. First, at the end of a session (or whenever context is getting long), you ask the AI to create a handoff. It reads through the entire conversation and writes a structured document covering the active task, completed work, key decisions, blockers, files involved, and more. That document gets saved as a Markdown file inside `docs/ai-context-handoff/` in your project, organised by month.

Second, when you start a new chat, you open the handoff file, attach it, and paste the Continuation Prompt from the bottom of that file as your first message. The new AI reads everything in the file, confirms what it understands, and picks up exactly where the last session ended — no re-explaining, no starting over.

## What Gets Captured

A handoff document is not a chat transcript. It is a distilled, organised record of the things that actually matter for the next session. That includes the current task and exactly what to do next, a prioritised queue of remaining work, key decisions and the reasoning behind them, dead ends to avoid repeating, unresolved blockers and open questions, files and assets that are in play, technical details like commands and environment variables, and a note on how you prefer to communicate. Sections that don't apply to a session are left as empty stubs rather than deleted — this tells the new AI they were considered and found empty, not overlooked.

---

## Table of Contents

1. [When to Use This Skill](#1-when-to-use-this-skill)
2. [How to Use This Skill](#2-how-to-use-this-skill)
3. [File Storage Architecture](#3-file-storage-architecture)
4. [Handoff Document Structure](#4-handoff-document-structure)
5. [Section Fill Rules](#5-section-fill-rules)
6. [Depth Calibration Guide](#6-depth-calibration-guide)
7. [Full Template](#7-full-template)
8. [Worked Examples](#8-worked-examples)
9. [Filename & Slug Rules](#9-filename--slug-rules)
10. [Multi-File Handoff Chains](#10-multi-file-handoff-chains)
11. [Quality Checklist](#11-quality-checklist)

---

## 1. When to Use This Skill

### Primary Triggers (explicit)

| User says something like… | Action |
|---|---|
| "Create a handoff" / "Make a handoff doc" | Full handoff |
| "I'm switching to a new chat" | Full handoff |
| "Context dump" / "Save this session" | Full handoff |
| "The AI is losing context, help" | Full handoff |
| "Document what we've done so far" | Full handoff |
| "Export this session" | Full handoff |
| "Quick handoff" / "Short handoff" | Lightweight handoff (skip optional sections) |

### Secondary Triggers (inferred)

- Conversation has exceeded ~40 messages with no handoff yet
- User is about to start a significantly different sub-task mid-session
- User mentions they need to share context with a teammate or another tool
- User is about to close the browser / end the session and wants continuity

### Never Use This Skill For

- General session summaries not meant to be ingested by an AI
- Internal notes or personal task lists (no Continuation Prompt needed)
- Short conversations with no meaningful accumulated context

---

## 2. How to Use This Skill

### Step-by-step execution

**Step 1 — Assess conversation depth.**  
Silently review the full conversation. Categorise it:
- `deep`: many tasks, code, decisions, files — full handoff needed
- `medium`: moderate discussion, a few tasks — standard handoff
- `shallow`: brief exchange, not much context — lightweight handoff or ask user

**Step 2 — Identify what is known vs. unknown.**  
Some sections will be empty because the conversation never covered them (e.g. no files were touched). Leave those sections in the document as explicit empty stubs — never omit sections entirely, as the new AI needs to know they were checked.

**Step 3 — Write the handoff document in Markdown.**  
Follow the Full Template in §7. Be a scribe, not a summariser — capture decisions verbatim where possible. Opinions, frustrations, and preferences stated by the human are first-class data.

**Step 4 — Generate the filename.**  
Follow the naming rules in §9.

**Step 5 — Save to the correct path.**  
Follow the storage architecture in §3.

**Step 6 — Output the Continuation Prompt block.**  
The last thing in the handoff doc (and optionally repeated in chat) is a ready-to-paste prompt the user can drop into a new chat as their opening message.

**Step 7 — Tell the user the file path and confirm.**  
Example:
```
Handoff saved → docs/ai-context-handoff/May 2026/2026-05-31T1430-learner-hub-auth.md

Copy the Continuation Prompt at the bottom of the file and paste it as your first message in the new chat.
```

---

## 3. File Storage Architecture

### Root location

All handoff files live under:

```
docs/
└── ai-context-handoff/
    ├── May 2025/
    │   ├── 2025-05-14T0930-auth-refactor.md
    │   └── 2025-05-28T1615-dashboard-bugfix.md
    ├── June 2025/
    │   └── 2025-06-03T1100-prisma-migration.md
    ├── April 2026/
    │   └── 2026-04-22T0845-design-system-v2.md
    └── May 2026/
        ├── 2026-05-10T1330-branding-sprint.md
        └── 2026-05-31T1430-learner-hub-auth.md
```

### Directory naming

| Pattern | Example |
|---|---|
| `<Month YYYY>` | `May 2026` |
| Month = full English month name | `January`, `February`, … `December` |
| Year = 4-digit year | `2026` |

### Filename format

```
YYYY-MM-DDTHHMM-<slug>.md
```

| Part | Description | Example |
|---|---|---|
| `YYYY-MM-DD` | ISO date | `2026-05-31` |
| `T` | Literal separator | `T` |
| `HHMM` | 24-hour local time, no colon | `1430` |
| `-` | Separator | `-` |
| `<slug>` | 2–5 lowercase hyphenated words describing the session | `learner-hub-auth` |
| `.md` | Extension | `.md` |

**Full example:** `2026-05-31T1430-learner-hub-auth.md`

### Slug rules

- Derived from the dominant topic of the session
- 2–5 words, lowercase, hyphen-separated
- No dates, no pronouns, no stopwords ("the", "a", "for")
- Descriptive enough to grep for later

**Good slugs:** `prisma-migration-fix`, `design-system-tokens`, `ci-cd-vps-setup`, `auth-token-refresh`  
**Bad slugs:** `session`, `handoff`, `context`, `new-chat`, `stuff`

### Creating the directory

When saving:

```bash
mkdir -p "docs/ai-context-handoff/May 2026"
```

If `docs/` does not exist at the project root, create it. If no clear project root exists, use the user's home or working directory as the anchor and document the path in the handoff.

---

## 4. Handoff Document Structure

Each subsection below describes one block of the handoff document. The full copy-paste template is in §7.

---

### 4.1 Document Header Block

The very top of every handoff file. Non-negotiable — always fill this completely.

```yaml
---
handoff_version: 1.0
created_at: 2026-05-31T14:30
session_depth: deep | medium | shallow
previous_handoff: <path to previous handoff file, or null>
next_handoff: null
primary_topic: <one-line description>
ai_model: Claude Sonnet 4.6
human_author: <name or "anonymous">
status: complete | partial | emergency
---
```

**Field notes:**

| Field | Purpose |
|---|---|
| `handoff_version` | Schema version — always `1.0` for now |
| `session_depth` | Guides how much the new AI needs to read before diving in |
| `previous_handoff` | Chain link — allows the new AI to load historical context |
| `next_handoff` | Filled in after the next handoff is created; leave `null` initially |
| `primary_topic` | Single sentence. The new AI reads this first |
| `status` | `complete` = full handoff, `partial` = interrupted mid-write, `emergency` = created in a rush |

---

### 4.2 Session Identity

High-level metadata about the session itself.

```markdown
## Session Identity

- **Date:** 31 May 2026
- **Time (local):** 14:30 IST
- **Duration (approx.):** 2h 15m
- **AI Model Used:** Claude Sonnet 4.6 (claude.ai)
- **Interface:** claude.ai web
- **Human:** Bhargav / anonymous
- **Session Type:** development | design | planning | debugging | research | mixed
- **Criticality:** low | medium | high | critical
```

---

### 4.3 Project & Workspace Context

The scaffolding. What project are we in? What does the codebase look like?

```markdown
## Project & Workspace Context

### Project Name
<e.g. LernenHub — Admin Portal>

### Repository
- **URL:** https://github.com/<org>/<repo>
- **Branch:** <current branch>
- **Local path:** ~/projects/<name>

### Tech Stack
- Runtime: Node.js 20 / Bun 1.x
- Framework: Next.js 14 (App Router)
- Language: TypeScript 5.x
- Database: PostgreSQL 16 via Prisma ORM
- Styling: Tailwind CSS + Shadcn/ui
- Deployment: Ubuntu VPS, PM2, nginx

### Environment
- **Dev machine:** macOS / Linux / Windows
- **OS on server:** Ubuntu 24.04
- **Node version:** 20.x
- **Package manager:** pnpm / npm / yarn / bun

### Key Config Files
- `.env.local` — local dev env vars
- `prisma/schema.prisma` — DB schema
- `.github/workflows/deploy.yml` — CI/CD
- `CLAUDE.md` — AI agent instructions

### Monorepo Structure (if applicable)
<Describe top-level packages or apps, or write "N/A — single package">
```

---

### 4.4 Current Task State

The heart of the handoff. What was being worked on, what is done, what is next.

```markdown
## Current Task State

### Active Task
**Task:** <name>
**Status:** in-progress | blocked | waiting-for-review | paused
**Started:** <date/time>
**Last action taken:** <what was the most recent thing done>
**Next immediate action:** <what should the new AI do first>

---

### Task Queue (ordered by priority)

| Priority | Task | Status | Notes |
|---|---|---|---|
| P0 | Fix broken migration on prod | blocked | Waiting for DB access |
| P1 | Add token refresh to auth flow | in-progress | 60% done |
| P2 | Write tests for UserService | not started | — |

---

### Completed This Session
- [x] Resolved PM2 process crash on server restart
- [x] Updated GitHub Actions deploy workflow to verify PM2 status
- [x] Fixed Prisma unapplied migration error

---

### Deferred / Parked Tasks
- Design system token refactor — deferred to next sprint
- Explore Bun as package manager — parked, needs benchmark first
```

---

### 4.5 Conversation Intelligence

Distilled knowledge from the conversation itself — insights, pivots, breakthroughs.

```markdown
## Conversation Intelligence

### Key Insights Reached
1. PM2's `--watch` flag was interfering with the prod build — removing it fixed the crash loop.
2. Prisma `migrate deploy` must run before the server starts, not after PM2 startup.
3. The GitHub Actions SSH step was using the wrong user — should be `ubuntu`, not `root`.

### Pivots Made
- Started with Bun, switched back to Node after build pipeline incompatibility discovered.
- Originally planning raw SQL for report query — switched to Prisma after realising schema covered it.

### Dead Ends to Avoid
- ❌ Running `prisma migrate dev` on the prod server — causes drift. Use `migrate deploy` only.
- ❌ Using `git push --force` on `main` — CI ignores force-pushes, use a PR.
- ❌ Nginx `proxy_pass` to `localhost:3001` without `http://` prefix — causes 502.

### Tone & Dynamics
- Human prefers direct, no-fluff responses
- Comfortable with technical depth — no hand-holding
- Was frustrated at one point about the migration issue — acknowledge before diving into solutions
- Preferred format: bullet points + code blocks, minimal prose
```

---

### 4.6 Technical Context

Code-level details that the new AI must know to avoid regression or repeat work.

```markdown
## Technical Context

### Code Written This Session

| File | Change Type | Summary |
|---|---|---|
| `.github/workflows/deploy.yml` | Modified | Added PM2 status check after deploy |
| `prisma/migrations/20260531_add_user_roles/` | Created | New migration for RBAC |
| `src/lib/auth/token.ts` | Modified | Added refresh token rotation logic |
| `src/app/(admin)/dashboard/page.tsx` | New | Dashboard skeleton with stats cards |

### Unresolved Code Issues
- `UserService.findByEmail()` has no error boundary — throws raw Prisma error on null
- `refreshToken` function doesn't handle concurrent refresh race conditions yet
- Dashboard stats are hardcoded — need to wire up to `/api/stats` endpoint

### Environment Variables

```env
# Added this session
REFRESH_TOKEN_SECRET=<set in .env.local and Vercel>
NEXT_PUBLIC_APP_URL=<check staging vs prod>

# Modified
DATABASE_URL=<updated connection string — check prod .env>
```

### Commands to Know

```bash
# Deploy manually
ssh ubuntu@<server-ip> "cd /var/www/lernen-hub && git pull && pnpm install && pnpm build && pm2 restart lernen-hub"

# Run migrations on prod
npx prisma migrate deploy

# Check PM2 status
pm2 status

# Tail logs
pm2 logs lernen-hub --lines 100
```

### Dependencies Added

```
pnpm add zod @tanstack/react-table lucide-react
pnpm add -D @types/node
```

### API Endpoints Modified

| Method | Endpoint | Change |
|---|---|---|
| POST | `/api/auth/refresh` | New — refresh token rotation |
| GET | `/api/users` | Added `role` filter param |
| DELETE | `/api/sessions/:id` | Fixed 500 on invalid session ID |
```

---

### 4.7 Decisions & Rationale Log

A record of *why* — the most overlooked part of context. The new AI must not re-litigate settled choices.

```markdown
## Decisions & Rationale Log

### Decision: Use Prisma over raw SQL for report queries
- **Date:** 2026-05-31
- **Decision:** Use Prisma ORM queries, not raw SQL
- **Why:** Schema already models all needed relations; raw SQL would duplicate schema knowledge
- **Alternatives considered:** Drizzle ORM, raw pg queries
- **Reversible?** Yes, but only with significant refactor

### Decision: PM2 ecosystem file over CLI flags
- **Date:** 2026-05-31
- **Decision:** Use `ecosystem.config.js` for PM2 instead of ad-hoc CLI flags
- **Why:** Reproducibility across server restores; avoids config drift
- **Alternatives considered:** Docker (deferred, overkill for current scale)
- **Reversible?** Yes

### Decision: Shadcn/ui over MUI for admin portal
- **Date:** Earlier session (carried forward)
- **Decision:** Shadcn/ui + Tailwind
- **Why:** Tailwind-native, no runtime overhead, easier dark mode
- **Alternatives considered:** MUI (rejected — too opinionated), Ant Design (rejected — design language mismatch)
- **Reversible?** No — deep in codebase, do not revisit
```

---

### 4.8 Open Questions & Blockers

What is unresolved. Honesty is the most important quality here.

```markdown
## Open Questions & Blockers

### Blockers (must resolve before proceeding)

| # | Blocker | Owner | Notes |
|---|---|---|---|
| B1 | Prod DB credentials not available | Bhargav | Need to pull from Bitwarden |
| B2 | Vercel env vars not synced with local | Bhargav | Manual step pending |

### Open Questions (need decision before implementation)

| # | Question | Context |
|---|---|---|
| Q1 | Should refresh tokens be stored in DB or Redis? | DB adds latency, Redis adds infra complexity |
| Q2 | Single-tenant or multi-tenant schema for Phase 2? | Affects schema design significantly |
| Q3 | Which logging service — Axiom, Datadog, or self-hosted Loki? | Budget constraint applies |

### Assumptions Made (not yet validated)
- Assuming the new server has at least 2GB RAM for Next.js build
- Assuming pnpm is already installed globally on VPS
- Assuming Cloudflare is handling SSL termination, not nginx directly
```

---

### 4.9 File & Asset Inventory

What files, designs, or assets are relevant and where they live.

```markdown
## File & Asset Inventory

### Source Files (active)

| Path | Purpose | Status |
|---|---|---|
| `src/app/(admin)/` | Admin portal routes | In progress |
| `src/lib/auth/` | Auth utilities | Stable |
| `prisma/schema.prisma` | Database schema | Modified this session |
| `.github/workflows/deploy.yml` | CI/CD pipeline | Modified this session |

### Design Files

| File | Location | Purpose |
|---|---|---|
| Design System v2 | Figma — BrillBytes workspace | Token definitions, component library |
| HR Templates | Local — `~/Documents/hr-templates/` | Payslip, offer letter |

### Documents & Notes

| File | Location | Purpose |
|---|---|---|
| This handoff | `docs/ai-context-handoff/May 2026/...` | Context transfer |
| Previous handoff | `docs/ai-context-handoff/May 2026/2026-05-28T...` | Prior session |
| CLAUDE.md | Repo root | AI agent instructions for this project |

### Assets

| Asset | Type | Location |
|---|---|---|
| BrillBytes logo mark | SVG | `public/brand/logo-dark.svg` |
| Favicon set | PNG/SVG | `public/favicon/` |

### External References

| Resource | URL | Why it matters |
|---|---|---|
| Prisma migration docs | https://www.prisma.io/docs/concepts/components/prisma-migrate | Used for prod migration commands |
| PM2 ecosystem docs | https://pm2.keymetrics.io/docs/usage/application-declaration/ | PM2 config reference |
```

---

### 4.10 Human Context

Who is the human, what do they care about, how do they communicate. Vital for tone calibration.

```markdown
## Human Context

### Identity
- **Name / handle:** <name>
- **Role:** <role>
- **Location:** <city, country + timezone>
- **Experience level:** junior | mid | senior | expert

### Communication Preferences
- <bullet list of how they like to be spoken to>
- <format preferences>
- <things that annoy them>

### Current Energy / Mood
<description — captured from conversation tone>

### Goals This Sprint
<list of higher-level goals beyond the immediate task>

### Known Preferences (carry forward always)
<persistent preferences the AI should always respect, e.g.:>
- Dark, minimal, premium aesthetic across all design work
- No cool/blue hues in warm-palette projects
- Preferred font pairing: Unbounded + DM Sans
- Conventional commits with co-author trailers for AI-assisted work
```

---

### 4.11 AI Behaviour Instructions

Explicit instructions for how the new AI should behave in the resumed session.

```markdown
## AI Behaviour Instructions

### Immediate priorities for the new session
1. Read §4.4 (Current Task State) first — start from the Active Task
2. Do not re-explain decisions already logged in §4.7 — treat them as settled
3. Review §4.8 blockers before suggesting any implementation that depends on them

### Constraints to respect
- Do not refactor code that is not directly related to the active task
- Do not suggest switching ORMs, frameworks, or package managers — settled decisions
- Do not use `migrate dev` on production — only `migrate deploy`
- Respect the existing Tailwind + Shadcn/ui design system

### Tone guidance
- Match the human's directness — no filler phrases
- When in doubt about scope, ask one sharp question instead of guessing
- Keep code blocks complete and copy-paste ready

### Things to proactively flag
- Any new env vars that need to be set
- Any breaking changes before writing them
- Any assumption that affects more than one file

### Things NOT to do
- Do not re-introduce libraries or patterns marked as dead ends in §4.5
- Do not produce partial code snippets that require mental reconstruction
- Do not ask "would you like me to proceed?" — just proceed and report
```

---

### 4.12 Continuation Prompt

**The most important output.** This is what gets pasted into the new chat. It must be self-contained, comprehensive, and formatted for direct use.

The continuation prompt lives at the very bottom of every handoff file and is formatted as a fenced code block so it can be copied without selecting surrounding text.

---

## 5. Section Fill Rules

| Section | Required? | When to leave empty |
|---|---|---|
| Document Header | **Always** | Never |
| Session Identity | **Always** | Never |
| Project & Workspace Context | **Always** | Never |
| Current Task State | **Always** | Never |
| Conversation Intelligence | Recommended | Leave empty stub if session was very short |
| Technical Context | Conditional | Leave empty stub if no code was written |
| Decisions & Rationale Log | Conditional | Leave empty stub if no decisions were made |
| Open Questions & Blockers | Conditional | Leave empty stub if nothing is unresolved |
| File & Asset Inventory | Conditional | Leave empty stub if no files were touched |
| Human Context | Recommended | At minimum note name and communication style |
| AI Behaviour Instructions | **Always** | Always include at least the first priority |
| Continuation Prompt | **Always** | Never |

### Empty section format

When a section has no content, write an explicit stub — never delete the heading:

```markdown
## Technical Context

_No code was written or modified this session._
```

Empty sections are a signal, not noise — they tell the new AI "this was checked and found empty."

---

## 6. Depth Calibration Guide

### Shallow session (< 10 messages, exploratory)

Fill: Header, Session Identity, Current Task State (minimal), Continuation Prompt  
Skip body: Technical Context, Decisions Log, File Inventory (leave as stubs)  
Target size: ~40–80 lines

### Medium session (10–40 messages, mix of discussion and work)

Fill: All core sections  
Expand: Conversation Intelligence, Decisions Log  
Target size: ~150–300 lines

### Deep session (40+ messages, heavy implementation)

Fill: All sections, all subsections  
Expand: Technical Context (full file table, commands, env vars), Decisions Log (every major choice)  
Target size: ~300–800 lines

### Emergency handoff (AI losing context mid-task, urgent)

Priority fill in this order:
1. Active Task + Next immediate action
2. Dead Ends to Avoid
3. Key commands or code needed right now
4. Continuation Prompt

Everything else: fill what you can, mark rest as `emergency — incomplete`  
Target size: whatever is feasible in under 5 minutes

---

## 7. Full Template

Copy this entire block for each new handoff. Replace all `<...>` placeholders.

---

```markdown
---
handoff_version: 1.0
created_at: <YYYY-MM-DDTHH:MM>
session_depth: <deep | medium | shallow>
previous_handoff: <path or null>
next_handoff: null
primary_topic: <one-line description of what this session was about>
ai_model: <model name, e.g. Claude Sonnet 4.6>
human_author: <name or anonymous>
status: <complete | partial | emergency>
---

# AI Context Handoff — <Primary Topic>

> **Handoff created:** <Date> at <Time>
> **Session depth:** <deep / medium / shallow>
> **Status:** <complete / partial / emergency>
> **Previous handoff:** <path or "none">

---

## Session Identity

- **Date:** <DD Month YYYY>
- **Time (local):** <HH:MM TZ>
- **Duration (approx.):** <Xh Ym>
- **AI Model Used:** <model name and interface>
- **Interface:** <claude.ai / Claude Code / API / Cursor / other>
- **Human:** <name or anonymous>
- **Session Type:** <development | design | planning | debugging | research | mixed>
- **Criticality:** <low | medium | high | critical>

---

## Project & Workspace Context

### Project Name
<project name>

### Repository
- **URL:** <url or "local only">
- **Branch:** <branch>
- **Local path:** <path>

### Tech Stack
<list technologies and versions>

### Environment
- **Dev machine OS:** <OS>
- **Server OS:** <OS or N/A>
- **Node / Runtime version:** <version>
- **Package manager:** <pnpm / npm / yarn / bun>

### Key Config Files
<list important config file paths with one-line purpose>

### Monorepo Structure
<describe or write "N/A — single package">

---

## Current Task State

### Active Task

**Task:** <task name>
**Status:** <in-progress | blocked | waiting-for-review | paused>
**Started:** <date/time>
**Last action taken:** <exactly what was the last thing done>
**Next immediate action:** <exactly what the new AI should do first — be specific>

---

### Task Queue

| Priority | Task | Status | Notes |
|---|---|---|---|
| P0 | | | |
| P1 | | | |
| P2 | | | |

_Or: No task queue — single active task only._

---

### Completed This Session

- [x] <task>
- [x] <task>

_Or: No tasks completed this session._

---

### Deferred / Parked Tasks

- <task> — reason for deferral

_Or: None._

---

## Conversation Intelligence

### Key Insights Reached

_No significant insights, or list:_
1. <insight>
2. <insight>

### Pivots Made

_No pivots, or list:_
- <pivot description>

### Dead Ends to Avoid

_None encountered, or list:_
- ❌ <what not to do and why>
- ❌ <what not to do and why>

### Tone & Dynamics

- <communication preference>
- <format preference>
- <mood or energy note>

---

## Technical Context

### Code Written This Session

| File | Change Type | Summary |
|---|---|---|
| | | |

_Or: No code was written or modified this session._

### Unresolved Code Issues

- <issue>

_Or: None._

### Environment Variables

```env
# New or changed — redact actual values
KEY=<description>
```

_Or: No env var changes._

### Commands to Know

```bash
# Important shell commands for this project
```

_Or: No special commands needed._

### Dependencies Added

```
<package manager> add <packages>
```

_Or: No new dependencies._

### API Endpoints Modified

| Method | Endpoint | Change |
|---|---|---|
| | | |

_Or: No API changes._

---

## Decisions & Rationale Log

### Decision: <title>
- **Date:** <date>
- **Decision:** <what was decided>
- **Why:** <rationale>
- **Alternatives considered:** <what else was on the table>
- **Reversible?** <Yes / No / With effort>

---

_Or: No architectural or significant decisions were made this session._

---

## Open Questions & Blockers

### Blockers (must resolve before proceeding)

| # | Blocker | Owner | Notes |
|---|---|---|---|
| B1 | | | |

_Or: No blockers._

### Open Questions (need decision before implementation)

| # | Question | Context |
|---|---|---|
| Q1 | | |

_Or: No open questions._

### Assumptions Made (not yet validated)

- <assumption>

_Or: No unvalidated assumptions._

---

## File & Asset Inventory

### Source Files (active)

| Path | Purpose | Status |
|---|---|---|
| | | |

_Or: No source files tracked this session._

### Design Files

| File | Location | Purpose |
|---|---|---|
| | | |

_Or: N/A_

### Documents & Notes

| File | Location | Purpose |
|---|---|---|
| This handoff | `docs/ai-context-handoff/<Month YYYY>/<filename>.md` | Context transfer |
| Previous handoff | <path or "none"> | Prior session |

### Assets

<list or write "N/A">

### External References

| Resource | URL | Why it matters |
|---|---|---|
| | | |

_Or: No external references._

---

## Human Context

### Identity
- **Name / handle:** <name>
- **Role:** <role>
- **Location:** <city, country + timezone>
- **Experience level:** <junior | mid | senior | expert>

### Communication Preferences
- <preference>
- <preference>

### Current Energy / Mood
<description — captured from conversation tone>

### Goals This Sprint
- <goal>

### Known Preferences (carry forward always)
- <persistent preference>
- <persistent preference>

---

## AI Behaviour Instructions

### Immediate priorities for the new session
1. <first action — be specific>
2. <second action>
3. <third action if needed>

### Constraints to respect
- <constraint>
- <constraint>

### Tone guidance
- <guideline>
- <guideline>

### Things to proactively flag
- <what to flag>

### Things NOT to do
- <prohibition>
- <prohibition>

---

## Continuation Prompt

> Copy everything inside the code block below and paste it as your **first message** in a new chat. Attach the handoff `.md` file to that message — you do not need to paste its contents inline, the file itself is the context.

```
I'm continuing a development session from a previous chat. I have attached the full AI context handoff document for that session as a file. Please read it completely before responding, then confirm what you understand and state the next action you will take.

Handoff file: docs/ai-context-handoff/<Month YYYY>/<filename>.md

Please:
1. Read the attached handoff file in full before replying
2. Confirm you have understood the handoff — state the active task and the next action you will take
3. Flag any blockers or clarifications needed before proceeding
4. Begin — do not wait for further instruction unless a blocker requires it

<!-- Add any additional context for this new session below this line, if needed -->
```

---

## 8. Worked Examples

### Example A: Emergency Handoff (minimal, mid-task)

**Scenario:** AI is 80+ messages deep into a Prisma migration fix. Context window nearly full. User needs to switch chats immediately.

```markdown
---
handoff_version: 1.0
created_at: 2026-05-31T16:45
session_depth: deep
previous_handoff: null
next_handoff: null
primary_topic: Fix unapplied Prisma migration causing runtime crash on prod
ai_model: Claude Sonnet 4.6
human_author: Bhargav
status: emergency
---

# AI Context Handoff — Prisma Migration Fix (Emergency)

## Session Identity
- **Date:** 31 May 2026
- **Time:** 16:45 IST
- **AI Model:** Claude Sonnet 4.6 — claude.ai
- **Session Type:** Debugging
- **Criticality:** Critical

## Current Task State

### Active Task
**Task:** Apply pending Prisma migration `20260531_add_user_roles` on production
**Status:** blocked
**Last action taken:** Confirmed via `prisma migrate status` that migration is pending on prod
**Next immediate action:** SSH into prod server and run `npx prisma migrate deploy`, then restart PM2

## Technical Context

### Commands to Know

```bash
ssh ubuntu@<server-ip>
cd /var/www/lernen-hub
npx prisma migrate deploy
pm2 restart learner-hub
pm2 logs learner-hub --lines 50
```

## Conversation Intelligence

### Dead Ends to Avoid
- ❌ Do NOT run `prisma migrate dev` on prod — creates shadow DB, can corrupt schema
- ❌ Do NOT restart PM2 before migration runs — server crashes immediately on startup

## Continuation Prompt

```
Continuing an emergency debugging session. I have attached the full handoff file — please read it before responding.

Handoff file: docs/ai-context-handoff/May 2026/2026-05-31T1645-prisma-migration-fix.md

Please:
1. Read the attached handoff file in full before replying
2. Confirm you have understood — state the active task and next action
3. Flag any blockers before proceeding
4. Begin immediately

<!-- Add any extra context for this session here if needed -->
```
```

---

### Example B: Full Deep Session

*(Apply all 12 sections from the Full Template with real conversation data. Omitted here — the template in §7 is the authoritative reference.)*

---

### Example C: Design Session (No Code)

**Scenario:** 2-hour design exploration session. No code written. Design decisions made. Figma files referenced.

Key differences from a dev session:
- Technical Context section → stub only (`_No code written._`)
- Decisions & Rationale Log → filled with design decisions (colour choices, font selections, layout approaches)
- File & Asset Inventory → Figma files, exported PNGs, design token docs
- Session Type → `design`
- AI Behaviour Instructions → include "Do not suggest code implementations unless asked"

---

## 9. Filename & Slug Rules

### Slug generation algorithm

1. Identify the 2–5 most distinctive nouns or verb-noun pairs from the session
2. Lowercase everything
3. Remove stopwords: the, a, an, for, of, to, with, in, on, at, by, and, or
4. Hyphenate
5. Keep total slug under 40 characters

### Examples

| Session topic | Good slug | Bad slug |
|---|---|---|
| Fixing Prisma migration on production | `prisma-migration-fix` | `fix` |
| Building LernenHub auth token refresh | `learner-hub-token-refresh` | `auth` |
| Design system dark theme exploration | `design-system-dark-theme` | `session` |
| CI/CD pipeline setup for VPS | `ci-cd-vps-setup` | `pipeline` |
| Branding sprint for BrillBytes | `brillbytes-branding` | `branding` |
| NestJS API rate limiting middleware | `nestjs-rate-limit-api` | `backend` |
| HR payslip template design | `hr-payslip-template` | `template` |

### Full filename examples

```
2026-05-31T1430-prisma-migration-fix.md
2026-05-31T0915-learner-hub-token-refresh.md
2026-04-22T1100-design-system-dark-theme.md
2025-12-10T1600-ci-cd-vps-setup.md
2026-05-10T1330-brillbytes-branding.md
```

---

## 10. Multi-File Handoff Chains

When work spans multiple sessions, handoffs form a chain. Each file references the previous and (eventually) the next.

### Chain structure

```
docs/ai-context-handoff/
├── May 2026/
│   ├── 2026-05-10T0900-auth-setup.md            ← chain start (previous_handoff: null)
│   ├── 2026-05-15T1400-auth-token-refresh.md    ← previous_handoff: auth-setup
│   ├── 2026-05-22T1100-auth-rbac.md             ← previous_handoff: auth-token-refresh
│   └── 2026-05-31T1430-auth-prod-deploy.md      ← previous_handoff: auth-rbac (current)
```

### YAML header in chained files

```yaml
previous_handoff: docs/ai-context-handoff/May 2026/2026-05-22T1100-auth-rbac.md
next_handoff: null  # update this after next session's handoff is created
```

### After creating a new handoff in a chain

Go back to the previous handoff file and update its `next_handoff` field:

```yaml
next_handoff: docs/ai-context-handoff/May 2026/2026-05-31T1430-auth-prod-deploy.md
```

### Chain reading instructions for new AI

In the Continuation Prompt, if a chain exists, add:

```
This is handoff #4 in a chain. Prior handoffs are available at:
- docs/ai-context-handoff/May 2026/2026-05-22T1100-auth-rbac.md (most recent prior)

You do NOT need to read earlier handoffs unless I ask. Start with this document only.
```

### When to start a new chain vs. continuing an existing one

- **Same project, same feature thread** → continue the chain (same slug family, increment context)
- **Same project, new feature** → start a new chain with a fresh slug
- **Different project entirely** → new chain, new directory if needed

---

## 11. Quality Checklist

Before saving a handoff, verify each item:

### Content completeness

- [ ] Active task is clearly stated with a specific next action (not vague like "continue work")
- [ ] All dead ends are documented with ❌ markers
- [ ] All significant decisions have rationale and alternatives noted
- [ ] All blockers are listed with owners
- [ ] Human communication preferences are captured
- [ ] AI behaviour instructions are present with at least 3 specific directives

### Structural correctness

- [ ] YAML front matter is valid (no unquoted colons in values)
- [ ] All section headers use `##` or `###` — no skipped levels
- [ ] Tables have header rows and alignment dashes
- [ ] Code blocks have language tags where applicable
- [ ] Empty sections are stubbed with italicised text — not deleted
- [ ] No placeholder text (`<fill this in>`) left in final file

### Filename & storage

- [ ] Filename follows `YYYY-MM-DDTHHMM-<slug>.md` exactly
- [ ] Slug is descriptive (2–5 meaningful words, no stopwords)
- [ ] Saved to `docs/ai-context-handoff/<Month YYYY>/` with correct month spelling
- [ ] Directory created with `mkdir -p` if it didn't exist
- [ ] Previous handoff's `next_handoff` field updated (if continuing a chain)

### Continuation Prompt

- [ ] Prompt is inside a fenced code block (user can copy without selecting surrounding text)
- [ ] Prompt instructs the new AI to read the attached file before responding
- [ ] Prompt references the exact file path
- [ ] Prompt includes the 4-step instruction set (read, confirm, flag, begin)
- [ ] Prompt does NOT include a "paste content here" placeholder — the file is attached, not inlined
- [ ] A comment line at the bottom of the prompt invites the user to add extra context if needed
- [ ] Prompt does not assume the new AI knows anything from the current session

---

*Skill version: 1.0*
*Skill name: ai-context-handoff*
*Designed for: Claude Code, claude.ai, and any AI coding environment where context continuity matters across sessions.*