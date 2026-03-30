---
name: "migrate-to-kiro-powers"
displayName: "Migrate to Kiro Powers"
description: "Bring existing Claude Code Skills into Kiro's power ecosystem. Paste a GitHub link to a Claude Skill and get a ready-to-use Kiro Power with steering files, structured metadata, and best practices applied."
keywords: ["claude", "skill", "power", "convert", "migrate", "kiro"]
author: "RomLevian"
---

# Migrate to Kiro Powers

## Overview

Bring your Claude Code Skills into Kiro and take advantage of Powers' enhanced capabilities — steering files for progressive disclosure, structured frontmatter for discovery, and optional MCP server integration.

The conversion is straightforward because both use markdown-based documentation with progressive disclosure patterns. You provide a GitHub link to a Claude Skill, and the agent fetches, restructures, and creates a ready-to-use Kiro Power.

## Available Steering Files

- **conversion-workflow** - Complete step-by-step conversion process with structural mapping, edge cases, and validation
- **skill-anatomy** - Deep reference on Claude Skill structure, file types, frontmatter, and progressive disclosure patterns

## Structural Mapping

| Claude Skill | Kiro Power | Purpose |
|---|---|---|
| `SKILL.md` | `POWER.md` | Core documentation (frontmatter added) |
| `references/*.md` | `steering/*.md` | Deep-dive content (loaded on-demand) |
| `.claude-plugin/marketplace.json` | POWER.md frontmatter | Metadata (name, description, keywords) |
| `CLAUDE.md` | `steering/contributing-guidelines.md` | Contributor docs (optional, filtered) |
| `scripts/` | Noted in POWER.md | Scripts referenced but not bundled |
| `examples/` | Inline in POWER.md or steering | Examples merged into documentation |
| `assets/` | Not converted | Templates/icons not used by Powers |
| `README.md` | Not needed | Powers UI handles discovery |
| `LICENSE.txt` | Attribution in POWER.md | License noted in footer |

## What Gets Converted

- All technical content from SKILL.md
- All reference documentation (references/ → steering/)
- Code examples, patterns, and decision matrices
- Best practice examples (✅/❌ patterns)
- Workflow steps and phase descriptions

## What Gets Removed

- `.claude-plugin/` directory (metadata moves to frontmatter)
- `README.md` (Powers UI handles discovery)
- `CHANGELOG.md` (not used by Kiro)
- Claude-specific installation/triggering instructions
- Token budget calculations
- Claude Code marketplace references
- Subagent spawning instructions (Kiro uses different patterns)
- `scripts/` directory (noted in docs but not bundled — Powers are documentation, not code)
- `evals/` directory (evaluation harness is Claude-specific)

## What Gets Adapted

- Claude-specific tool references → generic tool descriptions
- `reference/` links → `steering/` links
- Progressive disclosure stays the same concept (POWER.md = always loaded, steering = on-demand)
- Skill description → POWER.md frontmatter `description` field
- Skill name → POWER.md frontmatter `name` field (kebab-case)

## Power Type Decision

Most Claude Skills convert to Knowledge Base Powers (no mcp.json). Only add mcp.json if:
- The skill explicitly documents an MCP server
- The skill requires external tool integration that has a Kiro-compatible MCP server

Default: Knowledge Base Power.

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

Never use: `version`, `tags`, `repository`, `license` — these do not exist in Kiro Power frontmatter.

## Keyword Guidelines

- Include 5-7 specific terms for discovery
- Avoid generic: "help", "tool", "code", "debug", "test", "api"
- Prefer specific: "terraform", "react-hooks", "postgresql", "mcp-server"
- Generic keywords cause false activations and annoy users
- Mix specific and general terms related to the skill's domain

## Steering File Guidelines

- Only create `steering/` if reference files exist or POWER.md exceeds ~500 lines
- Each steering file should focus on a single topic
- List all steering files in POWER.md under "Available Steering Files"
- Steering files are loaded on-demand, not always — keep POWER.md as the primary doc

## Quick Usage

1. Paste a GitHub link to a Claude Skill
2. Say "convert this to a Kiro power"
3. The agent fetches the skill, restructures it, and creates the power
4. Install via Powers panel → Add Custom Power → Local Directory or Git Repository

## Troubleshooting

### GitHub link not accessible
- Ensure the repo is public
- Try the raw.githubusercontent.com URL for individual files
- Ask the user to paste SKILL.md content directly

### Power not showing in Kiro
- Verify POWER.md has valid frontmatter (name, displayName, description required)
- Check the power directory path is correct
- Reload Kiro window or restart

### Content too large for single POWER.md
- If SKILL.md + references exceed ~500 lines total, split into steering files
- Keep overview and common patterns in POWER.md
- Move detailed guides into steering/

### Skill has scripts/ directory
- Note the scripts in POWER.md documentation
- Don't bundle scripts — Powers are documentation packages
- If scripts are essential, suggest the user runs them separately

## License and Support

This power is open source (Apache-2.0). No data is collected or transmitted.
- [Privacy Policy](https://docs.github.com/en/site-policy/privacy-policies/github-general-privacy-statement)
- [Source Repository](https://github.com/RomLevian/Kiro)
- [Support / Issues](https://github.com/RomLevian/Kiro/issues)
