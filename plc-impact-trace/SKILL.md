---
name: plc-impact-trace
description: Trace the complete impact chain of a PLC signal (tag, I/O point, or address) — where it's written, where it's read, every interlock it gates, every alarm it feeds, every HMI element that displays it. Use when Boss asks "what does this signal do?", "if I change this tag, what breaks?", or "trace this fault back to its root cause". Wraps the right Nexus tools in a single guided pass so nothing gets missed.
version: 1.0.0
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [nexus, plc, troubleshooting, signal-trace, impact-analysis, his]
---

# /plc-impact-trace — Full signal impact chain

**Invocation:** `/plc-impact-trace <tag-or-address>` (e.g. `/plc-impact-trace Mtr01_RunCmd` or `/plc-impact-trace Local:1:I.Data.5`)

---

## Purpose

Boss is troubleshooting or considering a change and needs to know **everything a signal touches** before he commits. This skill chains the relevant Nexus tools so a five-minute investigation surfaces what a thirty-minute manual one would, and so nothing important gets skipped under time pressure.

The deliverable is a short, structured report on stdout — copy-pasteable into a ticket, a customer email, or the kanban board.

---

## Sequence

Take the tag or address Boss gave you. If it's a raw I/O address (`Local:1:I.Data.5`), call **`promote_raw_addresses_to_tags`** first to resolve it to the buffered tag name (HIS convention maps I/O once at the top of each periodic task). Continue with the tag.

### Pass 1 — Where is it used?

1. **`tag_where_used`** — every routine and rung that references the tag. This is the map of the rest of the trace.
2. **`address_xref`** + **`address_xref_summary`** — same data, faster index, also catches references via tag aliases and InOut parameters.
3. **`classify_signal_roles`** — labels each reference as `read`, `write`, `interlock`, `alarm`, `output`, etc. This is what separates "the tag exists in 14 rungs" from "the tag is **written** in 2, **gated by** 6, **read for display** in 6".

### Pass 2 — Trace the writes (root cause direction)

For each rung that **writes** the tag:

1. **`rung_forensic`** with that rung number — pull the ladder logic in readable form.
2. **`address_trace_chain`** — recursively follow every input on the rung back to its own writer, depth 3-4. This catches the typical "fault → permissive → command" cause chain.

Note the *path*: which boolean wins? Which is failing? Boss usually wants to see the chain laid out as `tag ← rung 41 ← Mtr_Fault ← rung 38 ← E_Stop_OK`.

### Pass 3 — Trace the reads (impact direction)

For each rung or HMI binding that **reads** the tag:

1. **`find_interlocks`** with the tag — the chain of permissives it gates. If Boss changes this tag, these rungs change behavior.
2. **`hmi_search_by_address`** — every HMI screen, button, indicator, faceplate that displays it. Note the screens by area so Boss knows who sees what changes.
3. **`screen_tag_bindings`** filtered to this tag — confirms the binding type (animated, momentary, lock-state, etc.).

### Pass 4 — Is it documented?

Quick sanity check before reporting:

1. **`tag_context`** — Nexus' summary of the tag (description, type, scope, value range if recorded).
2. **`tag_suspect_descriptions`** — does Nexus flag this tag as having a suspect description? If yes, the description is probably wrong; warn Boss.
3. **`reconcile_descriptions`** if the description is multi-source — pick the engineer-verified one if available.

---

## Output format

```
TAG: <tag-name>
TYPE: <BOOL/INT/REAL/etc>  SCOPE: <controller|program:Foo>  DESCRIPTION: <Nexus' best>

WRITERS (root cause direction):
  rung 41 of Area_100/Outputs — written when E_Stop_OK AND Mtr01_Permissive
  rung 7  of System/Watchdog  — cleared on comm fault
    └ E_Stop_OK ← rung 12 of Safety/EStop — written by SE01..SE04 OK chain

READS (impact direction):
  rung 23 of Area_100/Sequencing  — gates Mtr01_StartSeq
  rung 81 of Alarms              — feeds Alm_Mtr01_FailToStart
  HMI: AreaA_Overview            — animated indicator on motor faceplate (3 screens)

INTERLOCKS GATED: 4   ALARMS DRIVEN: 1   HMI BINDINGS: 5

NEXUS CONFIDENCE: high (engineer-verified description; tag last reconciled 2026-04-12)
SUSPECT DESCRIPTION FLAG: no
```

If any of the writers are in safety routines, prefix the line with **`⚠ SAFETY`** and remind Boss that changes here require SISTEMA + signature management.

---

## When *not* to use this skill

- Pure documentation work (e.g. "give me the description of this tag") → use **`tag_context`** directly, no full chain needed.
- Cross-PLC trace (signal produced/consumed between two controllers) → use **`tag_full_chain`** or **`tag_find_plant_wide`** instead, this skill focuses on one PLC.
- HMI-only question ("what's bound to this PV?") → use **`hmi_search_by_address`** + **`screen_tag_bindings`** directly.
