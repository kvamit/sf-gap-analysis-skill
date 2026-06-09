# gap-analysis

A multi-phase skill that produces a self-contained, offline-safe tabbed HTML report from any evidence of current-state problems. It walks through three gated phases — Discovery, Design, and Report — with human review checkpoints between each phase.

## When to use

Trigger this skill when the user asks for a:
- Gap analysis or current-state assessment
- Remediation plan or technical debt review
- Design comparison (before/after, options analysis, current vs target state)

Do **not** trigger for:
- A standalone diagram → use `generating-mermaid-diagrams` or `generating-visual-diagrams`
- A targeted code fix without a gap narrative → use the relevant `generating-*` skill
- A formal data lineage document → use `generate_lineage_doc.py`

## Setup

### Global install (available in all projects)

Copy the skill folder into your global Claude commands directory:

```bash
cp -r path/to/gap-analysis ~/.claude/commands/gap-analysis
```

Verify it loaded — restart Claude Code, then run `/gap-analysis`. It should appear in tab-completion.

### Project install (available only in one repo)

1. Copy the skill folder into the project's agents directory:

```bash
cp -r path/to/gap-analysis <project-root>/.agents/skills/gap-analysis
```

2. Create a symlink in `.claude/skills/` so Claude Code discovers it:

```bash
cd <project-root>
ln -s ../../.agents/skills/gap-analysis .claude/skills/gap-analysis
```

3. Restart Claude Code. The skill will appear as `/gap-analysis` when working inside that project.

> **Tip:** Commit `.agents/skills/gap-analysis/` to version control so the whole team gets it. The `.claude/skills/` symlink can be committed too, or added to each developer's local setup.

## Usage

Invoke via `/gap-analysis` in any Claude Code session.

Before proceeding, the skill will ask you for:

**Required**
- **Topic / scope** — e.g. "Analytics pipeline performance", "Flow governance"
- **Raw issues / evidence** — bullet list, CSV, docx, source code, or verbal description
- **Audience** — e.g. CTO, delivery team, client stakeholders
- **Report title and subtitle** — e.g. "Analytics Pipeline Process — Future State Design"

**Recommended**
- **Prepared-by name and date** — e.g. "Amit Vajpeyi, Salesforce CTO · May 2026"

**Optional**
- **Tabs desired** — defaults: Current State / Solution Options / Short-Term Design / Long-Term Design
- **Additional diagrams** — describe components and arrows in plain English
- **Before/after code** — paste actual Apex/SAQL/recipe code

## Phase flow

```
Phase 1 — Discovery
  Ingest evidence → classify each gap by severity (P1/P2/P3) and impact
  Write gap table to docs/<topic-slug>/gaps.md
  ↓ STOP — edit the file, then say "gaps confirmed" or "proceed"

Phase 2 — Design
  Short-term fixes (≤ 4 weeks) + long-term target state
  Solution comparison cards (short-term vs long-term per problem area)
  Write design summary to docs/<topic-slug>/design.md
  ↓ STOP — edit the file, then say "design confirmed" or "proceed"

Phase 3 — Report
  Generate single offline-safe HTML file from approved markdown files
  Output: docs/<topic-slug>/[Topic]_[YYYYMm]_[audience].html
```

## Output

A single HTML file that:
- Opens from `file://` with no internet connection required
- Uses Tailwind CSS (CDN script tag), system fonts only, inline SVG
- Is under 800 KB
- Contains tabbed navigation (primary tabs + optional sub-tabs)

**Default tabs:**
1. Current State — severity-tagged issues table + problem-map SVG
2. Solution Options — short-term vs long-term comparison cards
3. Short-Term Design — remediation steps + before/after code blocks
4. Long-Term Design — target architecture + migration sequence

## Tips

- **Richer issues table** — paste the raw issue list as bullets
- **Architecture diagram** — describe components and arrows in plain English
- **Sub-tabs within a tab** — e.g. "Tab 1 has sub-tabs: Apex Batch Chain, Flows, CRMA Recipes"
- **Executive summary** — add a Tab 0 with an SVG problem map and a 3-bullet impact summary
- **Long content** — generate one tab at a time, then ask to assemble
- **Markdown format** — convert your documents to .md file to save on token size
# sf-gap-analysis-skill
