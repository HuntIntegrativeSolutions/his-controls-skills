# HIS Controls — Industrial Automation Skills

Twelve AI agent skills for generating engineering-grade industrial automation documentation. Built for the [Hermes Agent](https://hermes-agent.nousresearch.com) platform, these skills produce contractually-defensible deliverables that survive chief-engineer review, regulatory audit, and commissioning disputes.

**Every skill cites the governing standard by edition and section.** Your customer's reviewers *will* check — these skills make sure the citations are there.

---

## The 12 Skills

| # | Skill | Deliverable | Primary Standards |
|---|-------|-------------|-------------------|
| 1 | **plc-fds** | Functional Design Specification | ANSI/ISA-5.06.01, ISA-88, IEC 61511 |
| 2 | **plc-soo** | Sequence of Operations | ISA-88, ANSI/ISA-5.06.01, OSHA 1910.119 |
| 3 | **plc-prog-arch** | Program Architecture (Studio 5000) | IEC 61131-3, ISA-88, 1756-PM008 |
| 4 | **plc-alarm-philosophy** | ISA-18.2 Alarm Philosophy | ISA-18.2, EEMUA 191, IEC 62682 |
| 5 | **plc-bom** | Hardware Bill of Materials | ISA-20, NEMA 250, IEEE 315 |
| 6 | **plc-wire-diagram** | Panel Wiring Diagram Package | NFPA 79 §13, ISA-5.4 |
| 7 | **plc-hmi-spec** | HMI Screen Specification | ISA-101, ANSI/ISA-5.06.01 |
| 8 | **plc-panel-layout** | Panel Layout + Power Budget | UL 508A, NFPA 79, NEC 409 |
| 9 | **plc-io-list** | I/O List Template | ISA-5.1, IEC 62443, NEC 504 |
| 10 | **hunt-diagram** | Standardized Graphviz→SVG Diagrams | HIS Diagram Standard |
| 11 | **nexus-walkthrough** | Brownfield PLC Inheritence Runbook | ISA-88, IEC 61131-3 |
| 12 | **plc-impact-trace** | Signal Impact Chain Analysis | IEC 61511, ISA-18.2 |

---

## What These Skills Do

Each skill is a standalone instruction set for an AI agent. When invoked, the skill guides the agent to:

1. **Query live plant data** (via Nexus MCP — a PLC introspection agent) for real tag names, I/O configurations, and interlocks
2. **Apply governing standards by section number** — no generic hand-waving
3. **Produce a complete, sectioned deliverable** in markdown or structured format
4. **Support multiple regulated-industry overlays** — Pharma (21 CFR Part 11), Oil & Gas (OSHA PSM), Water (AWIA), Power (NERC CIP), Food (FSMA)
5. **Support multiple PLC platforms** — Rockwell, Siemens, Beckhoff, and generic

The output is a deliverable with cover block, revision history, standards references, and customer-ready formatting.

---

## How They're Used

These are **Hermes Agent skills** — instruction packs loaded by an AI agent at runtime.

```
# Within a Hermes session with the industrial profile loaded:
/fds "Wastewater Lift Station No. 4"
/soo "Cooling Tower Fan VFD — Area 200"
/alarm-philosophy "Reactor Area 200 — Batch System"
/hunt-diagram plc "Motor AOI for Pump P-101"
```

The agent reads the skill instructions, queries live plant data if available, and produces the deliverable.

---

## Repo Structure

```
his-controls-skills/
├── README.md
├── LICENSE
├── plc-fds/SKILL.md                    # 1,471 lines — Functional Design Specification
├── plc-soo/SKILL.md                    # 1,333 lines — Sequence of Operations
├── plc-prog-arch/SKILL.md              # 1,011 lines — Program Architecture
├── plc-alarm-philosophy/SKILL.md       # 1,782 lines — Alarm Philosophy (largest)
├── plc-bom/SKILL.md                    #   748 lines — Hardware BOM
├── plc-wire-diagram/SKILL.md           #   649 lines — Wiring Diagram
├── plc-hmi-spec/SKILL.md               # 1,193 lines — HMI Screen Spec
├── plc-panel-layout/SKILL.md           #   544 lines — Panel Layout
├── plc-io-list/SKILL.md                #   557 lines — I/O List
├── hunt-diagram/
│   ├── SKILL.md                        #   197 lines — Diagram generator
│   ├── HIS-DIAGRAM-STANDARD.md         # Colors, shapes, conventions
│   ├── template.html                   # Standalone Graphviz→SVG→HTML renderer
│   ├── FullLogo_Transparent_NoBuffer.png
│   ├── examples/
│   │   ├── hunt-plc-machine-step-logic-sequencer.html
│   │   └── graphviz-react-demo.html
│   └── references/font-stack-note.md
├── nexus-walkthrough/SKILL.md          #    78 lines — Brownfield PLC runbook
└── plc-impact-trace/SKILL.md           #    92 lines — Signal impact tracing
```

**Total: ~9,700 lines of engineering instruction across 12 skills.**

---

## Standards Referenced

These skills cite standards by edition and section number:

| Standard | Edition | Skills Using It |
|----------|---------|-----------------|
| ANSI/ISA-5.06.01 | 2007 | plc-fds, plc-soo, plc-hmi-spec |
| ANSI/ISA-88.01 | 2010 | plc-fds, plc-soo, plc-prog-arch, nexus-walkthrough |
| ANSI/ISA-18.2 | 2016 (R2023) | plc-alarm-philosophy, plc-fds, plc-soo |
| ANSI/ISA-101.01 | 2015 (R2018) | plc-hmi-spec, plc-alarm-philosophy |
| IEC 61131-3 | — | plc-prog-arch, nexus-walkthrough |
| IEC 61511-1 | 2016 (Edition 2) | plc-fds, plc-alarm-philosophy, plc-impact-trace |
| IEC 62443 | Parts 2-1, 3-2, 3-3 | plc-fds, plc-io-list |
| ISO 13849-1 | 2023 | plc-fds, plc-soo |
| NFPA 79 | 2024 | plc-wire-diagram, plc-panel-layout, plc-soo |
| NFPA 70 (NEC) | Articles 110.26, 409 | plc-panel-layout |
| UL 508A | — | plc-panel-layout |
| EEMUA 191 | 2013 (Edition 3) | plc-alarm-philosophy |
| IEC 62682 | 2022 (Edition 2) | plc-alarm-philosophy |
| OSHA 1910.119 | — | plc-soo |
| 21 CFR Part 11 | — | plc-alarm-philosophy, plc-fds, plc-soo |

---

## Platform Support

Most skills support conditional content for multiple PLC platforms:

- **Rockwell Automation** — ControlLogix, CompactLogix, GuardLogix, Studio 5000, FactoryTalk View SE/ME
- **Siemens** — TIA Portal, WinCC
- **Beckhoff** — TwinCAT
- **Generic / Other** — Platform-agnostic sections

---

## Regulated Industry Overlays

When relevant, skills emit conditional sections for:

- **Pharma / Biotech** — GAMP 5, 21 CFR Part 11, EU Annex 11, tamper-evident alarm audit trails
- **Oil & Gas / Chemical** — OSHA PSM §(f), CCPS guidelines
- **Water / Wastewater** — AWIA §2013, AWWA standards
- **Power Generation** — NERC CIP-008
- **Food & Beverage** — FSMA, 21 CFR Part 117

---

## Prerequisites

- **Hermes Agent** (or compatible AI agent platform)
- **Nexus MCP** — optional but recommended; provides live PLC data for real tag names, I/O, and interlocks
- **HisMemory MCP** — optional; provides cross-session project memory

---

## About Hunt Integrative Solutions

Hunt Integrative Solutions LLC designs, builds, and commissions industrial control systems for biomass, water/wastewater, food & beverage, pharma, and discrete machine builders. Based in Los Baños, California.

These skills represent the documentation methodology developed over years of delivering projects where safety and compliance matter.

---

## License

MIT — see [LICENSE](LICENSE). Use, modify, and adapt these skills for your own projects and platforms.

The `hunt-diagram/FullLogo_Transparent_NoBuffer.png` is Hunt Integrative Solutions branding and is not covered by the MIT license — replace it with your own for any derivative works.

---

## Contributing

These skills are published as reference implementations. If you adapt them for other platforms (Siemens, Beckhoff, etc.), pull requests adding platform-conditional sections are welcome.
