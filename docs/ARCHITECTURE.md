# BEARY Architecture: Skill vs Workflow

This document explains how BEARY works as both an installable skill and a standalone workflow.

## Dual-Purpose Design

BEARY serves two audiences:

1. **End Users**: Install BEARY as a skill and summon it to perform research
2. **Developers**: Clone/fork the repo to understand, modify, or extend BEARY's capabilities

## As a Skill

When installed via `npx skills add sally-jankovic/BEARY`, the `beary/` directory is copied to the user's project at `.agents/skills/beary/`.

### Skill Entry Point

`beary/SKILL.md` is the main entry point. It defines:

```yaml
---
name: beary
description: Topic-to-whitepaper research workflow with citations
---
```

### Invocation

Users summon BEARY by typing `BEARY` in their prompt. The skill:

1. Detects the summon using `scripts/is-beary-summon.sh`
2. Displays the ASCII logo from `LOGO.md`
3. Runs the `research` command workflow

### Skill Composition

BEARY is composed of smaller, reusable skills:

```
beary/SKILL.md (main skill)
    ↓
commands/research.md (orchestrator)
    ↓
├─ skills/internet-research/SKILL.md
├─ skills/references/SKILL.md
├─ skills/whitepaper-writing/SKILL.md
└─ skills/user-context-template/SKILL.md
```

Each sub-skill can be invoked independently or as part of the larger workflow.

## As a Workflow

The `research` command (`beary/commands/research.md`) orchestrates the end-to-end process:

### Workflow Phases

```
┌─────────────────────────────────────────┐
│ Phase 0: Setup & Configuration          │
│ - Gather topic details                  │
│ - Select mode (HIBERNATION/HYPERPHAGIA) │
│ - Choose attendance (ATTENDED/UNATTENDED)│
└─────────────────────────────────────────┘
                   ↓
┌─────────────────────────────────────────┐
│ Phase 1: User Context                   │
│ - Read or bootstrap USER.md             │
│ - Understand audience & preferences     │
└─────────────────────────────────────────┘
                   ↓
┌─────────────────────────────────────────┐
│ Phase 2: Internet Research              │
│ - Create workspace in beary-scratchpad/ │
│ - General Understanding questions       │
│ - Deeper Dive questions                 │
│ - Review & synthesize notes             │
└─────────────────────────────────────────┘
                   ↓
┌─────────────────────────────────────────┐
│ Phase 3: Checkpoint (ATTENDED only)     │
│ - Present findings summary              │
│ - Wait for user approval                │
└─────────────────────────────────────────┘
                   ↓
┌─────────────────────────────────────────┐
│ Phase 4: Whitepaper Writing             │
│ - Read notes & references               │
│ - Create outline                        │
│ - Write sections with citations         │
│ - Review for clarity                    │
└─────────────────────────────────────────┘
                   ↓
┌─────────────────────────────────────────┐
│ Phase 5: Delivery                       │
│ - Move outputs to configured directory  │
│ - Present final deliverables            │
└─────────────────────────────────────────┘
```

### Context Passing

The workflow passes context between phases without re-reading:

- **USER.md**: Read once in Phase 1, used throughout
- **MODE**: Selected in Phase 0, controls research behavior in Phase 2
- **ATTENDED**: Selected in Phase 0, determines checkpoint behavior in Phase 3

This minimizes token usage and avoids redundant operations.

## File Structure

### Installation Structure (User's Project)

```
user-project/
├── .agents/
│   └── skills/
│       └── beary/              # Installed skill
│           ├── SKILL.md
│           ├── commands/
│           ├── skills/
│           └── templates/
├── beary-scratchpad/           # Temporary workspace (gitignored)
│   └── {TOPIC}/
│       ├── notes/
│       └── whitepaper/
└── whitepaper-output/          # Final deliverables
    └── {TOPIC}/
        ├── notes/
        └── whitepaper/
```

### Development Structure (This Repo)

```
sce-background-research/
├── README.md                   # User-facing documentation
├── CONTRIBUTING.md             # Developer guide
├── docs/
│   └── ARCHITECTURE.md         # This file
├── beary/                      # The skill package
│   ├── SKILL.md                # Main entry point
│   ├── AGENTS.md               # Agent behavior rules
│   ├── USER.md                 # User preferences template
│   ├── LOGO.md                 # ASCII art
│   ├── commands/
│   │   └── research.md         # Main workflow
│   ├── skills/
│   │   ├── internet-research/
│   │   ├── references/
│   │   ├── whitepaper-writing/
│   │   └── user-context-template/
│   ├── templates/              # File templates
│   └── scripts/                # Helper scripts
├── examples/                   # Sample outputs
└── tools/                      # Development utilities
```

## Key Design Principles

### 1. Modularity

Each skill is self-contained with its own `SKILL.md`. Skills can be:
- Used independently
- Composed into larger workflows
- Modified without affecting other skills

### 2. Template-Driven

All generated files follow templates in `beary/templates/`:
- Ensures consistency
- Makes output predictable
- Simplifies modifications

### 3. Mode-Based Behavior

HIBERNATION and HYPERPHAGIA modes control:
- Number of research questions
- Search terms per question
- Sufficiency checks
- Subtopic handling

This allows users to balance quality vs token cost.

### 4. Workspace Isolation

Research happens in `beary-scratchpad/` (gitignored) to:
- Keep the project clean
- Allow iterative work
- Separate drafts from deliverables

Final outputs move to a configurable directory (default: `whitepaper-output/`).

### 5. User Preferences

`USER.md` captures:
- Target audience (technical level, role)
- Purpose style (exploration vs decision support)
- Source priorities and exclusions
- Freshness preferences
- Output path configuration

This personalizes the research without requiring per-invocation configuration.

## Execution Model

### Agent Instructions

BEARY is designed for AI coding assistants (like Cascade/Windsurf). The workflow files contain:

1. **Structured steps**: Numbered, sequential instructions
2. **Turbo annotations**: `// turbo` marks safe-to-auto-run commands
3. **Conditional logic**: Mode-specific branches (HIBERNATION vs HYPERPHAGIA)
4. **File references**: Explicit paths to templates and skills

### Command Execution

The workflow uses shell scripts for specific tasks:
- `scripts/is-beary-summon.sh`: Detects exact "BEARY" invocation
- Future scripts can add validation, formatting, or automation

### File Operations

The agent performs:
- Directory creation (`mkdir -p`)
- File creation from templates
- Content editing (research notes, citations, whitepapers)
- File moving (workspace → output directory)

## Extending the Architecture

### Adding a New Phase

To add a workflow phase:

1. Add a new step in `beary/commands/research.md`
2. Create a new skill in `beary/skills/new-phase/SKILL.md`
3. Reference it in the workflow step
4. Update this architecture doc

### Adding a New Mode

To add a research mode (e.g., "BALANCED"):

1. Define mode parameters in `beary/skills/internet-research/SKILL.md`
2. Add mode selection option in `beary/commands/research.md` step 0.2
3. Implement mode-specific logic in research steps 4.1 and 4.2

### Supporting New Output Formats

To support formats beyond Markdown:

1. Create new templates in `beary/templates/`
2. Add format selection to `USER.md` template
3. Modify `beary/skills/whitepaper-writing/SKILL.md` to handle format
4. Update file extensions in workflow steps

## Skill Installation Mechanics

When `npx skills add sally-jankovic/BEARY` runs:

1. The package manager clones this repo
2. Copies `beary/` to `.agents/skills/beary/` in the user's project
3. The AI assistant reads `.agents/skills/beary/SKILL.md` to understand capabilities
4. User can now invoke BEARY in their prompts

The skill is self-contained—all dependencies (templates, scripts, sub-skills) are in the `beary/` directory.

## Why This Structure?

This dual-purpose design provides:

- **For users**: Simple installation, clear invocation, minimal configuration
- **For developers**: Clear architecture, modular components, easy extension
- **For maintainers**: Single source of truth, testable components, documented behavior

The skill/workflow distinction allows BEARY to be both a tool (for end users) and a reference implementation (for developers building similar agentic workflows).
