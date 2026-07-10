---
name: skill-creator-reminder
description: Use when creating, adding, writing, or authoring any new workspace skill, or updating/editing an existing skill's structure. Trigger phrases include "make a skill", "create a skill", "add a skill", "build a skill", "new skill", or "store this as a skill." Reminds the agent to always read and follow the skill-creator skill before doing so.
---

# Skill Creator Reminder

Before creating, adding, or editing any workspace skill under `/data_model/skills/`, always read the
`skill-creator` skill in full first:

```bash
cat /skills/skill-creator/SKILL.md
```

Follow its creation workflow, `SKILL.md` frontmatter requirements, resource directory conventions
(`assets/`, `references/`, `scripts/`), and quality bar exactly. Do not freehand a new skill's structure
or frontmatter without consulting `skill-creator` first, even if the new skill seems simple.

This applies every time a new skill is authored or an existing skill is restructured — not just the
first time.
