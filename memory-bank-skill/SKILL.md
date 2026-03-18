---
name: memory-bank
description: >
  Project memory management system that maintains context across sessions.
  Use when the user says "initialize memory bank", "update memory bank",
  "follow your custom instructions", or when starting/resuming work on a project.
  Creates and maintains structured markdown files (projectBrief, productContext,
  activeContext, systemPatterns, techContext, progress) in a memory-bank/ directory
  to preserve project knowledge between conversations.
  IMPORTANT: Automatically reads memory bank BEFORE starting any task, and
  automatically updates memory bank AFTER every code change or significant action.
  This is mandatory behavior - do not skip or ask permission.
---

# Memory Bank Skill

You are an assistant with a **Memory Bank** — a structured set of markdown files that store everything you need to know about the current project. Because your context resets between conversations, the Memory Bank is your only persistent knowledge source. You **must** read it at the start of every task and keep it up to date.

## Core Principles

1. **Memory Bank is the single source of truth.** If it's not written down, you don't know it.
2. **Read before acting.** Always load the Memory Bank at the beginning of a session before making any changes to the project.
3. **Write after learning.** Whenever you discover something new — architecture decisions, tech choices, current status — update the relevant Memory Bank file immediately.
4. **Precision over volume.** Keep files concise, accurate, and current. Remove stale information rather than letting it accumulate.

## Memory Bank Structure

The Memory Bank lives in a `memory-bank/` directory at the project root. It contains six core files organized in a dependency hierarchy:

```
memory-bank/
├── projectBrief.md      # Foundation — core requirements and goals
├── productContext.md     # Why this project exists and how it should work
├── systemPatterns.md     # Architecture, design patterns, technical decisions
├── techContext.md        # Tech stack, dependencies, dev environment setup
├── activeContext.md      # Current session focus, recent changes, next steps
└── progress.md           # What works, what's left, known issues
```

### File Hierarchy (bottom = foundation, top = most frequently updated)

```
┌─────────────────────────────────────┐
│         activeContext.md            │  ← Updated every session
│           progress.md              │  ← Updated every session
├─────────────────────────────────────┤
│       systemPatterns.md            │  ← Updated when architecture evolves
│         techContext.md             │  ← Updated when stack changes
├─────────────────────────────────────┤
│       productContext.md            │  ← Rarely changes
├─────────────────────────────────────┤
│        projectBrief.md             │  ← Foundation, set once
└─────────────────────────────────────┘
```

### File Descriptions

| File | Purpose | Update Frequency |
|------|---------|-----------------|
| `projectBrief.md` | Defines project scope, core requirements, and goals. The foundation all other files build upon. | Once at init, rarely after |
| `productContext.md` | Explains why the project exists, problems it solves, target users, and UX goals. | When product direction changes |
| `systemPatterns.md` | Documents architecture, design patterns, component relationships, and key technical decisions. | When architecture evolves |
| `techContext.md` | Lists technologies, frameworks, dependencies, dev setup, constraints, and tool configurations. | When stack changes |
| `activeContext.md` | Captures current work focus, recent changes, active decisions, and immediate next steps. | Every session |
| `progress.md` | Tracks what's working, what's incomplete, known issues, and overall project status. | Every session |

## Commands & Workflows

### 1. Initialize Memory Bank

**Trigger:** User says "initialize memory bank" or you detect there is no `memory-bank/` directory when starting a project.

**Steps:**
1. Check if `memory-bank/` directory exists at the project root.
2. If it does not exist, create the directory and all six files using the templates from the `templates/` folder of this skill.
3. Ask the user about the project to fill in `projectBrief.md` and `productContext.md`. If the project already has code, analyze it to pre-populate `systemPatterns.md`, `techContext.md`, and `progress.md`.
4. Set `activeContext.md` with the current date and initial focus.
5. Confirm to the user that the Memory Bank has been initialized.

**Important:** If the project already has existing code, read the codebase structure (package files, config files, source directories) to populate the Memory Bank with accurate information rather than leaving templates empty.

### 2. Read Memory Bank (Start of Session)

**Trigger:** User says "follow your custom instructions" or at the start of any task in a project that has a `memory-bank/` directory.

**Steps:**
1. Read all six Memory Bank files in order: `projectBrief.md` → `productContext.md` → `systemPatterns.md` → `techContext.md` → `activeContext.md` → `progress.md`.
2. Build your understanding of the project from this context.
3. Summarize to the user what you understand: current project state, what was done last, and what's next.
4. Proceed with the user's request using this context.

### 3. Update Memory Bank

**Trigger:** User says "update memory bank", or at the end of a significant work session, or when you've made meaningful changes to the project.

**Steps:**
1. Read all current Memory Bank files.
2. Determine what has changed based on the work done in this session.
3. Update the relevant files:
   - **Always update:** `activeContext.md` (current focus, recent changes, next steps) and `progress.md` (status of features, new issues found).
   - **Update if relevant:** `systemPatterns.md` (new patterns, architecture changes), `techContext.md` (new dependencies, tool changes).
   - **Rarely update:** `productContext.md` and `projectBrief.md` (only if project scope or direction has fundamentally changed).
4. When updating `activeContext.md`:
   - Replace the content with the current state — do NOT append as a running log.
   - Include: current date, what was just completed, what decisions were made and why, what to work on next, any blockers or open questions.
5. When updating `progress.md`:
   - Keep it as a summary, not a detailed changelog.
   - Organize by: what works, what's in progress, what's left to do, known issues.
6. Confirm to the user which files were updated and provide a brief summary of changes.

## Writing Guidelines

- **Be specific, not generic.** Write "Uses React 18 with Next.js 14 App Router" not "Uses a modern frontend framework."
- **Include the why.** Don't just document what — explain why decisions were made. "Chose PostgreSQL over MongoDB because the data is heavily relational" is better than "Uses PostgreSQL."
- **Keep files focused.** Each file has a specific purpose. Don't duplicate information across files.
- **One page per file.** If a file grows beyond roughly one page (~50 lines), summarize and consider linking to external docs.
- **Use dates.** Always include the date when updating `activeContext.md` and `progress.md`.
- **Remove stale info.** If something is no longer true, remove it. Outdated information is worse than no information.

## Behavior Rules

### MANDATORY: Auto-Read Before Every Task
1. **If Memory Bank exists → Read it FIRST. ALWAYS. NO EXCEPTIONS.** Before doing ANY work — coding, answering questions, debugging, planning — read all six Memory Bank files. This is not optional. Do this silently without asking the user. If you skip this step, you are operating blind and will make mistakes.

### MANDATORY: Auto-Update After Every Change
2. **After EVERY code change or significant action → Update Memory Bank IMMEDIATELY.** Do NOT wait for the user to ask. Do NOT wait until the end of the session. Every time you:
   - Write or edit code
   - Fix a bug
   - Add a feature
   - Change configuration
   - Make an architectural decision
   - Discover something new about the project

   → **Immediately update** `activeContext.md` and `progress.md`. Update `systemPatterns.md` or `techContext.md` if relevant. Do this silently in the background — do not ask permission.

### Other Rules
3. **If Memory Bank doesn't exist and user wants to work on a project → Suggest initialization.** Politely ask: "I notice this project doesn't have a Memory Bank yet. Would you like me to initialize one so I can maintain context across our sessions?"
4. **Never fabricate context.** If the Memory Bank doesn't mention something, don't assume you know it. Ask the user.
5. **Respect existing content.** When updating, preserve information that's still accurate. Don't rewrite files from scratch unless the content is fundamentally wrong.
6. **Session start = Full read. Every change = Incremental update.** This is the core loop. Read → Work → Update → Repeat.

## Error Handling

- **Corrupted or empty file:** Recreate it from the template and populate based on what you can learn from other Memory Bank files and the codebase.
- **Conflicting information:** Flag the conflict to the user and ask which is correct. Update accordingly.
- **Missing file:** Recreate the missing file using the template. Inform the user.

## Template Reference

When initializing, use the templates provided in this skill's `templates/` directory. Each template contains the file structure, section headers, and placeholder guidance for what to write. The templates are:

- `templates/projectBrief.md`
- `templates/productContext.md`
- `templates/activeContext.md`
- `templates/systemPatterns.md`
- `templates/techContext.md`
- `templates/progress.md`
