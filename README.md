# Claude/Copilot Skills

Claude/Copilot skills for development workflows.

## What this repo contains

This repository stores skill definitions and references used to guide agent behavior for specific tasks.

Current skill:
- `google-python-docstrings`: Writes, fixes, and standardizes Python docstrings in Google style.

## Repository structure

```text
.
├── README.md
└── skills/
	└── google-python-docstrings/
		├── SKILL.md
		└── references/
			└── rules.md
```

## Skill details

### `google-python-docstrings`

Path: `skills/google-python-docstrings/`

Purpose:
- Standardize Python docstrings using Google-style sections.
- Ensure consistent `Args:`, `Returns:`/`Yields:`, `Raises:`, `Attributes:`, and `Example:` formatting.
- Keep docs API-focused, concise, and practical.

Key references:
- `skills/google-python-docstrings/SKILL.md`
- `skills/google-python-docstrings/references/rules.md`

## How to use

1. Open the target Python file(s) in your editor.
2. Ask the agent to apply the `google-python-docstrings` skill.
3. Provide scope (module, class, function, or file-level) and any project-specific conventions.
4. Validate output against `references/rules.md` before merging.

Example requests:
- "Add Google-style docstrings to this module."
- "Convert mixed docstrings in this file to the repository style."
- "Fill in missing `Args`, `Returns`, and `Example` sections."

## Contributing new skills

When adding another skill:

1. Create `skills/<skill-name>/SKILL.md`.
2. Add optional references under `skills/<skill-name>/references/`.
3. Keep rules concrete and example-driven.
4. Update this README with the new skill name and purpose.

## License

Add your preferred license information here.
