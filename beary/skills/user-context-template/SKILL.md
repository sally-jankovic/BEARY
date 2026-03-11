---
name: user-context-template
description: Generate or refresh `.agents/skills/beary/USER.md` from a clean template for BEARY research preferences (audience, priorities, source quality rules, and output path). Use when `.agents/skills/beary/USER.md` is missing, incomplete, or the user asks to reset/tune research defaults.
---

# User Context Template

Create or update `.agents/skills/beary/USER.md` using the template at `.agents/skills/beary/templates/user-context-template.md`.

## Rules

- Keep the same section order as the template.
- Preserve explicit user customizations if they already exist.
- If a field is unknown, keep the template default.
- Ensure the output path block includes the `<!-- OUTPUT_PATH: ... -->` line.

## Output

Write a complete `.agents/skills/beary/USER.md` that is human-editable and ready for future runs.
