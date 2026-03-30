---
name: "skill-to-power"
displayName: "Skill to Power"
description: "Converts Claude Code Skills into Kiro Powers. Paste a GitHub link to a Claude Skill and this power will fetch it, restructure it, and create a ready-to-use Kiro Power."
keywords: ["claude", "skill", "power", "convert", "migrate", "kiro"]
author: "RomLevian"
---

# Claude Skill to Kiro Power Converter

## Overview

This power helps you convert Claude Code Skills into Kiro Powers. Claude Skills are markdown-based knowledge packages for Claude Code, while Kiro Powers are enhanced packages that can additionally bundle MCP servers, steering files, and structured metadata.

The conversion is straightforward because both use markdown-based documentation. You provide a GitHub link to a Claude Skill, and this power guides the agent through fetching, restructuring, and creating a ready-to-use Kiro Power.

## Available Steering Files

- **conversion-workflow** - Complete step-by-step conversion process including fetching, mapping, and generating power files

## Structural Mapping

| Claude Skill | Kiro Power | Purpose |
|---|---|---|
| SKILL.md | POWER.md | Core documentation (with frontmatter added) |
| references/*.md | steering/*.md | Deep-dive content (loaded on-demand) |
| .claude-plugin/marketplace.json | POWER.md frontmatter | Metadata (name, description, keywords) |
| CLAUDE.md | steering/contributing-guidelines.md | Contributor docs (optional, filtered) |
| README.md | Not needed | Powers UI handles discovery |
| CHANGELOG.md | Not needed | Not used by Kiro |

## What Gets Removed

- `.claude-plugin/` directory (metadata moves to frontmatter)
- `README.md` (Powers UI handles discovery)
- `CHANGELOG.md` (not used by Kiro)
- Claude-specific installation instructions
- Token budget calculations
- Claude Code marketplace references

## What Gets Kept

- All technical content from SKILL.md
- All reference documentation
- Code examples and patterns
- Decision matrices and flowcharts
- Best practice examples (✅/❌ patterns)

## Frontmatter Rules

Only these 5 fields exist in POWER.md frontmatter:

```yaml
---
name: "kebab-case-name"           # Required: lowercase kebab-case
displayName: "Human Readable Name" # Required: Title Case
description: "Max 3 sentences"     # Required: what it does and why
keywords: ["specific", "terms"]    # Optional: 5-7 specific terms
author: "Author Name"              # Optional: creator name
---
```

Never use: `version`, `tags`, `repository`, `license` — these fields do not exist.

## Quick Usage

1. Paste a GitHub link to a Claude Skill
2. Say "convert this to a Kiro power"
3. The agent fetches the skill, restructures it, and creates the power at `~/.kiro/powers/<power-name>/`
4. Install via Powers panel → Add Custom Power → Local Directory

## Troubleshooting

### GitHub link not accessible
- Ensure the repo is public
- Try pasting the raw content of SKILL.md directly instead

### Power not showing in Kiro
- Verify POWER.md has valid frontmatter (name, displayName, description are required)
- Check the power directory path is correct
- Restart Kiro if needed

### Keywords too generic
- Avoid broad terms like "help", "tool", "code"
- Use specific domain terms from the skill's content
- Generic keywords cause false activations
