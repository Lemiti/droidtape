# Open Questions

This file tracks unresolved questions about droidtape. It is the place to record uncertainty **instead of guessing**.

- The **SRS** (`docs/SRS.md`) says *what* the product must do.
- **ADRs** (`docs/decisions/`) record *how and why* once a question is decided.
- This file holds everything still undecided, linked to requirement IDs.

## How to use this file

1. Add a new row with the next free `OI-` number. Keep it to one clear question.
2. Link the requirement IDs it affects (for example `FR-RPL-006`).
3. Set the status: `Open`, `Investigating`, `Proposed` (an ADR draft exists), or `Resolved`.
4. When resolved, link the ADR that records the decision. Do **not** delete resolved rows; keep them as history.
5. Automated tools and contributors: if the SRS is silent or ambiguous about something you need, add a question here and stop short of inventing the answer.

## Questions

| ID | Question | Related | Status | Notes / Resolution |
|---|---|---|---|---|
| OI-1 | Final project name, with conflict check across code hosts, package registries, and trademarks. | All | Partially resolved | Name `droidtape` chosen in [ADR-0001](decisions/0001-project-name.md). Trademark search result still to be recorded in the ADR. |
| OI-2 | Final license selection. | NFR-OSS-001 | Resolved | Apache-2.0 (see `LICENSE`). Record in an ADR if not already captured. |
| OI-3 | Minimum supported Android version. | NFR-POR-002 | Open | API 29 (Android 10) proposed. Validate during the spike. |
| OI-4 | Which two MCP-capable agent clients are the release verification targets. | NFR-POR-004 | Open | |
| OI-5 | Default visual and structural comparison tolerances and ignore-regions for Action Replay. | FR-RPL-006 | Open | Needs real recordings to calibrate. |
| OI-6 | Is a numeric flakiness score needed beyond the "mixed outcomes" indicator? | FR-TST-007 | Open | |
| OI-7 | Exact assertion vocabulary and expression syntax for Task Specs. | FR-TST-003, FR-TST-004 | Open | |
| OI-8 | Validate performance targets against real measurements. | NFR-PERF-* | Open | Measure during the spike; revise targets if unrealistic. |
| OI-9 | Default redaction rules shipped with the product (for example, password fields). | FR-REC-006 | Open | |
| OI-10 | Privacy handling of model-call data (prompts and responses may contain sensitive content). | FR-AGI-004, FR-REC-006 | Open | |
| OI-11 | Implementation language(s) and overall stack for the core. | C-4, NFR-OSS-004 | Open | The README currently lists a planned stack. Decide via ADR-0002, weighing contributor reach, iteration speed, and maintenance burden for a small team. Do not treat the README as the decision. |
| OI-12 | Should Observation Replay also replay recorded **model outputs** (so replay needs no model calls), or only recorded observations? | FR-RPL-002, FR-AGI-004 | Open | As specified, the agent still calls its model during Observation Replay, so replay can cost money. README wording should not claim "no model spend" until this is decided. |
| OI-13 | Are SDKs needed for v1, given agents connect through MCP with no code changes? | FR-AGI-001 | Open | Likely unnecessary for v1; revisit after the spike. |
| OI-14 | Package-name placeholders: which registries hold placeholder releases, and what is their maintenance plan (tokens, 2FA, ownership)? | ADR-0001 | Open | Keep README status and ADR consequences consistent with what is actually published. |
| OI-15 | Is DCO sign-off the right contribution requirement, or is the license's contribution clause enough? | NFR-OSS-002 | Open | DCO is currently required in the README; confirm and set up enforcement (for example, a DCO check on pull requests). |
| OI-16 | Method for obtaining structured UI information from the device, and its speed and reliability limits. | FR-DEV-003, FR-DEV-006, NFR-PERF-001 | Open | Resolve with the technical spike, then record in an ADR. |
| OI-17 | Contact addresses for security reports and Code of Conduct enforcement. | NFR-OSS-002 | Resolved | Contact address set to leminegeso@gmail.com in `SECURITY.md` and `CODE_OF_CONDUCT.md`. |

## Planned decision records

| ADR | Topic | Depends on |
|---|---|---|
| ADR-0001 | Project name | Done (trademark result pending) |
| ADR-0002 | Implementation language and stack | OI-11, OI-13, spike results |
| ADR-0003 | Recording format and versioning | FR-REC-004, FR-REC-005, EI-FILE-001 |
| ADR-0004 | Device access and UI-hierarchy capture | OI-16 |
| ADR-0005 | Agent-facing interface structure | EI-AGT-001 to EI-AGT-003 |
| ADR-0006 | Storage layout for recordings | FR-REC-003, NFR-REL-001 |
