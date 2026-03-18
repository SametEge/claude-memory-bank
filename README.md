# Memory Bank Skill for Claude Code

A persistent memory management skill for Claude Code that maintains project context across conversations. Inspired by [Cline's Memory Bank](https://docs.cline.bot/features/memory-bank) feature, adapted as a portable skill for Claude Code.

## The Problem

Claude Code starts every conversation with a blank slate. It doesn't remember what you worked on yesterday, what architectural decisions were made, or what's left to do. You end up repeating context every single time.

## The Solution

**Memory Bank** gives Claude Code a structured, file-based memory system. It creates a `memory-bank/` directory in your project root containing six markdown files that act as persistent project knowledge. Claude reads them at the start of each session and updates them as work progresses.

No more re-explaining your project. No more lost context. No more forgotten decisions.

## How It Works

```
memory-bank/
├── projectBrief.md      # Foundation — what this project is and its goals
├── productContext.md     # Why it exists, who it's for, UX goals
├── systemPatterns.md     # Architecture, design patterns, technical decisions
├── techContext.md        # Tech stack, dependencies, dev setup
├── activeContext.md      # Current focus, recent changes, next steps
└── progress.md           # What works, what's left, known issues
```

These files form a hierarchy — `projectBrief.md` is the stable foundation at the bottom, while `activeContext.md` and `progress.md` are updated every session at the top:

```
  activeContext.md / progress.md       ← Updated every session
  systemPatterns.md / techContext.md   ← Updated when architecture or stack changes
  productContext.md                    ← Rarely changes
  projectBrief.md                     ← Set once, the foundation
```

## Installation

1. Download or clone this repository
2. In Claude Code, go to **Settings** or use the skill menu
3. Click **Upload a skill**
4. Upload the `memory-bank-skill/` folder (or zip it first)
5. The skill is now available in your Claude Code environment

## Usage

### Initialize Memory Bank (First Time)

Tell Claude:

> "Initialize memory bank"

Claude will:
- Create the `memory-bank/` directory in your project root
- Generate all six files from templates
- Ask you about your project to fill in the details
- If code already exists, analyze it to pre-populate technical files

### Start a Session (Every Time)

Tell Claude:

> "Follow your custom instructions"

Claude will:
- Read all Memory Bank files
- Summarize the current project state
- Pick up exactly where you left off

### Update Memory Bank (After Work)

Tell Claude:

> "Update memory bank"

Claude will:
- Review what changed during the session
- Update `activeContext.md` with current focus and next steps
- Update `progress.md` with completed work and remaining tasks
- Update other files if architecture or tech stack changed

## File Reference

| File | What It Stores | When It Changes |
|------|---------------|-----------------|
| `projectBrief.md` | Project name, scope, requirements, goals | Once at setup |
| `productContext.md` | Why the project exists, target users, UX goals | When product direction shifts |
| `systemPatterns.md` | Architecture, design patterns, key decisions with reasoning | When architecture evolves |
| `techContext.md` | Languages, frameworks, dependencies, dev setup, constraints | When stack changes |
| `activeContext.md` | Current work focus, recent changes, decisions, next steps | **Every session** |
| `progress.md` | Completed features, in-progress work, remaining tasks, bugs | **Every session** |

## Example Workflow

```
Day 1:
  You → "Initialize memory bank"
  Claude → Creates memory-bank/, asks about your project, fills in all files

Day 1 (end of session):
  You → "Update memory bank"
  Claude → Saves what you worked on, decisions made, what's next

Day 2:
  You → "Follow your custom instructions"
  Claude → "Welcome back. Yesterday we set up the authentication module
            using JWT. Next up is the password reset flow. The known issue
            with token expiry on Safari is still open. Ready to continue?"
```

## Tips

- **Let Claude auto-detect:** If a `memory-bank/` directory exists, Claude will suggest reading it automatically
- **Keep files lean:** The skill encourages ~50 lines per file. Link to external docs for details
- **Don't manually edit unless needed:** Let Claude manage the files — it knows the format and hierarchy
- **Works with any project:** JavaScript, Python, Rust, Go — language doesn't matter. It's just markdown

## Project Structure

```
memory-bank-skill/
├── SKILL.md                    # Skill definition with YAML frontmatter + full instructions
└── templates/
    ├── projectBrief.md         # Template for project scope
    ├── productContext.md        # Template for product context
    ├── systemPatterns.md       # Template for architecture patterns
    ├── techContext.md           # Template for tech stack
    ├── activeContext.md         # Template for active context
    └── progress.md              # Template for progress tracking
```

## License

MIT License — use it however you want.

## Contributing

Found a bug or have an idea? Open an issue or submit a pull request.

---
