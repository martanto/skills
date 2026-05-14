# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This repository stores **skill definitions** — structured markdown files that guide agent (Claude/Copilot) behavior for specific tasks. Skills are not executable code; they are instruction sets consumed by AI agents.

Use this repository to create new skills, modify and improve existing skills, and measure skill performance. Apply when users want to:
- Create a skill from scratch
- Edit or optimize an existing skill
- Run evals to test a skill
- Benchmark skill performance with variance analysis
- Optimize a skill's `description` field for better triggering accuracy

Reference: [skill-creator/SKILL.md](https://github.com/anthropics/skills/blob/main/skills/skill-creator/SKILL.md)

## Skill Structure

Each skill lives under `skills/<skill-name>/` and follows this layout:

```
skills/<skill-name>/
├── SKILL.md          # Main skill definition (frontmatter + instructions)
└── references/       # Optional supporting detail files
    └── rules.md      # Detailed rules, examples, and validation checklists
```

### `SKILL.md` frontmatter

```yaml
---
name: <skill-name>
description: <one-line trigger description — tells the agent when to invoke this skill>
metadata:
  skill-author: <author>
---
```

The `description` field is critical: it drives automatic skill invocation. Write it as a clear trigger condition (e.g., "Use whenever the user asks to add or improve docstrings...").

## Required Sections in SKILL.md

Every `SKILL.md` must include these sections in order:

1. **Title with one-sentence description** — A top-level heading naming the skill followed by a single sentence explaining what it does.
2. **Overview** — Brief summary of the skill's capabilities and key concepts.
3. **When to Use This Skill** — Must include at least one explicit trigger sentence. Example: "Invoke this skill when the user asks to write docstrings in Google docstring style." Follow with a bulleted list of use cases and a "Do not use" line for out-of-scope scenarios.
4. **Reference Files** — List of supporting files available in `references/`, formatted as a bulleted list. Example:
   - `trace.md`: Description of what it covers.
   - `stream.md`: Description of what it covers.
5. **Additional Resources** *(optional)* — Links to external documentation, tutorials, or APIs relevant to the skill.

## Adding a New Skill

To create a new skill, run the skill creator by invoking:
```
Skill("example-skills:skill-creator")
```
This launches an interactive workflow that guides you through drafting, testing, evaluating, and iterating on the skill.

Manual steps (if not using the skill creator):
1. Create `skills/<skill-name>/SKILL.md` with the frontmatter above.
2. Add optional `skills/<skill-name>/references/` files for detailed rules.
3. Update `README.md` with the new skill's name and purpose.

## Current Skills

- **`google-python-docstrings`** — Writes and standardizes Python docstrings in Google style. References `references/rules.md` for validation.
- **`obspy`** — Write, debug, and explain Python code using the ObsPy seismology framework. References `references/trace.md`, `references/stream.md`, `references/processing.md`, and `references/io.md`.
