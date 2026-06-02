---
name: running-gap-analysis
description: "Multi-phase gap analysis skill producing a self-contained tabbed HTML report. Use when the user needs a structured assessment of current-state problems, solution options, and short/long-term design recommendations for any Salesforce topic. TRIGGER when: user asks for a gap analysis, current-state assessment, remediation plan, design comparison, technical debt review, or says 'assess', 'gap', 'current vs target', 'options analysis', 'before and after'. DO NOT TRIGGER when: user wants a single diagram (use generating-visual-diagrams or generating-mermaid-diagrams), a code change (use the relevant generating-* skill), or a data lineage doc (use a lineage-specific skill)."
license: AMIT
metadata:
  version: "1.0"
---

# running-gap-analysis: Structured Gap Analysis → HTML Report

Use this skill when the user needs a **structured, multi-phase gap analysis** that ends with a self-contained tabbed HTML report.

The skill runs in three phases:
1. **Discovery** — extract and structure the raw gaps from user-provided inputs
2. **Design** — synthesise short-term fixes and long-term target-state recommendations. 
3. **Report** — generate a single offline-safe HTML file using the project's standard design system

---

## Scope

### In Scope
- Any topic where the user has evidence of current-state problems (issues list, CSVs, spreadsheets, docs, org metadata)
- Salesforce or Salesforce-adjacent domains: Apex, Flows, LWC, CRMA/CRM Analytics, OmniStudio, Data Cloud, Agentforce, integrations, security, data model, deployments
- Output: tabbed HTML report (Tailwind CSS, vanilla JS, offline-safe, no external fonts)

### Out of Scope — Delegate Instead
- Standalone diagram only → [generating-mermaid-diagrams](../generating-mermaid-diagrams/SKILL.md) or [generating-visual-diagrams](../generating-visual-diagrams/SKILL.md)
- Targeted code fix without a gap narrative → relevant `generating-*` skill
- Formal data lineage document → `generate_lineage_doc.py` script in project root

---

## Required Inputs (gather before proceeding)

Ask for or infer from the conversation:

| Input | Required? | Notes |
|---|---|---|
| **Topic / scope** | Required | e.g. "Pipeline performance", "Apex batch chain", "Flow governance" |
| **Raw issues / evidence** | Required | Bullet list, CSV, docx excerpt, org metadata, source code or description |
| **Audience** | Required | e.g. Architect, delivery team, client stakeholders |
| **Report title and subtitle** | Required | e.g. "Analytics Pipeline Process — Future State Design · <Customer Name>" |
| **Prepared-by name and date** | Recommended | e.g. "John Doe, Salesforce Architect · May 2026" |
| **Tabs desired** | Optional | Default: Current State  / Solution Options / Short-Term Design / Long-Term Design |
| **Additional diagrams** | Optional | Describe system components and arrows in plain English |
| **Before/after code** | Optional | Paste actual Apex/SAQL/recipe code or Link to source code |

If inputs are missing, ask the user before generating. A bad brief produces a useless report.

---

## Phase 1 — Discovery: Extract and Structure Gaps

### 1.1 Ingest raw evidence

Accept any combination of:
- Bullet-point issue lists (paste inline)
- CSV files (`code-analyzer-results.csv`, `flows_analysis.csv`, defect tracker exports)
- Word/PDF excerpts describing problems
- Verbal description from the user
- Link to source code
- code scans, defect lists, Splunk findings, org assessments, stakeholder pain points

### 1.2 Identify the current state
- Current context and background
- Problem Statement
- Objective

### 1.3 Classify each gap

For every issue identified, assign:

| Field | Values |
|---|---|
| **#** | Sequential integer |
| **Issue** | Short name (≤ 8 words) |
| **Source / Location** | Apex class, Flow name, LWC, CRMA recipe, dataset, object, etc. |
| **Severity** | P1 Critical / P2 High / P3 Medium |
| **Detail** | One-paragraph root-cause description |
| **Business Impact** | Quantified where possible (pipeline delay, data staleness, user friction) |
| **Fix / Solution** | One-line pointer to the recommended fix |

### 1.4 Output: Write gap table to a markdown file for editing

Write the structured gap table to a file at `docs/<topic-slug>/gaps.md` (create the directory if needed), using this format:

```markdown
## Gap Analysis: [Topic] — [Date]

<!-- Edit this table directly before telling Claude to proceed. Add rows, adjust severities, correct impacts. -->

## Issue [N]: [Short title, ≤ 6 words]

**Issue Detail:**
**Source / Location:**
**Severity:**
**Business Impact:**
[One sentence. What fails, degrades, or costs money. Quantify where evidence supports it.]
```

**Stop here.** Tell the user:

> "I've written the gap table to `docs/<topic-slug>/gaps.md`. Open the file, edit it directly — add rows, adjust severities, fix wording — then come back and say **'gaps confirmed'** (or just **'proceed'**) and I'll move to Phase 2."

Do NOT proceed to Phase 2 until the user explicitly confirms. When they do, re-read the file from disk before continuing — use the current file contents, not what was written in this session.

---

## Phase 2 — Design: Synthesise Recommendations

### 2.1 Short-term fixes (≤ 4 weeks, no architectural change)

For each P1/P2 gap, describe:
- Exact change required (file, recipe, query, config)
- Effort estimate (hours/days)
- Risk if left unfixed

### 2.2 Long-term target state (architectural redesign)

Describe:
- Target architecture (in plain English + component list for SVG diagram)
- Which gaps this resolves permanently
- Dependencies and sequencing
- Migration / cutover considerations

### 2.3 Solution comparison cards

For each major problem area produce a **Short-Term vs Long-Term** comparison:
- Short-term: What changes, what stays broken, risk
- Long-term: Full resolution, cost/effort, prerequisite steps

### 2.4 Output: Write design summary to a markdown file for editing

Append the design summary (short-term fixes, long-term target state, comparison cards) to `docs/<topic-slug>/design.md` (same directory as `gaps.md`).

**Stop here.** Tell the user:

> "I've written the design summary to `docs/<topic-slug>/design.md`. Review and edit it — adjust effort estimates, add missing steps, remove anything inaccurate — then say **'design confirmed'** (or **'proceed'**) and I'll generate the HTML report."

Do NOT generate the HTML until the user explicitly confirms. When they do, re-read both `gaps.md` and `design.md` from disk — use the current file contents as the source of truth for the report.

---

## Phase 3 — Report: Generate Offline-Safe HTML

Once the gap table and design are confirmed, generate the HTML report using the design system below.

### HTML Design System (mandatory — do not deviate)

**Framework:** Tailwind CSS via CDN script tag only:
```html
<script src="https://cdn.tailwindcss.com"></script>
```

**Tailwind custom colours:**
```js
colors: {
  navy:  '#002D62',
  blue:  '#0051A5',
  sky:   '#E8F2FF',
  slate: '#4A5568',
  border:'#CBD5E0',
  red:   '#C53030',
  amber: '#B45309',
  green: '#276749',
}
```

**Font — system stack ONLY (mandatory):**
```css
body { font-family: 'Segoe UI', system-ui, -apple-system, Arial, sans-serif; }
code, .font-mono { font-family: 'Cascadia Code', Consolas, 'Courier New', monospace; }
```
- No `<link>` to Google Fonts or any external font service
- No `@import url(...)` for fonts
- No base64-encoded font blobs (triggers Windows Defender)

**Tailwind fontFamily config:**
```js
fontFamily: {
  sans: ['Segoe UI', 'system-ui', 'Arial', 'sans-serif'],
  mono: ['Cascadia Code', 'Consolas', 'Courier New', 'monospace'],
}
```

**Tab behaviour (vanilla JS — no frameworks):**
```js
// Primary tabs
document.querySelectorAll('.tab-btn').forEach(btn => {
  btn.addEventListener('click', () => {
    document.querySelectorAll('.tab-btn').forEach(b => b.classList.remove('active'));
    document.querySelectorAll('.tab-panel').forEach(p => p.classList.remove('active'));
    btn.classList.add('active');
    document.getElementById('tab-' + btn.dataset.tab).classList.add('active');
  });
});
// Sub-tabs
document.addEventListener('click', e => {
  const btn = e.target.closest('.sub-tab-btn');
  if (!btn) return;
  const group = btn.dataset.group;
  document.querySelectorAll(`.sub-tab-btn[data-group="${group}"]`).forEach(b => b.classList.remove('active'));
  document.querySelectorAll(`.sub-panel[data-group="${group}"]`).forEach(p => p.classList.remove('active'));
  btn.classList.add('active');
  const target = document.getElementById(btn.dataset.target);
  if (target) target.classList.add('active');
});
```

**CSS tab state:**
```css
.tab-btn.active { border-bottom: 3px solid #0051A5; color: #0051A5; font-weight: 600; }
.tab-panel { display: none; }
.tab-panel.active { display: block; }
.sub-tab-btn.active { background: #0051A5; color: #fff; }
.sub-panel { display: none; }
.sub-panel.active { display: block; }
@media print { .no-print { display: none; } }
```

**Layout:** `max-w-screen-xl mx-auto px-6`. Sticky header + nav with `z-10`.

**Severity badges:**
- P1 Critical: `bg-red-100 text-red px-2 py-0.5 rounded-full font-semibold`
- P2 High: `bg-amber-100 text-amber px-2 py-0.5 rounded-full font-semibold`
- P3 Medium: `bg-green-100 text-green px-2 py-0.5 rounded-full font-semibold`

**Problem/solution cards:** Dark navy `bg-navy text-white rounded-xl p-6` for summaries. White bordered `bg-white border border-border rounded-lg` for detail tables.

**Code blocks:** `bg-gray-900` background. Comments `#6b7280`, removed lines `#f87171`, added lines `#86efac`, default `#a5f3fc`.

**Tables:** Header `bg-navy text-white`. Alternating hover `hover:bg-gray-50`. Dividers `divide-y divide-gray-100`.

### Default Tab Structure

```
Tab 1 — Current State
  - Problem-map SVG diagram (describe components in plain English; generate inline SVG)
  - Severity-tagged issues table (all gaps from Phase 1)
  - Sticky jump-nav within the tab for long issue lists

Tab 2 — Solution Options
  - Short-term vs long-term comparison cards (one per major problem area)
  - Effort/risk matrix table

Tab 3 — Short-Term Design
  - Architecture diagram showing immediate changes (inline SVG)
  - Step-by-step remediation plan
  - Before/after code blocks where applicable

Tab 4 — Long-Term Design
  - Target-state architecture diagram (inline SVG)
  - Migration/cutover sequence
  - Dependencies and open decisions
```

Customise tabs as needed. Sub-tabs are supported within any primary tab.

### Offline Safety Checklist (self-check before outputting)

Search the generated HTML for these strings — if any are found, remove them and regenerate that section:

- `fonts.googleapis.com`
- `fonts.gstatic.com`
- `base64`
- `@import url`
- Any `<link rel="stylesheet">` other than the Tailwind CDN script tag

Target file size: **under 800 KB**. Exceeding this usually means a font blob leaked in.

### Output Format

- Single complete HTML file in one code block
- No explanatory text before or after the code block
- Filename: `[Topic]_[YYYYMm]_[audience].html` e.g. `Apex_Pipeline_2026May.html`
- Must open correctly from `file://` with no internet required

---

## Tips for Best Results

| Goal | What to provide |
|---|---|
| Richer issues table | Paste the raw issue list as bullets — the skill formats it |
| Architecture diagram | Describe components and arrows in plain English |
| Code before/after | Paste actual LWC/Apex/SAQL/recipe code with "show before and after" |
| Sub-tabs within a tab | List them: "Tab 1 has sub-tabs: Apex Batch Chain, Flows, CRMA Recipes" |
| Custom colour scheme | Override the Tailwind config colours section with brand hex values |
| Long content | Generate one tab at a time, then ask to assemble |
| Executive summary tab | Add a Tab 0 with an SVG problem map and a 3-bullet impact summary |

---
