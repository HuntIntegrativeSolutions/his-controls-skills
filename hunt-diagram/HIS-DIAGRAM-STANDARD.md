# HIS Diagram Standard v1.0
# Hunt Integrative Solutions LLC — Graphviz/SVG/HTML Diagram Standard

---

## 1. Tool Selection Guide

| Use Case | Tool | Reason |
|---|---|---|
| PLC ladder logic, AOI diagrams, sequential logic | **Graphviz** | Hierarchical flow, cluster support, precise layout |
| Process flow, pipeline stages, data flow | **Graphviz** | Left-to-right pipelines with labeled edges |
| Network topology, device connectivity | **Graphviz** | Node-edge model matches physical topology |
| Interactive drag-and-drop mockups | React Flow | Only when client needs to rearrange nodes |
| Simple flowcharts in docs/markdown | Mermaid | Documentation-only, not standalone HTML |

**Default: Graphviz → SVG → HTML via template.html**

---

## 2. Color Palette (Semantic Layers)

All colors use hex. No CSS color names.

| Layer | Semantic Meaning | Fill (bgcolor) | Font Color | Border Color |
|---|---|---|---|---|
| **Field / PLC Logic / AOI** | Field devices, rungs, AOI blocks | `#0d2112` | `#3fb950` | `#238636` |
| **Safety / E-Stop / Fault** | Safety relays, e-stops, fault conditions | `#3d1a1a` | `#f85149` | `#da3633` |
| **Comms / Protocol / MCP** | Ethernet/IP, DeviceNet, OPC-UA, MCP servers | `#1a0f2e` | `#d2a8ff` | `#8957e5` |
| **HMI / SCADA / UI** | FactoryTalk, Ignition, operator panels | `#1c2128` | `#79c0ff` | `#388bfd` |
| **Cloud / AI / Decision** | Cloud services, AI agents, decision logic | `#2d2208` | `#ffa657` | `#d29922` |

**Global:**
- Page background: `#0d1117`
- Font family: `"Segoe UI"` on all nodes, edges, and graph labels
- Default edge color: `#8b949e`

---

## 3. Node Shape Rules

| Shape | Graphviz `shape=` | When to Use |
|---|---|---|
| Box (rectangle) | `box` | Standard logic block, AOI, function block |
| Diamond | `diamond` | Decision / branch point |
| Parallelogram | `parallelogram` | Input / Output (signal entering/leaving system) |
| Octagon | `octagon` | Safety / E-Stop node |
| Ellipse | `ellipse` | Start / End state, timer |
| Cylinder | `cylinder` | Data store, historian, database |
| House (inverted) | `invhouse` | HMI screen, operator interface |
| Double circle | `doublecircle` | Final/accepting state in sequence |

---

## 4. Required Graph-Level Attributes

Every diagram MUST include these at the top of the `digraph` block:

```dot
digraph HuntDiagram {
    label="Hunt Integrative Solutions LLC\n{Diagram Title} — {YYYY-MM-DD}"
    labelloc="t"
    fontname="Segoe UI"
    fontsize=14
    fontcolor="#8b949e"
    bgcolor="#0d1117"
    rankdir=TB
    splines=ortho
    nodesep=0.6
    ranksep=0.8
    node [fontname="Segoe UI" fontsize=11 style="filled,rounded" penwidth=1.5]
    edge [fontname="Segoe UI" fontsize=9 color="#8b949e"]
}
```

Adjust `rankdir` and `splines` per Step 7 (Layout Direction Rules).

---

## 5. Standard Cluster Names

### PLC / AOI Diagrams
```
cluster_field      — Field inputs (sensors, switches, field devices)
cluster_safety     — Safety circuit (e-stop chain, safety relay)
cluster_plc        — PLC rack / AOI logic
cluster_outputs    — Coils, drives, actuators
cluster_hmi        — HMI/SCADA layer
```

### Workflow / Pipeline Diagrams
```
cluster_trigger    — What starts the process
cluster_process    — Core steps
cluster_decision   — Branch/approval gates
cluster_output     — Final results, notifications
cluster_error      — Error handling, fallback paths
```

### Network Diagrams
```
cluster_field_net  — Device-level network (DeviceNet, PROFIBUS)
cluster_control_net — Control-level network (ControlNet, EtherNet/IP)
cluster_enterprise — IT / plant-floor integration
cluster_cloud      — Cloud / remote access layer
```

---

## 6. Allen-Bradley Naming Conventions

### Instruction Mnemonics (use as node labels or edge labels)
| Mnemonic | Meaning |
|---|---|
| `XIC` | Examine If Closed (NO contact) |
| `XIO` | Examine If Open (NC contact) |
| `OTE` | Output Energize (standard coil) |
| `OTL` | Output Latch |
| `OTU` | Output Unlatch |
| `OSR` | One-Shot Rising |
| `OSF` | One-Shot Falling |
| `TON` | Timer On-Delay |
| `TOF` | Timer Off-Delay |
| `CTU` | Count Up |
| `CTD` | Count Down |
| `MOV` | Move (data copy) |
| `ADD/SUB/MUL/DIV` | Math instructions |
| `EQU/NEQ/GRT/LES` | Compare instructions |
| `JSR/RET` | Jump to Subroutine / Return |

### AOI Parameter Naming
- Input params: `In_`, `Enable`, `Reset`, `Permissive`
- Output params: `Out_`, `Running`, `Faulted`, `Ready`
- InOut (tag aliases): use the full tag name, e.g., `Drive_P101`
- Boolean tags: `b` prefix → `bRunning`, `bFaulted`
- Real/analog tags: `r` prefix → `rSpeed_SP`, `rSpeed_PV`
- Integer tags: `n` prefix → `nFaultCode`

### Tag Naming Convention
`{Area}_{Device}_{Function}_{Number}`
Example: `PUMP_P101_Run_CMD`, `PUMP_P101_Fault_STS`, `MTR_M201_Speed_PV`

---

## 7. Edge Conventions

| Edge Type | Color | Style | penwidth |
|---|---|---|---|
| Normal / data flow | `#8b949e` | solid | 1.0 |
| Safety / E-Stop path | `#f85149` | solid | 2.5 |
| Seal-in / latch circuit | `#388bfd` | dashed | 1.5 |
| Alarm / fault propagation | `#da3633` | dotted | 1.5 |
| Enable / permissive | `#3fb950` | solid | 1.5 |
| Communication / protocol | `#d2a8ff` | solid | 1.0 |
| HMI read/write | `#79c0ff` | dashed | 1.0 |
| Cloud / AI data path | `#ffa657` | dashed | 1.0 |

Edge label placement: `[label="XIC P101_Run" fontsize=9 fontcolor="#8b949e"]`

---

## 8. Layout Direction Rules

| Diagram Type | `rankdir` | `splines` |
|---|---|---|
| PLC ladder / rung logic | `TB` | `ortho` |
| Sequential process flow | `TB` | `ortho` |
| Pipeline / data flow | `LR` | `curved` |
| Network topology | `LR` | `spline` |
| State machine | `TB` | `spline` |
| AOI internal logic | `TB` | `ortho` |

---

## 9. Output File Naming

Pattern: `hunt-{type}-{slug}.html`

- `{type}`: `plc`, `workflow`, `pipeline`, `network`, `custom`
- `{slug}`: description lowercased, spaces→hyphens, strip special chars
  - "Motor AOI for Pump P-101" → `motor-aoi-for-pump-p-101`
  - "EtherNet/IP Network" → `ethernet-ip-network`

Output location: `/mnt/c/Users/HIS Controls/Desktop/hunt-{type}-{slug}.html`

---

## 10. Diagram Quality Checklist

Before writing the final HTML:
- [ ] Graph label includes "Hunt Integrative Solutions LLC" and date
- [ ] Every node has a layer-appropriate fill/font/border color
- [ ] Every cluster has a `label`, `style=filled`, `color` matching its layer
- [ ] Safety paths use red edges with penwidth=2.5
- [ ] AB mnemonics used correctly on edge labels for PLC diagrams
- [ ] No default gray nodes (every node has explicit `fillcolor`)
- [ ] `fontname="Segoe UI"` on all node/edge/graph attributes
- [ ] File named `hunt-{type}-{slug}.html` on Desktop
