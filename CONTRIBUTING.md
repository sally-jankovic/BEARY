# Contributing to BEARY

BEARY is an agentic workflow for background research that produces cited whitepapers. This guide explains the architecture and how to extend or modify it.

## Architecture Overview

BEARY is structured as a **modular skill system** with a command-based workflow:

```
beary/
├── SKILL.md                 # Main skill entry point
├── AGENTS.md                # Agent behavior rules
├── USER.md                  # User preferences template
├── commands/                # Workflow orchestration
│   └── research.md          # Main research-to-whitepaper workflow
├── skills/                  # Modular sub-skills
│   ├── internet-research/
│   ├── references/
│   ├── whitepaper-writing/
│   └── user-context-template/
├── templates/               # File templates for research artifacts
└── scripts/                 # Helper scripts
```

### Core Concepts

#### 1. Skills vs Workflows

- **Skill**: A reusable capability defined in a `SKILL.md` file. Skills can be composed together.
- **Workflow**: A sequence of steps that orchestrates multiple skills to achieve a goal.

BEARY is both:
- **As a skill**: Installable via `npx skills add sally-jankovic/BEARY`
- **As a workflow**: The `research` command orchestrates internet-research → references → whitepaper-writing

#### 2. Modular Sub-Skills

Each sub-skill is self-contained and can be used independently:

| Skill | Purpose | Location |
|-------|---------|----------|
| `internet-research` | Systematic web research with citation tracking | `beary/skills/internet-research/` |
| `references` | Citation format and bibliography management | `beary/skills/references/` |
| `whitepaper-writing` | Transform notes into structured whitepapers | `beary/skills/whitepaper-writing/` |
| `user-context-template` | Bootstrap user preferences | `beary/skills/user-context-template/` |

#### 3. Research Modes

BEARY supports two token-management strategies:

- **HIBERNATION**: Conservative mode (2 questions, 2 search terms, sufficiency checks)
- **HYPERPHAGIA**: Generous mode (3-4 questions, 3 search terms, no early stopping)

These modes are passed through the workflow and control behavior in the `internet-research` skill.

#### 4. File Organization

Research happens in a temporary workspace (`beary-scratchpad/{TOPIC}/`) and final outputs move to a configurable output directory (default: `whitepaper-output/`).

```
beary-scratchpad/{TOPIC}/
├── notes/
│   ├── {TOPIC}-research-questions.md
│   └── {TOPIC}-notes.md
└── whitepaper/
    ├── {TOPIC}-whitepaper.md
    └── {TOPIC}-references.md
```

## How to Extend BEARY

### Adding a New Sub-Skill

1. Create a new directory under `beary/skills/your-skill-name/`
2. Add a `SKILL.md` file with frontmatter:
```markdown
---
name: your-skill-name
description: What this skill does
license: MIT
compatibility: Any requirements
metadata:
  author: your-name
  version: "1.0"
---

# Your Skill Name

[Skill documentation here]
```

3. Reference it in `beary/commands/research.md` or create a new command workflow

### Modifying Research Behavior

The research logic lives in `beary/skills/internet-research/SKILL.md`. Key sections:

- **Step 4.1**: General Understanding questions (initial research)
- **Step 4.2**: Deeper Dive questions (in-depth research)
- **Step 5**: Review and synthesis

To adjust question counts, search terms, or sufficiency checks, edit the mode-specific instructions in these sections.

### Adding New Templates

Templates live in `beary/templates/` and define the structure of research artifacts:

- `topic-research-questions.md`: Question organization
- `topic-notes.md`: Note-taking format
- `whitepaper.md`: Whitepaper structure
- `references.md`: Bibliography format

To add a new template:
1. Create the template file in `beary/templates/`
2. Reference it in the appropriate skill's `SKILL.md`

### Creating New Commands

Commands are workflow orchestrators in `beary/commands/`. To create a new command:

1. Create `beary/commands/your-command.md`
2. Use YAML frontmatter:
```markdown
---
description: What this command does
---

# Your Command Name

## Workflow Steps

### 1. First Step
[Instructions]

### 2. Second Step
[Instructions]
```

3. Reference it in `beary/SKILL.md`

## Development Guidelines

### Agent Behavior Rules

`beary/AGENTS.md` contains critical rules for AI agents. Key principles:

- **No unsolicited documentation**: Don't create files unless explicitly asked
- **Preserve user text**: When editing markdown, keep original text intact
- **No emojis**: Keep output professional
- **Be concise**: Minimize verbose explanations

### Testing Changes

When modifying BEARY:

1. Test in a clean directory with `npx skills add` (if testing as a skill)
2. Run through both HIBERNATION and HYPERPHAGIA modes
3. Verify file structure matches templates
4. Check citation accuracy in generated references

### File Naming Conventions

- Skills: `SKILL.md` (uppercase)
- Commands: `command-name.md` (lowercase with hyphens)
- Templates: `template-name.md` (lowercase with hyphens)
- Agent rules: `AGENTS.md` (uppercase)

## Common Modification Scenarios

### Adjusting Token Usage

Edit mode parameters in `beary/skills/internet-research/SKILL.md`:
- Question counts (lines 33-43, 101-109)
- Search terms per question
- Sufficiency check logic (line 89)

### Changing Output Structure

Modify templates in `beary/templates/`:
- `whitepaper.md`: Adjust section headers and structure
- `topic-notes.md`: Change note organization

### Adding Source Filters

Edit `beary/skills/user-context-template/SKILL.md` to add new preference questions about source types, recency, or domains.

## Workflow Execution Flow

```
User summons BEARY
    ↓
Check USER.md exists (bootstrap if needed)
    ↓
Gather topic details + mode selection
    ↓
Create workspace in beary-scratchpad/
    ↓
Internet Research Skill
    ├─ General Understanding (2-4 questions)
    ├─ Deeper Dive (1-3 questions or subtopics)
    └─ Review & Synthesize
    ↓
[Optional] User review checkpoint (ATTENDED mode)
    ↓
Whitepaper Writing Skill
    ├─ Read notes & references
    ├─ Create outline
    ├─ Write sections with citations
    └─ Review for clarity
    ↓
Move outputs to configured directory
    ↓
Final delivery
```

## Best Practices

1. **Keep skills modular**: Each skill should work independently
2. **Use templates consistently**: All generated files should follow templates
3. **Respect token modes**: HIBERNATION should genuinely conserve tokens
4. **Maintain citation accuracy**: References must be real and correctly formatted
5. **Test end-to-end**: Changes to one skill may affect downstream workflows

## Questions or Issues?

For questions about extending BEARY or contributing changes, open an issue in the repository with:
- What you're trying to accomplish
- Which files you've modified
- Any error messages or unexpected behavior
