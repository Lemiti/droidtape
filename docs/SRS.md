# Software Requirements Specification (SRS)

## droidtape — Record, Replay, and Regression-Testing for Mobile-Use AI Agents

| Field | Value |
|---|---|
| Document ID | SRS-001 |
| Version | 0.1 (Draft) |
| Date | 2026-11-06 |
| Status | Draft for implementation planning |
| Product name | `droidtape` (**working title**, subject to change) |
| License intent | Open source (Apache-2.0 proposed; see Open Issues) |
| Structure basis | ISO/IEC/IEEE 29148 (adapted) |

---

## 0. Reading Guide

### 0.1 Scope of this document

This document defines **WHAT** the system must do and the qualities it must have. It deliberately does **not** define **HOW** it is built. Architecture, technology choices, data-storage mechanisms, internal module structure, and algorithms belong to the Design phase and MUST NOT be inferred from this document.

### 0.2 Notes for automated implementers and design agents

1. Treat every requirement ID (e.g., `FR-REC-003`) as a stable identifier. Reference IDs in designs, commits, tests, and pull requests.
2. Requirement keywords follow RFC 2119: **shall/must** (mandatory), **should** (recommended), **may** (optional).
3. Each requirement carries a priority:
   - **[M] Must** — required for release v1.0.
   - **[S] Should** — targeted for v1.x; v1.0 if feasible without delaying Must items.
   - **[C] Could** — desirable; future release.
4. Where this document is silent or ambiguous, **do not guess**. Record the question in `docs/open-questions.md` referencing the relevant requirement ID.
5. Where an external technology is named (e.g., the Model Context Protocol), it is named only because it is an **external interface the product must interoperate with**, not as an implementation choice.
6. Section 7 (Open Issues) lists known unresolved decisions. Do not treat them as decided.

---

## 1. Introduction

### 1.1 Purpose

This SRS specifies the requirements for **droidtape**, an open-source developer tool that records, replays, and regression-tests AI agents that operate Android devices ("mobile-use agents").

### 1.2 Problem Statement

Developers building agents that operate real mobile apps face three recurring problems:

1. **Opacity:** When an agent fails on a device, there is no structured record of what it saw, what it decided, and what happened.
2. **Non-reproducibility:** Agent runs are non-deterministic and mobile apps are stateful, so failures are difficult to reproduce and investigate.
3. **No safety net for change:** Changing a prompt, model, or agent logic can silently break previously working tasks, and there is no standard way to detect this.

### 1.3 Product Vision

A developer can attach any compatible agent to a real Android device, capture every step of its work, step back through that work, re-run it against recorded or live conditions, branch from any step to try a change, and turn good or bad runs into repeatable tests that run in CI.

### 1.4 Scope

**In scope for v1:**
- Giving agents controlled access to an Android device.
- Recording agent runs on that device.
- Inspecting and replaying recorded runs (against recorded observations and against a live device).
- Forking a recorded run from any step.
- Defining and executing repeatable regression tests with pass/fail reporting suitable for CI.
- A command-line interface and a local web-based viewer.

**Out of scope for v1:** see Section 6.

### 1.5 Definitions, Acronyms, and Abbreviations

| Term | Definition |
|---|---|
| **Agent** | An AI-driven program that decides and issues actions to accomplish a task. |
| **Mobile-use agent** | An agent that accomplishes tasks by observing and operating a mobile device's user interface. |
| **Target device** | The Android device (physical or emulated) the agent operates. |
| **Run** | One complete attempt by an agent to perform one task, from start to end. |
| **Step** | One observe-decide-act cycle within a run. |
| **Observation** | What the agent perceives at a step: screenshot, structured UI hierarchy, and basic device state. |
| **Action** | An operation the agent performs on the device (e.g., tap, swipe, type). |
| **Recording** | The persisted, complete record of a run. |
| **Observation Replay** | Re-running an agent against the *recorded* observations of a prior run, without a live device. |
| **Action Replay** | Re-executing a run's *recorded actions* on a live device and comparing resulting screens to the recording. |
| **Fork** | A new run that reuses a prior run up to a chosen step and then diverges with a change. |
| **Divergence** | The first point where behavior (agent action or screen state) differs from the recording. |
| **Task Spec** | A declarative definition of a task, its success criteria, and limits, used for regression testing. |
| **Assertion** | A checkable condition on observed state or on the run's trajectory. |
| **Attempt** | One run executed as part of a test. |
| **Pass rate** | Passed attempts divided by total attempts for a Task Spec. |
| **Flaky** | A Task Spec is flaky when repeated attempts under identical specification and configuration produce mixed outcomes (at least one pass and at least one fail). |
| **Harness** | The program or script that runs an agent and connects it to the system. |
| **MCP** | Model Context Protocol, an open protocol through which agents access external tools. |
| **CI** | Continuous Integration. |

### 1.6 References

- ISO/IEC/IEEE 29148:2018, Systems and software engineering: Life cycle processes: Requirements engineering.
- RFC 2119, Key words for use in RFCs to Indicate Requirement Levels.
- Model Context Protocol specification (current published version at time of design).
- JUnit XML report format (de facto standard for CI test reporting).

---

## 2. Overall Description

### 2.1 Product Perspective

droidtape is a standalone, local-first developer tool. It sits between an agent and an Android device. It does not provide, host, or train agents or models; it observes, records, replays, and evaluates agents that others build. It is intended to work with many agents without requiring changes to their code beyond configuration.

### 2.2 Product Functions (Summary)

1. **Device access:** discover devices; provide observation and action capabilities to agents.
2. **Agent integration:** let any compatible agent connect with minimal change.
3. **Recording:** capture complete, structured recordings of runs.
4. **Run management:** list, inspect, export, import, and delete recordings.
5. **Replay:** step through recordings; Observation Replay; Action Replay; run comparison.
6. **Fork:** branch a run from any step with a change.
7. **Regression testing:** define tasks, run them repeatedly, report results, gate CI.
8. **Visualization:** a local web viewer for recordings, comparisons, and reports.
9. **Safety and privacy:** action guardrails and redaction of sensitive content.

### 2.3 User Classes and Characteristics

| User class | Description | Primary needs |
|---|---|---|
| **Agent developer** (primary) | Builds or tunes agents that operate Android apps. | Understand failures, reproduce them, try changes, detect regressions. |
| **QA / test engineer** | Verifies agent-driven workflows on apps. | Repeatable tests, pass rates, flakiness data, CI integration. |
| **Researcher** | Evaluates agent models or strategies on mobile tasks. | Comparable, shareable, reproducible runs and metrics. |
| **Open-source contributor** | Extends or maintains the project. | Clear docs, stable formats, testability without hardware. |

All users are assumed to be comfortable with a command line.

### 2.4 Operating Environment

- **Host:** Linux (primary), macOS (secondary), Windows (tertiary; see NFR-POR-001).
- **Reference environment** (for performance requirements): x86-64 laptop, 4 CPU cores, 8 GB RAM, no GPU, running Linux, with one physical Android device attached.
- **Target devices:** physical Android devices and Android emulators with developer debugging enabled.
- **Agents:** any agent or harness able to use the system's agent-facing interface (see FR-AGI-001).

### 2.5 Design and Implementation Constraints

- **C-1:** The product shall be open source under an OSI-approved license.
- **C-2:** The product shall be local-first: all core functionality shall work with no cloud service operated by the project.
- **C-3:** The product shall not require root access on the target device or modification of the app under test.
- **C-4:** The product shall run acceptably on the reference environment without requiring an emulator, containers, or external database servers.
- **C-5:** Data formats shall be documented and versioned independently of the implementation.
- **C-6:** The design shall not preclude future support for additional target platforms (e.g., iOS, web browsers) without breaking existing recordings.

### 2.6 Assumptions and Dependencies

- **A-1:** Users can enable developer debugging on their device.
- **A-2:** Agents are non-deterministic; identical inputs may produce different actions.
- **A-3:** Apps under test are stateful and may change over time (content, layout, timing).
- **A-4:** Some screens (e.g., games, custom-rendered or embedded web content) expose little or no structured UI information.
- **A-5:** Model-call data (prompts, responses, token counts, cost) is known only to the agent/harness, not to the device.

---

## 3. External Interface Requirements

### 3.1 Command-Line Interface

| ID | Pri | Requirement |
|---|---|---|
| EI-CLI-001 | M | The system shall provide a command-line interface exposing all v1 functionality. |
| EI-CLI-002 | M | Commands shall support non-interactive operation suitable for scripts and CI. |
| EI-CLI-003 | M | Listing and inspection commands shall offer machine-readable (JSON) output. |
| EI-CLI-004 | M | Every command shall provide built-in help describing purpose, options, and examples. |

### 3.2 Agent-Facing Interface

| ID | Pri | Requirement |
|---|---|---|
| EI-AGT-001 | M | The system shall expose device observation and action capabilities to agents through the **Model Context Protocol (MCP)**. |
| EI-AGT-002 | M | The agent-facing interface shall be identical whether the backing source is a live device or an Observation Replay (see FR-RPL-002). |
| EI-AGT-003 | S | The system shall document the agent-facing capabilities (names, inputs, outputs, error behavior) as a stable, versioned reference. |

### 3.3 Web Viewer Interface

| ID | Pri | Requirement |
|---|---|---|
| EI-UI-001 | M | The system shall provide a browser-based viewer served locally. |
| EI-UI-002 | M | The viewer shall function with no internet access. |

### 3.4 Device Interface

| ID | Pri | Requirement |
|---|---|---|
| EI-DEV-001 | M | The system shall communicate with Android target devices (physical and emulated) using only capabilities available when standard developer debugging is enabled. |

### 3.5 File and Data Interfaces

| ID | Pri | Requirement |
|---|---|---|
| EI-FILE-001 | M | Recordings shall be stored in an open, documented, versioned format. |
| EI-FILE-002 | M | Task Specs shall be plain-text, human-readable, version-control-friendly files with a documented, versioned schema. |
| EI-FILE-003 | M | Test results shall be emitted in JSON and in JUnit-compatible XML. |
| EI-FILE-004 | C | The system may export recordings to an OpenTelemetry-compatible representation. |

---

## 4. Functional Requirements

### 4.1 Device Access (DEV)

| ID | Pri | Requirement |
|---|---|---|
| FR-DEV-001 | M | The system shall discover and list connected target devices with identifier, model, Android version, and connection state. |
| FR-DEV-002 | M | The user shall be able to explicitly select the target device. If more than one device is available and none is selected, the system shall refuse to proceed and prompt for a selection rather than guess. |
| FR-DEV-003 | M | The system shall provide agents with these observations: a screenshot, a structured UI hierarchy (elements with text, description, type, bounds, and state), the foreground application identity, and screen size/orientation. |
| FR-DEV-004 | M | The system shall provide agents with these actions: tap, long-press, swipe/scroll, text input, key events (at minimum back, home, enter), launch an application, and wait. |
| FR-DEV-005 | M | Every action shall return an explicit success or failure result with error detail on failure. |
| FR-DEV-006 | M | The system shall remain usable when the UI hierarchy is unavailable or empty for a screen, providing screenshot-only observation, and shall mark the observation as degraded in the recording. |
| FR-DEV-007 | S | Actions shall be targetable by screen coordinates and by reference to an element from the current UI hierarchy. |
| FR-DEV-008 | S | The user shall be able to configure a **safety policy** that allows or blocks specific actions and applications (e.g., block uninstalling apps, factory reset, or specified apps). Blocked requests shall be refused with a clear result and recorded. |
| FR-DEV-009 | C | The system may support operating multiple devices concurrently. |

### 4.2 Agent Integration (AGI)

| ID | Pri | Requirement |
|---|---|---|
| FR-AGI-001 | M | Any agent or harness capable of using the agent-facing interface (EI-AGT-001) shall be able to operate a device and have its run recorded **without modification to its code**, beyond configuring its connection to the system. |
| FR-AGI-002 | M | A run shall be startable and endable explicitly, with a task description and optional metadata (e.g., tags, agent name, model identifier). |
| FR-AGI-003 | M | If an agent uses the system without explicitly starting a run, the system shall create a run implicitly and mark it as such. |
| FR-AGI-004 | S | The system shall provide a means for an agent or harness to attach optional per-step information to a recording: model identifier, prompt and response content, token counts, cost, and free-text rationale. |
| FR-AGI-005 | S | The run outcome (success, failure, abandoned) shall be reportable by the agent or harness when known. |

### 4.3 Recording (REC)

| ID | Pri | Requirement |
|---|---|---|
| FR-REC-001 | M | For every step, the system shall record: step index, timestamp, observation (screenshot and UI hierarchy), requested action with parameters, action result, and step duration. |
| FR-REC-002 | M | For every run, the system shall record: unique run identifier, task description, target device details, start and end times, final status (completed, aborted, errored, timed out), tool version, and recording-format version. |
| FR-REC-003 | M | Recording shall be incremental and durable: if the agent, the system, or the host terminates unexpectedly, all steps completed before termination shall be retained and the run marked as incomplete. |
| FR-REC-004 | M | Recordings shall be stored locally by default, be self-contained, and be portable between machines. |
| FR-REC-005 | M | The recording format shall be versioned. Newer releases within the same major version shall read recordings produced by older releases. |
| FR-REC-006 | M | The user shall be able to configure **redaction** that masks sensitive content (by screen region, UI element, or text pattern) before it is persisted, and shall be able to disable screenshot persistence entirely. |
| FR-REC-007 | M | The system shall record whether each observation was degraded (see FR-DEV-006) and any actions refused by the safety policy (see FR-DEV-008). |
| FR-REC-008 | S | The system shall compute and store per-run summary metrics: step count, total duration, and (where supplied under FR-AGI-004) token usage and cost. |
| FR-REC-009 | S | Users shall be able to label runs with free-form tags. |

### 4.4 Run Management (RUN)

| ID | Pri | Requirement |
|---|---|---|
| FR-RUN-001 | M | The user shall be able to list recorded runs and filter by task, status, date range, tag, agent, and model. |
| FR-RUN-002 | M | The user shall be able to view a run's summary and step details from the command line. |
| FR-RUN-003 | M | The user shall be able to delete one or more runs. |
| FR-RUN-004 | S | The user shall be able to export a run as a single shareable file and import such a file. Importing shall never execute any content from the file. |
| FR-RUN-005 | S | The system shall report storage consumed by recordings and support a configurable retention policy. |
| FR-RUN-006 | S | The system shall detect and report corrupted or incomplete recordings. |

### 4.5 Inspection and Replay (RPL)

| ID | Pri | Requirement |
|---|---|---|
| FR-RPL-001 | M | The user shall be able to step forward, step backward, and jump to any step of a recorded run, viewing the observation, action, result, and timing at that step. |
| FR-RPL-002 | M | **Observation Replay:** the system shall serve a run's recorded observations to an agent, in recorded order, in place of a live device and through the same agent-facing interface, so agent behavior can be re-evaluated without a device. |
| FR-RPL-003 | M | During Observation Replay, the system shall compare each action the agent issues with the recorded action at the same step, and shall detect and report the first divergence, including the step and the nature of the difference (different action type, target, or parameters). |
| FR-RPL-004 | M | On divergence during Observation Replay, the system shall by default stop and report. |
| FR-RPL-005 | M | **Action Replay:** the system shall re-execute a run's recorded actions on a live target device and, after each step, compare the resulting screen state with the recorded state, reporting per-step differences. |
| FR-RPL-006 | M | Action Replay comparison shall consider both visual appearance and UI structure, and shall allow configurable tolerances and ignore-regions (e.g., clock, battery, notifications) to avoid false differences. |
| FR-RPL-007 | M | Replays shall never alter the original recording. Each replay shall produce a new, linked run record identifying its source run and replay type. |
| FR-RPL-008 | M | Given the same recording and the same replay configuration, comparison results shall be deterministic. |
| FR-RPL-009 | S | The user shall be able to compare two runs of the same task, highlighting the first divergence and differences in outcome, step count, duration, and cost. |
| FR-RPL-010 | C | Observation Replay may support continuing after divergence under a user-selected policy. |

### 4.6 Fork (FRK)

| ID | Pri | Requirement |
|---|---|---|
| FR-FRK-001 | M | The user shall be able to create a new run derived from an existing run at a user-chosen step N, preserving steps before N. |
| FR-FRK-002 | M | At the fork point the user shall be able to change at least one of: the task instruction, the agent/model configuration supplied to the harness, or the action taken at step N. |
| FR-FRK-003 | M | Two fork modes shall be supported: **Observation fork** (earlier steps served from the recording, no device needed) and **Device fork** (device state re-established by replaying earlier recorded actions, then continuing live). |
| FR-FRK-004 | M | A forked run shall record its parent run and fork step. |
| FR-FRK-005 | S | The user shall be able to compare a forked run with its parent (see FR-RPL-009). |

### 4.7 Regression Testing (TST)

| ID | Pri | Requirement |
|---|---|---|
| FR-TST-001 | M | The system shall support Task Specs defined in plain-text files (EI-FILE-002). |
| FR-TST-002 | M | A Task Spec shall be able to define: identifier and name, task instruction, target application and preconditions, success criteria, limits (maximum steps, maximum duration, optional maximum cost), number of attempts, and required pass rate. |
| FR-TST-003 | M | Success criteria shall be expressible as Assertions over observed state, including: text or element present or absent, foreground application identity, and element property values; combinable with all-of and any-of logic; evaluable at the final step or at any step. |
| FR-TST-004 | M | Assertions shall also be expressible over a run's trajectory, including at minimum: maximum step count and forbidden actions or screens. |
| FR-TST-005 | M | The user shall be able to generate a draft Task Spec from an existing recorded run, then edit it. |
| FR-TST-006 | M | The system shall execute a Task Spec against a user-specified agent command or configuration for the specified number of attempts, recording each attempt as a run. |
| FR-TST-007 | M | The system shall report, per Task Spec: outcome per attempt, pass rate, flaky indicator, failure reason and failing step for each failed attempt, and average steps, duration, and cost. |
| FR-TST-008 | M | The system shall return process exit statuses that distinguish: all requirements met; pass-rate requirement not met; and infrastructure or usage error. |
| FR-TST-009 | M | The system shall produce results in JSON, JUnit-compatible XML, and a human-readable summary. |
| FR-TST-010 | S | The user shall be able to compare a test result set against a baseline and flag regressions where pass rate falls by more than a configurable threshold. |
| FR-TST-011 | S | The user shall be able to provide setup and teardown commands to run before and after each attempt (e.g., to reset application state). |
| FR-TST-012 | S | The user shall be able to run multiple Task Specs together and select them by name or tag. |
| FR-TST-013 | S | Task Specs shall be runnable in Observation Replay mode, requiring no device, to check agent logic deterministically. |
| FR-TST-014 | C | The system may run attempts in parallel across multiple devices. |

### 4.8 Visualization (UI)

| ID | Pri | Requirement |
|---|---|---|
| FR-UI-001 | M | The viewer shall list recorded runs with key metadata and allow opening any run. |
| FR-UI-002 | M | The viewer shall present a run as a timeline of steps and, for the selected step, show the screenshot with the action visually overlaid (e.g., tap location, swipe path), the UI hierarchy, the action and result, timing, and any attached model/rationale data. |
| FR-UI-003 | M | The viewer shall be bound to the local machine by default and shall not require any external network resource. |
| FR-UI-004 | S | The viewer shall provide a run-comparison view (FR-RPL-009) and a test-report view (FR-TST-007). |
| FR-UI-005 | S | The viewer shall support keyboard navigation between steps. |
| FR-UI-006 | C | The viewer may provide a live view of a run in progress. |

### 4.9 Configuration and Diagnostics (CFG)

| ID | Pri | Requirement |
|---|---|---|
| FR-CFG-001 | M | The system shall be configurable through a configuration file with command-line overrides and shall provide sensible defaults such that a single attached device and a single agent work with no configuration. |
| FR-CFG-002 | S | The system shall provide a diagnostic command that checks prerequisites (e.g., device connectivity, debugging enabled, permissions) and reports actionable remedies. |
| FR-CFG-003 | M | Error messages shall state what failed and what the user can do about it. |

### 4.10 Extensibility (EXT)

| ID | Pri | Requirement |
|---|---|---|
| FR-EXT-001 | C | The system may provide documented extension points for custom assertion types and redaction rules. |
| FR-EXT-002 | C | The system may allow additional agent-facing integration styles beyond MCP in future releases without breaking recordings. |

---

## 5. Non-Functional Requirements

### 5.1 Performance and Resource Use

All measurements are on the reference environment (Section 2.4) unless stated.

| ID | Pri | Requirement |
|---|---|---|
| NFR-PERF-001 | M | The system shall add no more than 250 ms median latency to any observation or action, beyond the time taken by the device itself. |
| NFR-PERF-002 | M | Listing 1,000 recorded runs shall complete in under 2 seconds. |
| NFR-PERF-003 | M | Opening a 500-step run in the viewer shall display the first step in under 3 seconds. |
| NFR-PERF-004 | M | Commands that do not need a device shall start and respond within 1 second for simple operations. |
| NFR-PERF-005 | M | The system's own memory use while recording and serving the viewer shall not exceed 500 MB. |
| NFR-PERF-006 | S | Persisted data per step shall average no more than 500 KB at default settings, with a user-adjustable quality/size trade-off. |

### 5.2 Reliability and Data Integrity

| ID | Pri | Requirement |
|---|---|---|
| NFR-REL-001 | M | Recordings shall not be silently lost or corrupted by agent crashes, system crashes, or abrupt termination (see FR-REC-003). |
| NFR-REL-002 | M | If the device disconnects mid-run, the system shall mark the run as errored, retain all prior steps, and report the cause clearly. |
| NFR-REL-003 | M | The system shall never alter or delete a recording except at the explicit direction of the user. |
| NFR-REL-004 | S | The system shall verify recording integrity on read and report problems. |

### 5.3 Usability

| ID | Pri | Requirement |
|---|---|---|
| NFR-USA-001 | M | A developer with an Android device and an MCP-capable agent shall be able to install the system and record a first run within **15 minutes** following only the quick-start documentation. |
| NFR-USA-002 | M | Documentation shall include: quick start, core concepts, reference for every command, recording-format specification, Task Spec schema, troubleshooting guide, and at least three complete example tasks. |
| NFR-USA-003 | S | Terminology in the CLI, viewer, and documentation shall be consistent with Section 1.5. |

### 5.4 Portability and Compatibility

| ID | Pri | Requirement |
|---|---|---|
| NFR-POR-001 | M | Linux shall be fully supported. macOS should be supported (S). Windows may be supported (C). |
| NFR-POR-002 | M | The system shall support physical devices and emulators running Android 10 (API level 29) or later (**proposed**; see Open Issues). |
| NFR-POR-003 | M | The system shall be installable with a single command from a public package registry and shall operate offline after installation. |
| NFR-POR-004 | M | The system shall be verified against at least two distinct MCP-capable agent clients before each major release. |

### 5.5 Security and Privacy

| ID | Pri | Requirement |
|---|---|---|
| NFR-SEC-001 | M | The system shall send no telemetry and make no network requests of its own by default. Any network-using feature shall be opt-in and documented. |
| NFR-SEC-002 | M | The local viewer shall accept connections only from the local machine by default. |
| NFR-SEC-003 | M | Documentation shall warn that recordings may contain sensitive personal data and describe redaction and safe sharing practices. |
| NFR-SEC-004 | M | The system shall not execute content contained in imported recordings or Task Specs, except user-defined setup/teardown commands that the user has explicitly configured. |
| NFR-SEC-005 | S | The project shall scan dependencies for known vulnerabilities and incompatible licenses in CI. |
| NFR-SEC-006 | S | Recordings shall be excluded from version control by default in project templates and guidance. |

### 5.6 Maintainability and Open-Source Health

| ID | Pri | Requirement |
|---|---|---|
| NFR-OSS-001 | M | The project shall be published under an OSI-approved open-source license. |
| NFR-OSS-002 | M | The repository shall include: README, contribution guide, code of conduct, security-reporting policy, issue and pull-request templates, and a changelog. |
| NFR-OSS-003 | M | The project shall follow semantic versioning, with the recording format and Task Spec schema versioned and changes to them documented. |
| NFR-OSS-004 | M | The automated test suite and CI shall run **without** a physical device or emulator, using a simulated device. |
| NFR-OSS-005 | S | Core logic shall have automated test coverage of at least 80%. |
| NFR-OSS-006 | S | CI shall run on every pull request and block merging on failure. |

---

## 6. Out of Scope (v1)

The following are explicitly **not** required for v1.0:

1. Support for iOS, desktop, or browser-based agents.
2. Hosted or cloud services, accounts, authentication, or multi-user collaboration.
3. Cloud device farms or remote device management.
4. Automated judging of task success by a language model (success is defined by Assertions).
5. Building, hosting, training, or fine-tuning agents or models.
6. Automatic bypass of device protections, CAPTCHAs, or app security features.
7. Performance benchmarking of the apps under test.
8. Parallel multi-device execution (C-priority only).

---

## 7. Open Issues

| # | Issue | Related requirements |
|---|---|---|
| OI-1 | Final project name and conflict check against existing projects and registries. | All |
| OI-2 | Final license selection (Apache-2.0 proposed). | NFR-OSS-001 |
| OI-3 | Minimum supported Android version (API 29 proposed). | NFR-POR-002 |
| OI-4 | Which two MCP-capable agent clients are the release verification targets. | NFR-POR-004 |
| OI-5 | Default visual and structural comparison tolerances and default ignore-regions for Action Replay. | FR-RPL-006 |
| OI-6 | Whether a numeric flakiness score is needed beyond the mixed-outcome indicator. | FR-TST-007 |
| OI-7 | Exact assertion vocabulary and expression syntax for Task Specs. | FR-TST-003, FR-TST-004 |
| OI-8 | Validation of performance targets against real measurements; targets may be revised. | NFR-PERF-* |
| OI-9 | Default redaction rules shipped with the product (e.g., password fields). | FR-REC-006 |
| OI-10 | Handling of model-call data privacy (prompts/responses may contain sensitive content). | FR-AGI-004, FR-REC-006 |

---

## 8. Release Acceptance Criteria (v1.0)

v1.0 is releasable when **all [M] requirements** are met and the following scenarios pass:

| ID | Scenario | Verifies |
|---|---|---|
| AC-1 | On a clean machine, a new user installs the tool, connects a physical device and an MCP-capable agent, and records a first run within 15 minutes using only the quick start. | NFR-USA-001, NFR-POR-003, FR-AGI-001 |
| AC-2 | Three reference tasks are recorded on a physical device; each recording is complete and viewable step-by-step in CLI and viewer. | FR-REC-001/002, FR-RPL-001, FR-UI-002 |
| AC-3 | Observation Replay of a recorded run with no device attached reproduces the recorded observations to an agent and reports divergence correctly when the agent's action is altered. | FR-RPL-002/003/004 |
| AC-4 | Action Replay on a live device reports no differences for an unchanged app and reports the correct step when the app state is deliberately altered. | FR-RPL-005/006 |
| AC-5 | A run is forked at step N in both modes and the fork records its parent and fork step. | FR-FRK-001 to 004 |
| AC-6 | A Task Spec executed for 10 attempts yields per-attempt outcomes, pass rate, flaky indicator, JSON and JUnit output, and correct exit statuses in CI for pass, threshold failure, and infrastructure error. | FR-TST-006 to 009 |
| AC-7 | Killing the agent and the system mid-run, and unplugging the device mid-run, each leave a readable partial recording marked incomplete or errored. | FR-REC-003, NFR-REL-001/002 |
| AC-8 | With redaction configured for a password field and a text pattern, neither appears in the persisted screenshots or UI hierarchy. | FR-REC-006 |
| AC-9 | The full automated test suite passes in CI with no device or emulator attached. | NFR-OSS-004 |
| AC-10 | The system makes no outbound network connections in a default recording session. | NFR-SEC-001 |

---

## 9. Requirements Summary by Priority

| Category | Must | Should | Could |
|---|---|---|---|
| Device access (DEV) | 6 | 2 | 1 |
| Agent integration (AGI) | 3 | 2 | 0 |
| Recording (REC) | 7 | 2 | 0 |
| Run management (RUN) | 3 | 3 | 0 |
| Replay (RPL) | 8 | 1 | 1 |
| Fork (FRK) | 4 | 1 | 0 |
| Testing (TST) | 9 | 4 | 1 |
| Visualization (UI) | 3 | 2 | 1 |
| Configuration (CFG) | 2 | 1 | 0 |
| Extensibility (EXT) | 0 | 0 | 2 |

*Non-functional and interface requirements carry their own priorities in Sections 3 and 5.*

---

## 10. Document History

| Version | Date | Change |
|---|---|---|
| 0.1 | 2026-11-06 | Initial draft. |
