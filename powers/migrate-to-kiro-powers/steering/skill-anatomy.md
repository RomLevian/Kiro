# Claude Skill Anatomy

Deep reference on Claude Skill structure, file types, and patterns. Use this when you need to understand what you're converting.

---

## Skill Directory Structure

```
skill-name/
├── SKILL.md              # Required: main documentation + frontmatter
├── CLAUDE.md             # Optional: contributor/maintenance guidelines
├── README.md             # Optional: user-facing docs (for GitHub)
├── LICENSE.txt           # Optional: license terms
├── .claude-plugin/       # Optional: marketplace metadata
│   └── marketplace.json
├── references/           # Optional: deep-dive docs (loaded on-demand)
│   ├── topic-a.md
│   └── topic-b.md
├── scripts/              # Optional: executable code for tasks
│   ├── helper.py
│   └── build.sh
├── examples/             # Optional: example files
│   └── sample-output/
├── assets/               # Optional: templates, icons, fonts
│   └── template.html
└── evals/                # Optional: evaluation test cases
    └── evals.json
```

Note: some skills use `reference/` (singular) instead of `references/` (plural). Handle both.

## SKILL.md Frontmatter

```yaml
---
name: skill-name
description: When to trigger and what it does. This is the primary triggering mechanism.
compatibility: Optional tool/dependency requirements
---
```

- `name`: identifier, usually kebab-case or snake_case
- `description`: can be long — it's the primary trigger for Claude to decide whether to use the skill. Often includes "Use when..." patterns and explicit trigger phrases.
- `compatibility`: rarely used, lists required tools

## Progressive Disclosure (3 Levels)

Skills use a three-level loading system that maps directly to Powers:

| Level | Skill | Power | When Loaded |
|-------|-------|-------|-------------|
| 1. Metadata | name + description (~100 words) | POWER.md frontmatter | Always in context |
| 2. Main doc | SKILL.md body (<500 lines ideal) | POWER.md body | When power activates |
| 3. Deep docs | references/*.md (unlimited) | steering/*.md | On-demand |

This is the same concept in both systems — the conversion preserves this pattern naturally.

## marketplace.json

```json
{
  "name": "Human Readable Name",
  "description": "What the skill does",
  "category": "development|productivity|creative|other",
  "tags": ["tag1", "tag2", "tag3"],
  "icon": "icon-name"
}
```

Mapping to Power frontmatter:
- `name` → `displayName`
- `description` → `description` (refine to 3 sentences)
- `tags` → `keywords` (filter to 5-7 specific terms)
- `category` → not used in Powers
- `icon` → not used in Powers

## Common Skill Patterns

### Workflow Skills
Structured as phases/steps. Example: mcp-builder has Phase 1-4.
Convert: preserve the phase structure in POWER.md.

### Tool Guide Skills
Document a specific tool with installation, usage, troubleshooting.
Convert: maps directly to Knowledge Base Power pattern.

### Creative Skills
Generate content (art, documents, presentations).
Convert: focus on the methodology and patterns, skip Claude-specific generation instructions.

### File Processing Skills
Handle specific file types (PDF, DOCX, XLSX).
Convert: document the approach and patterns, note that scripts are not bundled.

## Claude-Specific Patterns to Remove

### Subagent Instructions
```
# Remove patterns like:
"Spawn a subagent that..."
"Launch two subagents in the same turn..."
"For each test case, spawn two subagents..."
```

### Triggering/Description Optimization
```
# Remove entire sections about:
- "Description Optimization"
- "How skill triggering works"
- "Generate trigger eval queries"
- run_loop.py / run_eval.py references
```

### Evaluation Harness
```
# Remove:
- evals.json references
- grading.json references
- benchmark.json references
- generate_review.py references
- "Running and evaluating test cases" sections
```

### Claude Code CLI
```
# Remove:
- "claude -p" references
- "/skill-test" references
- "present_files" tool references
```

### Cowork/Claude.ai Specific
```
# Remove entire sections:
- "Cowork-Specific Instructions"
- "Claude.ai-specific instructions"
```

## Patterns to Adapt (Not Remove)

### WebFetch Instructions
```
# Original:
"Use WebFetch to load https://example.com/docs"

# Converted:
"See documentation at: https://example.com/docs"
```

### Reference File Loading
```
# Original:
"Load [📋 Best Practices](./reference/best_practices.md)"

# Converted:
"See the best-practices steering file for detailed guidelines."
```

### Tool-Specific Instructions
```
# Original:
"Use the Read tool to examine the file"

# Converted:
"Examine the file" (generic, tool-agnostic)
```

## Edge Cases

### Skills with Only SKILL.md
Simple conversion — just add frontmatter and clean up content. No steering directory needed.

### Skills with Very Large SKILL.md (>500 lines)
Split into POWER.md (overview + key sections) and steering files (detailed sections). Use the skill's own section headers as natural split points.

### Skills that Reference External URLs
Keep the URLs. Remove the Claude-specific instruction to fetch them. The Kiro agent can fetch URLs on its own.

### Skills with scripts/ That Are Essential
Document what the scripts do in POWER.md. Link to the original repo. Don't bundle them — Powers are documentation packages.

### Community Skills (Non-Anthropic)
Same process. Check the license. Credit the original author. Some community skills may have different directory structures — adapt as needed.

### Skills with MCP Server Documentation
If the skill documents how to build or use an MCP server, consider creating a Guided MCP Power with mcp.json. Ask the user if they want MCP integration or just the documentation.
