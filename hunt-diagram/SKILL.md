---
name: hunt-diagram
description: Create a Hunt Integrative Solutions standardized Graphviz→SVG→HTML diagram. Reads HIS-DIAGRAM-STANDARD.md for colors/shapes, generates DOT code, substitutes into template.html, asks where to save, then writes the file.
version: 1.0.0
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [diagramming, graphviz, svg, his-standard, his]
    ported_from: claude-code/hunt-diagram
---
# hunt-diagram

Generate a standardized HIS diagram from a description.

**Invocation:** `/hunt-diagram {type} "{description}"`

**Types:** `plc`, `workflow`, `pipeline`, `network`, `custom`

**Example:** `/hunt-diagram plc "Motor AOI for Pump P-101"`

---

## Instructions

### Step 1 — Parse Arguments

From `$ARGUMENTS`:
- Extract `{type}` (first word): must be one of `plc`, `workflow`, `pipeline`, `network`, `custom`.
- Extract `{description}` (the quoted string or remaining text).
- Derive `{slug}`: lowercase the description, replace spaces and `/` with `-`, strip special chars except `-`.
  - "Motor AOI for Pump P-101" → `motor-aoi-for-pump-p-101`
  - "EtherNet/IP Network Layout" → `ethernet-ip-network-layout`

### Step 2 — Ask for Revision Letter and Save Location

Use `AskUserQuestion` (Claude Code) or `clarify` (Hermes) — one call, two questions — to capture the revision letter and save location. **Never default to Desktop silently — always ask.**

**Hermes fallback:** If neither `AskUserQuestion` nor `clarify` is available in the current execution context, ask inline in the chat reply with the two questions and proceed with defaults (Rev A, Desktop) if the user says "go" or doesn't specify.

```
AskUserQuestion([
  {
    question: "Revision letter for this diagram?",
    header: "Diagram Rev",
    multiSelect: false,
    options: [
      { label: "A — first issue (Recommended for new diagrams)", description: "Use Rev A for first issue. Increment when the diagram changes." },
      { label: "B / C / D / later", description: "Continuing an existing diagram. I'll specify which letter." },
      { label: "Draft — internal working copy", description: "Filename will use 'Draft' instead of a letter." }
    ]
  },
  {
    question: "Where should the diagram HTML be saved?",
    header: "Save Location",
    multiSelect: false,
    options: [
      { label: "Active project folder — I'll type the path (Recommended)", description: "WSL path (e.g., /mnt/c/Users/HIS Controls/Projects/{project}/)." },
      { label: "Same folder as other project documents — I'll type the path", description: "WSL path." },
      { label: "Current working directory", description: "Use the current shell working directory." },
      { label: "Desktop", description: "/mnt/c/Users/HIS Controls/Desktop/ — Windows Desktop via WSL. Use only for one-off / scratch diagrams." }
    ]
  }
])
```

If a "type the path" option was chosen, follow up via Other to capture the path.

Output filename: `hunt-{type}-{slug}-Rev{rev}.html`. Re-running for the same description with the same Rev will warn the user before overwriting; with a different Rev, the new file lands alongside the previous version.

### Step 3 — Load the Standard and Examples

Read `${CLAUDE_SKILL_DIR}/HIS-DIAGRAM-STANDARD.md` before writing any DOT code.

Also read the example files in `${CLAUDE_SKILL_DIR}/examples/` as reference for correct output:
- `graphviz-react-demo.html` — original working demo; reference for sidebar layout, rendering API (`instance()` + `renderString()`), zoom/pan, and export functions.
- `hunt-plc-machine-step-logic-sequencer.html` — first HIS-standard diagram; reference for correct DOT structure, layer colors, AB mnemonics, cluster layout, and edge conventions.

When in doubt about structure or style, read these files — they are the ground truth.

Internalize:
- The five-layer color palette (fill, font, border for each layer).
- Node shape rules for the diagram type.
- Required graph-level attributes (label with company + date, bgcolor, fontname).
- Cluster names appropriate to the diagram type.
- Edge conventions (color, style, penwidth by semantic type).
- Layout direction for this diagram type (rankdir + splines).
- AB naming conventions if type is `plc`.

### Step 4 — Generate DOT Code

Write complete, valid DOT code following the standard exactly:

1. **Graph header** — include ALL required graph-level attributes. Use a **web-safe font stack** so non-Windows render hosts don't fall back to Times:

   ```dot
   digraph HuntDiagram {
       label="Hunt Integrative Solutions LLC\n{description} Rev {rev} — {YYYY-MM-DD}"
       labelloc="t"
       fontname="Inter, Segoe UI, Arial, sans-serif"
       fontsize=14
       fontcolor="#8b949e"
       bgcolor="#0d1117"
       rankdir={TB or LR per standard}
       splines={ortho or curved per standard}
       nodesep=0.6
       ranksep=0.8
       node [fontname="Inter, Segoe UI, Arial, sans-serif" fontsize=11 style="filled,rounded" penwidth=1.5]
       edge [fontname="Inter, Segoe UI, Arial, sans-serif" fontsize=9 color="#8b949e"]
   ```

2. **Clusters** — use standard cluster names for the diagram type. Each cluster:

   ```dot
   subgraph cluster_field {
       label="Field Inputs"
       style=filled
       fillcolor="#0a1a0e"
       color="#238636"
       fontcolor="#3fb950"
       fontname="Inter, Segoe UI, Arial, sans-serif"
       fontsize=11
   ```

3. **Nodes** — every node must have explicit colors from the palette. No default gray nodes:

   ```dot
   P101_Run [label="P-101\nRun CMD" shape=box fillcolor="#0d2112" fontcolor="#3fb950" color="#238636"]
   ```

4. **Edges** — use semantic edge styles per the standard:

   ```dot
   P101_XIC -> P101_OTE [label="XIC P101_Run" color="#3fb950" penwidth=1.5]
   EStop -> SafetyRelay [color="#f85149" penwidth=2.5]
   ```

5. **Minimum node count by type:**
   - `plc`: at least 8 nodes across 3+ clusters
   - `workflow`: at least 6 nodes across 2+ clusters
   - `pipeline`: at least 5 nodes in linear flow
   - `network`: at least 6 nodes across 2+ clusters
   - `custom`: at least 4 nodes

### Step 5 — Substitute into Template

Read `${CLAUDE_SKILL_DIR}/template.html`.

Make exactly **four** substitutions:

| Marker | Replace With |
|---|---|
| `{{PAGE_TITLE}}` | `{description} Rev {rev} — HIS Diagram` |
| `{{DIAGRAM_NAME}}` | `{description}` |
| `{{INITIAL_DOT_CODE}}` | The DOT code, JS-escaped (see below) |
| `{{PRESET_BUTTONS}}` | One preset button HTML (see below) |

**JS-escaping the DOT code** — the DOT string goes inside a JS template literal. Escape:
- All backtick characters `` ` `` → `` \` ``
- All `${` sequences → `\${`
- Do NOT escape regular double quotes (they're fine in template literals).
- Do NOT add surrounding backticks — the template already has them.

**Preset button HTML:**
```html
<button class="preset-btn" onclick="loadPreset('main')">{description}</button>
```

### Step 6 — Write File

Write the substituted HTML to `{save_location}/hunt-{type}-{slug}-Rev{rev}.html` (using the path captured in Step 2).

### Step 7 — Report

After writing, output a summary:
```
Diagram written: {full path}
Type: {type}
Rev: {rev}
Nodes: {count}
Layers: {list of layers used}
Browser: opening automatically (PostToolUse hook)
```

---

## Quality Gates (check before writing)

- [ ] Graph label contains "Hunt Integrative Solutions LLC", the Rev letter, and today's date.
- [ ] All **four** substitution markers replaced (`{{PAGE_TITLE}}`, `{{DIAGRAM_NAME}}`, `{{INITIAL_DOT_CODE}}`, `{{PRESET_BUTTONS}}`) — none remain in output.
- [ ] Every node has explicit `fillcolor`, `fontcolor`, `color` — no bare/unstyled nodes.
- [ ] At least one cluster defined.
- [ ] Safety-related nodes use `shape=octagon` and red palette.
- [ ] For `plc` type: AB mnemonics appear on edge labels.
- [ ] DOT code has no syntax errors (balanced braces, valid attribute syntax).
- [ ] No JS syntax in the final HTML that would break the template literal (unescaped backticks or `${`).
- [ ] All `fontname` directives use the web-safe stack `"Inter, Segoe UI, Arial, sans-serif"` — no bare `Segoe UI` that would fall back to Times on non-Windows hosts.
- [ ] Save location was asked via `AskUserQuestion` — not silently defaulted to Desktop.
