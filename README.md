# Claude/Copilot Skills

Claude/Copilot skills for development workflows.

## What this repo contains

This repository stores skill definitions and references used to guide agent behavior for specific tasks.

Current skills:
- `google-python-docstrings`: Writes, fixes, and standardizes Python docstrings in Google style.
- `obspy`: Write, debug, and explain Python code using the ObsPy seismology framework.


## Skill details

| Skill | Example request |
|---|---|
| `google-python-docstrings` | "Add Google-style docstrings to all public functions in this file. Include `Args`, `Returns`, and `Raises` sections where applicable, and add a short `Example` for each function." |
| `obspy` | "Download 10 minutes of BHZ waveforms from IRIS for station IU.ANMO after the 2023 Turkey earthquake, remove the instrument response, bandpass filter between 0.05–0.5 Hz, and plot the result." |


## Installing

### Quick install (all skills)

Clone the repo and copy the skills into your Claude Code user skills directory:

**macOS / Linux**
```bash
git clone https://github.com/martanto/skills.git
cp -r skills/skills/* ~/.claude/skills/
```

**Windows (PowerShell)**
```powershell
git clone https://github.com/martanto/skills.git
Copy-Item -Recurse skills\skills\* "$env:USERPROFILE\.claude\skills\"
```

Restart Claude Code — all skills will be available immediately.

### Install a single skill

```bash
git clone https://github.com/martanto/skills.git
cp -r skills/skills/google-python-docstrings ~/.claude/skills/
```

### Keep skills up to date

```bash
cd skills
git pull
cp -r skills/* ~/.claude/skills/
```

### Manual install via `settings.json`

Add the `skills/` folder path directly to your Claude Code `settings.json` (or `settings.local.json` for personal use):

```json
{
  "skills": [
    {
      "type": "directory",
      "path": "/path/to/cloned/skills/skills"
    }
  ]
}
```

To install a single skill:

```json
{
  "skills": [
    {
      "type": "file",
      "path": "/path/to/cloned/skills/skills/google-python-docstrings/SKILL.md"
    }
  ]
}
```

After updating `settings.json`, restart Claude Code for the changes to take effect.

## Contributing new skills

When adding another skill:

1. Create `skills/<skill-name>/SKILL.md`.
2. Add optional references under `skills/<skill-name>/references/`.
3. Keep rules concrete and example-driven.
4. Update this README with the new skill name and purpose.

## License

Add your preferred license information here.
