# Conversion Workflow

Step-by-step process for converting a Claude Skill into a Kiro Power.

---

## Step 1: Fetch the Skill

When the user provides a GitHub link:

1. Fetch the repo contents. Key files to look for:
   - `SKILL.md` — main documentation (required)
   - `references/*.md` — deep-dive content
   - `.claude-plugin/marketplace.json` — metadata (name, description, keywords)
   - `CLAUDE.md` — contributor guidelines
   - `README.md` — can be discarded

2. If the link points to a specific file (e.g., raw SKILL.md), fetch that directly.

3. If GitHub fetch fails, ask the user to paste the content of SKILL.md manually.

---

## Step 2: Determine Power Name and Metadata

Extract metadata from `marketplace.json` if available:
- `name` → convert to kebab-case for power name
- `description` → refine to max 3 sentences
- `keywords` or tags → select 5-7 specific terms

If no marketplace.json:
- Derive name from repo name (kebab-case)
- Extract description from first paragraph of SKILL.md
- Identify keywords from the skill's domain

**Keyword quality check:**
- Avoid generic: "help", "tool", "code", "debug", "test", "api"
- Prefer specific: "terraform", "react-hooks", "postgresql", "kubernetes"
- Generic keywords cause false activations and annoy users

---

## Step 3: Generate POWER.md

Create `~/.kiro/powers/<power-name>/POWER.md`:

1. Add frontmatter at the top (only these 5 fields):
```yaml
---
name: "<kebab-case-name>"
displayName: "<Human Readable Name>"
description: "<max 3 sentences>"
keywords: ["<specific>", "<terms>"]
author: "<from marketplace.json or repo owner>"
---
```

2. Below frontmatter, include the full SKILL.md content with these modifications:
   - Update all `references/` links to `steering/`
   - Remove any `../SKILL.md` back-references
   - Remove Claude-specific instructions (installation, token budgets, marketplace refs)

3. Add "Available Steering Files" section listing all converted reference files:
```markdown
## Available Steering Files
- **filename** - Description of what this file covers
```

4. Add attribution section:
```markdown
## License & Attribution
**Original Work:** Converted from [skill-name](github-link) by [Original Author].
**Source Version:** Based on [commit/tag at time of conversion].
```

---

## Step 4: Convert References to Steering

For each file in `references/`:
1. Copy to `~/.kiro/powers/<power-name>/steering/`
2. Update internal links within each file:
   - `references/` → `steering/`
   - `../SKILL.md` → reference to main POWER.md
3. Remove any Claude-specific content

**Only create steering/ if reference files exist.** If SKILL.md is self-contained and under ~500 lines, no steering directory is needed.

---

## Step 5: Handle CLAUDE.md (Optional)

If CLAUDE.md exists:

**Keep:**
- Content philosophy (what belongs, what doesn't)
- Writing style guidelines
- Quality standards checklists
- Update criteria

**Discard:**
- Claude-specific installation instructions
- Token budget calculations
- Claude Code marketplace references
- Claude-specific behavioral instructions

Save useful content as `steering/contributing-guidelines.md`.

If nothing useful remains after filtering, skip this file entirely.

---

## Step 6: Validate

Before finishing, verify:
- [ ] POWER.md has valid frontmatter (name, displayName, description required)
- [ ] name is lowercase kebab-case
- [ ] Keywords are specific, not generic (no "help", "tool", "code")
- [ ] All steering files listed in "Available Steering Files" section
- [ ] Internal links updated from `references/` to `steering/`
- [ ] No Claude-specific instructions remain
- [ ] Attribution section present with original author and source
- [ ] No non-existent frontmatter fields used (no version, tags, repository, license)

---

## Step 7: Output Summary

Tell the user:

```
Power created at: ~/.kiro/powers/<power-name>/

Files:
- POWER.md: Core documentation with frontmatter
- steering/<files>: Deep-dive content (if applicable)

To install: Kiro Powers panel → Add Custom Power → Local Directory → ~/.kiro/powers/<power-name>
```

---

## Final Structure

```
~/.kiro/powers/<power-name>/
├── POWER.md
└── steering/                    (only if references existed)
    ├── <converted-reference>.md
    └── contributing-guidelines.md  (only if CLAUDE.md had useful content)
```

No mcp.json — this creates Knowledge Base Powers. If the original skill referenced MCP servers, note that in POWER.md but don't auto-generate mcp.json without user input.
