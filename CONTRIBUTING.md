# Contributing to HIS Controls Skills

Thank you for considering contributing to the HIS Controls industrial automation skills library. These skills generate engineering-grade documentation for PLC, HMI, SCADA, and safety system projects. Contributions that improve correctness, standards compliance, or real-world applicability are welcome.

## Code of Conduct

This project follows the [Contributor Covenant Code of Conduct](CODE_OF_CONDUCT.md). By participating, you are expected to uphold this code. Please report unacceptable behavior to [t0311750@gmail.com](mailto:t0311750@gmail.com).

## What We're Looking For

- **Correctness fixes** — wrong instruction references, incorrect standard citations, or unsafe engineering patterns
- **Missing standards** — skills should cite every governing standard by edition and section number
- **Real-world edge cases** — conditions you've encountered on actual commissioning jobs that the skill doesn't cover
- **Documentation clarity** — sections that are confusing or ambiguous when an agent follows them

## What We're NOT Looking For

- Style/preference changes without engineering justification
- Features outside the scope of industrial automation engineering
- Breaking changes to the skill output format without discussion first

## Setup

These skills are designed for the [Hermes Agent](https://hermes-agent.nousresearch.com) platform. To test a skill locally:

1. Install Hermes Agent following the [official setup guide](https://hermes-agent.nousresearch.com/docs)
2. Copy the skill directory into `~/.hermes/profiles/<your-profile>/skills/`
3. Invoke the skill from any Hermes session: `/skill-name`

No build tools, dependencies, or runtime environment is needed — the skills are markdown instruction sets consumed by AI agents.

## Pull Request Process

1. **Open an issue first** — describe the problem or improvement before writing code. Use the bug report template for fixes, feature request for additions.
2. **Branch naming** — use descriptive names: `fix/fds-pressure-loop-citation`, `add/soo-regulated-overlay`, `docs/clarify-bom-spares`
3. **One logical change per PR** — don't bundle unrelated fixes. This makes review faster and reverting safer.
4. **Cite your sources** — if you're correcting a standard reference, link to the official document or a reputable summary. If you're adding an edge case, describe the real-world scenario.
5. **Update the skill's revision history** — each SKILL.md has a revision block at the bottom. Add your change with date, author, and a one-line description.
6. **Keep output formats stable** — the skills produce deliverables that go to customers. Don't change section numbering, field names, or document structure without discussion.

## Review Criteria

A maintainer will check:

- [ ] The change addresses a real engineering problem or gap
- [ ] Standard citations are correct (edition, section number)
- [ ] The skill still produces a coherent deliverable end-to-end
- [ ] No safety-critical ambiguities introduced (E-stop logic, SIL boundaries, etc.)
- [ ] Revision history updated

## Engineering Standards Referenced

Contributors should be familiar with the standards these skills cite:

| Standard | Topic |
|----------|-------|
| ANSI/ISA-5.06.01 | Functional requirements documentation |
| ANSI/ISA-88.00.01 | Batch control / procedural automation |
| ANSI/ISA-101.01 | HMI design |
| ISA-18.2 / IEC 62682 | Alarm management |
| IEC 61131-3 | PLC programming languages |
| IEC 61511 | Functional safety — process sector |
| ISO 13849 | Safety of machinery |
| NFPA 79 | Electrical standard for industrial machinery |
| UL 508A | Industrial control panels |
| IEC 62443 | Industrial network security |

## Questions?

Open a [GitHub Discussion](https://github.com/HuntIntegrativeSolutions/his-controls-skills/discussions) for general questions. Use issues for specific bugs or feature requests.

---

*Hunt Integrative Solutions LLC — industrial automation systems integration*
