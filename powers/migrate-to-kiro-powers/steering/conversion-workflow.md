# Conversion Workflow

Complete step-by-step process for converting a Claude Skill into a Kiro Power.

---

## Step 1: Fetch the Skill

When the user provides a GitHub link:

1. Identify the repo and skill directory. Skills live at paths like:
   - `https://github.com/org/skills/tree/main/skills/skill-name`
   - `https://github.com/user/skill-name` (standalone repo)

2. Fetch key files using raw.githubusercontent.com:
   - `SKILL.md` — main documentation (required)
   - `references/*.md` or `reference/*.md` — deep-dive content (both naming conventions exist)
   - `.claude-plugin/marketplace.json` — metadata
   - `CLAUDE.md` — contributor guidelines
   - `LICENSE.txt` — license terms

3. Check for additional directories:
   - `scripts/` — executable code (note in docs, don't bundle)
   - `examples/` — example files (merge into documentation)
   - `assets/` — templates/icons (skip)
   - `evals/` — evaluation harness (skip, Claude-specific)

4. If GitHub fetch fails, ask the user to paste SKILL.md content directly.

---

## Step 2: Analyze the Skill Structure

Before converting, understand what you're working with:

### Parse SKILL.md Frontmatter

Claude Skills use YAML frontmatter:
```yaml
---
name: skill-name
description: When to trigger and what it does
compatibility: Optional tool requirements
---
```

Extract:
- `name` → will become power `name` (ensure kebab-case)
- `description` → refine to max 3 sentences for power `description`

### Parse marketplace.json (if exists)

```json
{
  "name": "Skill Name",
  "description": "What it does",
  "category": "development",
  "tags": ["tag1", "tag2"]
}
```

Extract:
- `name` → use for `displayName`
- `tags` → starting point for `keywords`

### Determine Power Type

- If the skill documents an MCP server → Guided MCP Power (with mcp.json)
- If the skill is pure documentation/workflow → Knowledge Base Power (no mcp.json)
- Default: Knowledge Base Power

### Assess Content Size

- Count total lines across SKILL.md + all reference files
- If under ~500 lines total → single POWER.md, no steering needed
- If over ~500 lines → POWER.md overview + steering files for each reference

---

## Step 3: Generate Metadata

### Power Name
- Convert skill name to kebab-case: `skill_name` → `skill-name`
- Keep it concise and descriptive
- No version numbers

### Display Name
- Title Case version of the name
- Human-friendly: "MCP Builder" not "mcp-builder"

### Description
- Max 3 sentences
- Focus on what it does and why it's useful
- Active voice

### Keywords
Quality check — avoid generic terms:
- ❌ "help", "tool", "code", "debug", "test", "api", "build"
- ✅ "terraform", "react-hooks", "postgresql", "mcp-server", "fastmcp"

Select 5-7 specific terms from the skill's domain.

### Author
- Use the converter's name/username (they created the power)
- Credit original skill author in the attribution section

---

## Step 4: Generate POWER.md

Create the power directory and POWER.md:

### 4.1 Add Frontmatter

```yaml
---
name: "<kebab-case-name>"
displayName: "<Human Readable Name>"
description: "<max 3 sentences>"
keywords: ["<specific>", "<terms>"]
author: "<converter's name>"
---
```

Only these 5 fields. Never use: version, tags, repository, license.

### 4.2 Convert SKILL.md Body

Apply these transformations to the skill content:

**Link updates:**
- `./reference/` or `./references/` → reference to steering files
- `../SKILL.md` → remove or reference main POWER.md
- Relative paths to scripts → note as external dependencies

**Remove Claude-specific content:**
- Skill triggering/description optimization instructions
- Subagent spawning patterns (`spawn a subagent that...`)
- Claude Code CLI references (`claude -p`, `/skill-test`)
- Token budget calculations
- `.claude-plugin` marketplace references
- Evaluation harness instructions (evals.json, grading)
- `present_files` tool references
- Cowork-specific instructions

**Adapt patterns:**
- "Read this reference file" → "See the [steering-file] steering file"
- "Use WebFetch to load URL" → keep the URL, remove the Claude-specific instruction
- Progressive disclosure concept stays the same (POWER.md = always loaded, steering = on-demand)

### 4.3 Add Available Steering Files Section

```markdown
## Available Steering Files
- **filename** - Description of what this file covers
```

List every steering file that will be created from reference docs.

### 4.4 Add Attribution and License Footer

```markdown
## License & Attribution

This power is converted from [skill-name](github-link) by [Original Author] (License-Type).
- [Original Repository](github-link)
- [Privacy Policy](link-if-applicable)
- [Support](github-issues-link)
```

---

## Step 5: Convert References to Steering

For each file in `references/` or `reference/`:

1. Create `steering/<filename>.md`
2. Apply the same content transformations as Step 4.2:
   - Update internal links (`references/` → `steering/`)
   - Remove Claude-specific instructions
   - Remove back-references to `../SKILL.md`
3. Keep all technical content, code examples, and patterns

### Naming Convention
- Keep original filenames but ensure kebab-case
- `mcp_best_practices.md` → `mcp-best-practices.md` (optional, either works)

### Large Reference Files
- If a reference file exceeds ~300 lines, consider splitting into focused topics
- Add a table of contents at the top if keeping as one file

---

## Step 6: Handle Special Files

### CLAUDE.md
If it exists, extract useful content:

**Keep:**
- Content philosophy (what belongs, what doesn't)
- Writing style guidelines
- Quality standards checklists
- Update criteria and maintenance notes

**Discard:**
- Claude-specific installation instructions
- Token budget calculations
- Claude Code marketplace references
- Claude-specific behavioral instructions
- Subagent configuration

Save useful content as `steering/contributing-guidelines.md`. If nothing useful remains, skip entirely.

### scripts/
- Don't bundle scripts in the power
- Note important scripts in POWER.md: "The original skill includes helper scripts for X. These can be found at [original repo link]."
- If scripts are essential to the workflow, describe what they do so users can recreate or find alternatives

### examples/
- Merge small examples inline into POWER.md or relevant steering files
- For large example sets, create a `steering/examples.md`

### LICENSE.txt
- Note the license type in the attribution footer
- If Apache
 2.0 or MIT, the power can use the same or a compatible license

---

## Step 7: Validate

Before finishing, verify:

### Frontmatter
- [ ] POWER.md has valid frontmatter (name, displayName, description required)
- [ ] `name` is lowercase kebab-case
- [ ] `description` is max 3 sentences
- [ ] No non-existent frontmatter fields (no version, tags, repository, license)

### Keywords
- [ ] 5-7 specific keywords, not generic
- [ ] No broad terms like "help", "tool", "code"

### Content
- [ ] All steering files listed in "Available Steering Files" section
- [ ] Internal links updated from `references/` to `steering/`
- [ ] No Claude-specific instructions remain (subagents, triggering, evals)
- [ ] No `../SKILL.md` back-references
- [ ] Code examples and patterns preserved
- [ ] Attribution section present with original author, license, and source link

### Structure
- [ ] POWER.md under ~500 lines (or steering files created for overflow)
- [ ] Each steering file focused on a single topic
- [ ] No mcp.json unless the skill explicitly documents an MCP server
- [ ] No scripts/ or assets/ bundled (Powers are documentation, not code)

---

## Step 8: Output Summary

Tell the user:

```
Power created at: <path>

Files:
- POWER.md: Core documentation with frontmatter
- steering/<files>: Deep-dive content (if applicable)

To install:
- Local: Kiro Powers panel → Add Custom Power → Local Directory → <path>
- Git: Push to a public GitHub repo, then Add Custom Power → Git Repository

To submit to Kiro registry: https://kiro.dev/powers/submit/
```

---

## Common Conversion Patterns

### Simple Skill (SKILL.md only, no references)
```
Input:                    Output:
skill-name/               power-name/
└── SKILL.md              └── POWER.md
```

### Skill with References
```
Input:                    Output:
skill-name/               power-name/
├── SKILL.md              ├── POWER.md
└── references/           └── steering/
    ├── guide-a.md            ├── guide-a.md
    └── guide-b.md            └── guide-b.md
```

### Complex Skill (references + scripts + examples)
```
Input:                    Output:
skill-name/               power-name/
├── SKILL.md              ├── POWER.md
├── CLAUDE.md             └── steering/
├── references/               ├── guide-a.md
│   ├── guide-a.md            ├── guide-b.md
│   └── guide-b.md            └── contributing-guidelines.md
├── scripts/              (scripts noted in POWER.md, not bundled)
├── examples/             (examples merged into docs)
└── LICENSE.txt           (noted in attribution footer)
```

### Skill with MCP Server Documentation
```
Input:                    Output:
skill-name/               power-name/
├── SKILL.md              ├── POWER.md
└── references/           ├── mcp.json          (if MCP server documented)
    └── server-guide.md   └── steering/
                              └── server-guide.md
```
