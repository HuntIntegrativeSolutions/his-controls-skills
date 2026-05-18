# Font Stack Discrepancy Note

The authoritative source is the **SKILL.md quality gates**, not HIS-DIAGRAM-STANDARD.md.

- **HIS-DIAGRAM-STANDARD.md line 34** says: `fontname="Segoe UI"` (bare)
- **SKILL.md Step 4 and Quality Gates** say: `fontname="Inter, Segoe UI, Arial, sans-serif"` (web-safe stack)

**Always use the web-safe stack.** The bare Segoe UI on non-Windows hosts (Linux CI, macOS, Graphviz servers) falls back to Times New Roman, producing embarrassing diagrams.

If HIS-DIAGRAM-STANDARD.md hasn't yet been updated, the SKILL.md overrides it. The skill instructions and quality gates are the enforcement mechanism.
