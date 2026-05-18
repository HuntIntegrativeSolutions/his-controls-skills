---
name: nexus-walkthrough
description: Step-by-step runbook for inheriting a brownfield PLC into HIS Controls' Nexus index. Use when Boss says "I just got handed a new site" or "we're bidding on a migration" or "this PLC is undocumented" — anything that means we need a full inventory + understanding of an existing system before quoting or modifying. Chains the right Nexus tools in the right order so nothing gets missed.
version: 1.0.0
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [nexus, plc, brownfield, migration, audit, his]
---

# /nexus-walkthrough — Brownfield PLC inheritance runbook

**Invocation:** `/nexus-walkthrough` (then I'll ask for the site name and access method)

---

## Purpose

Boss inherited a PLC he didn't write. Before anyone touches it — quote, modify, migrate, document — we need a complete picture of what's there and what's already indexed in Nexus. This skill walks the right tools in the right order so we don't quote blind or modify something safety-critical without understanding it.

The deliverable is a one-page handoff sheet: what we know, what we don't know, what's risky, what's reusable.

---

## Sequence

### Step 1 — What does Nexus already know?

Call **`plant_summary`** first — it's free and tells us what site(s) Nexus has indexed. Then **`list_documented_plcs`** to see which controllers have been onboarded. If our target site appears: jump to Step 3. If not: continue to Step 2 (the harder path).

If you see PLCs you don't recognize, ask Boss whether they belong to the active site or are inherited from a prior install.

### Step 2 — Onboard the PLC

If the site is brand new to Nexus:

1. Ask Boss for the source files: usually an `.ACD` (Studio 5000), `.L5X` export, `.PRT` (PLC-5/SLC) printout, or a FactoryTalk View `.APA` for the HMI side.
2. Call **`onboard_plc`** with the path. This ingests the program, tags, and rung text into the index. Expect a few minutes for a large program.
3. After it returns, call **`full_plc_documentation`** to produce the complete docs + diagrams + knowledge graph. Save the output path to memory.

### Step 3 — Walk the inventory

Now Nexus has the data. Pull the artifacts that always matter:

1. **`generate_io_inventory`** — the trust-level-tagged physical I/O list. Look for `unverified` entries; those need a field check before commissioning.
2. **`generate_sequence_of_operations`** — extracted from the rung logic. Boss will compare it to whatever spec we have on paper.
3. **`generate_network_topology`** — Ethernet/IP, DLR rings, NAT, switch layout. Confirm against the actual rack hardware.
4. **`hmi_project_summary`** if FactoryTalk / Ignition is in scope.

### Step 4 — Find the bombs

Before quoting any modification:

1. **`find_area_interlocks`** for each safety-relevant area — surface the interlock chains. These are the rungs that hurt people if you change them wrong.
2. **`tag_suspect_descriptions`** — Nexus flags tags whose description seems inconsistent with usage. These are typos, stale copy-pastes, or refactors-in-progress. Boss will want a list.
3. **`migration_readiness`** if the customer is talking about a PLC-5 → CLX or SLC → CompactLogix migration. The output is the scope-of-work skeleton.

### Step 5 — Write the handoff

Compose a one-page summary with these sections (use the **`/plc-fds`** skill if Boss wants the full FDS treatment instead):

- **Site**: name, location, customer, current owner of the controls.
- **PLC inventory**: controller family, firmware, slot fill, network address.
- **I/O totals**: counts per type (DI/DO/AI/AO), per area; flagged unverified count.
- **Software contents**: program/routine count, AOI catalog (use **`aoi_library_search`**), UDT count.
- **Safety architecture**: GuardLogix? safety I/O? safety signature locked?
- **Documented gaps**: tags Nexus marks `unverified` + rungs with no descriptions.
- **Recommended next step**: typically "field walk + commissioning checklist" or "FDS gap analysis" depending on customer ask.

Save the output to `/mnt/i/HISMemory/projects/handoffs/{job-number}-handoff.md` and store a memory entry referencing it.

---

## Boundaries

- This skill **inventories**; it does not modify the PLC. Modifications go through `/plc-fds` → customer sign-off → online edit with the proper test discipline.
- If Nexus reports anything in the safety logic, **do not propose changes from this skill**. Safety changes require a separate SISTEMA / 61511 workflow.
- If `tag_suspect_descriptions` exceeds 200 entries, the PLC has bigger problems than a walk-through — flag that to Boss as "this needs a documentation engagement before any modification work."
