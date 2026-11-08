# Contributing to droidtape

Thanks for your interest in droidtape! This project records, replays, and regression-tests AI agents that operate Android devices. Contributions of all kinds are welcome: code, documentation, bug reports, ideas, and feedback from using it with real agents.

> **Status: pre-alpha.** The project is being designed and built in the open. Formats and interfaces will change, and some areas are not ready for code contributions yet (see [Where to start](#where-to-start)).

By participating, you agree to follow our [Code of Conduct](CODE_OF_CONDUCT.md).

## Table of contents

- [Where to start](#where-to-start)
- [How the project is organized](#how-the-project-is-organized)
- [Reporting bugs](#reporting-bugs)
- [Suggesting features](#suggesting-features)
- [Making a code or docs change](#making-a-code-or-docs-change)
- [Commit sign-off (DCO)](#commit-sign-off-dco)
- [AI-assisted contributions](#ai-assisted-contributions)
- [Design decisions](#design-decisions)
- [Testing](#testing)
- [Security issues](#security-issues)
- [License of contributions](#license-of-contributions)

## Where to start

1. Read the [README](README.md) for the vision.
2. Read the [Software Requirements Specification](docs/SRS.md). It defines **what** the project must do. Every requirement has a stable ID such as `FR-REC-003`.
3. Check [open questions](docs/open-questions.md) and [design decisions](docs/decisions/) to see what is already settled or still undecided.
4. Look for issues labeled `good first issue` or `help wanted`.

If you would like to work on something sizeable, **open an issue first** so we can agree on the approach before you invest time.

## How the project is organized

| Where | What |
|---|---|
| `docs/SRS.md` | Requirements: the "what". Changes here go through pull requests like code. |
| `docs/decisions/` | Architecture Decision Records (ADRs): the "why" behind design choices. |
| `docs/open-questions.md` | Unresolved questions linked to requirement IDs. |

Source code layout and development setup instructions will be added here once the implementation language and structure are decided in an ADR.

## Reporting bugs

Please open an issue and include:

- What you expected to happen and what actually happened.
- Steps to reproduce, as minimal as possible.
- Your environment: operating system, device or emulator model, Android version, and droidtape version.
- Relevant logs or error output.

**Important: recordings may contain personal data.** A recorded run includes screenshots and UI contents from a real device, which can show messages, names, passwords, or other private information. Before attaching a recording to an issue, review it and redact anything sensitive, or share only the specific step needed to illustrate the problem. When in doubt, describe the problem in text instead.

## Suggesting features

Open an issue describing the problem you want to solve, not just the solution. If the feature touches the SRS, point to the requirement IDs involved, or say that it is new. New requirements are added to the SRS through a pull request.

## Making a code or docs change

1. **Fork** the repository and create a branch from `main`. Use a short, descriptive name, such as `fix-run-listing-filter` or `docs-quickstart`.
2. **Keep changes focused.** One logical change per pull request is much easier to review than a large mixed one.
3. **Reference requirement IDs** in your pull request description when the change implements or affects a requirement (for example, "Implements FR-DEV-001").
4. **Add or update tests** for any behavior you change (see [Testing](#testing)).
5. **Update documentation** if user-visible behavior changes.
6. **Sign off your commits** (see below).
7. Open a pull request and describe **what** changed and **why**. Link related issues.

Reviewers aim to respond within about a week. This is a small project, so please be patient and feel free to leave a polite nudge.

### Commit messages

Write clear, imperative-mood messages: "Add device discovery", not "Added" or "Adds". Start with a short summary line (about 72 characters or fewer). Add a body when the reason for the change is not obvious.

## Commit sign-off (DCO)

This project uses the [Developer Certificate of Origin (DCO)](https://developercertificate.org/). By signing off on a commit, you certify that you wrote the change or have the right to submit it under the project's license.

Sign off by adding a `Signed-off-by` line to each commit message. Git does this for you:

```
git commit -s -m "Add device discovery"
```

This appends a line like:

```
Signed-off-by: Your Name <your.email@example.com>
```

The name and email must match your Git configuration (`git config user.name` and `git config user.email`).

**Forgot to sign off?**

- Last commit only: `git commit --amend -s --no-edit`, then `git push --force-with-lease`.
- Several commits: `git rebase --signoff main`, then `git push --force-with-lease`.

## AI-assisted contributions

AI coding tools are welcome here, and parts of this project are built with them. The rules are simple:

- **You are responsible for what you submit.** Read, understand, and test every line. "The AI wrote it" is not a reason a change can't be questioned.
- **Do not submit code you can't explain** or can't verify works.
- **Disclose significant AI assistance** in the pull request description (for example, "Drafted with an AI assistant, reviewed and tested by me"). You do not need to disclose trivial autocomplete.
- **Make sure the output is compatible with this project's license**, and that it does not reproduce code you have no right to contribute.
- Low-effort, unreviewed, bulk-generated pull requests will be closed.

## Design decisions

Significant design choices, such as the implementation language, the recording format, or how devices are accessed, are recorded as **Architecture Decision Records** in [`docs/decisions/`](docs/decisions/). If your change makes or alters such a decision, include a new or updated ADR in your pull request.

Requirements are in the SRS and design is in ADRs. Please keep the two separate: the SRS says what, the ADRs say how and why.

## Testing

The automated test suite is intended to run **without a physical device or emulator**, using a simulated device, so that anyone (and CI) can run it. Do not write tests that require a phone to be plugged in unless they are clearly marked as optional hardware tests and skipped by default.

Test commands will be documented here once the implementation language is decided.

## Security issues

**Do not report security vulnerabilities in public issues.** Follow the process in [SECURITY.md](SECURITY.md).

## License of contributions

droidtape is licensed under the [Apache License 2.0](LICENSE). By submitting a contribution, you agree that it is licensed under the same terms, as described in section 5 of that license, and you certify the DCO above.
