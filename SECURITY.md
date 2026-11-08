# Security Policy

droidtape records the activity of agents operating real Android devices. Recordings can contain sensitive information, and the tool can control a device. We take security and privacy issues seriously.

## Supported versions

droidtape is **pre-alpha**. Only the latest code on the `main` branch is supported. Once stable releases exist, this table will list which versions receive security fixes.

| Version | Supported |
|---|---|
| `main` (latest) | Yes |
| Anything else | No |

## Reporting a vulnerability

**Please do not open a public issue for security problems.**

Report privately using either of these:

1. **GitHub private vulnerability reporting** (preferred): go to the repository's **Security** tab and choose **Report a vulnerability**.
2. **Email:** [leminegeso@gmail.com](mailto:leminegeso@gmail.com)

Please include:

- A description of the issue and its impact.
- Steps to reproduce, or a proof of concept.
- The affected version or commit.
- Your environment (operating system, device or emulator, Android version).
- Whether you plan to disclose publicly, and any deadline you have in mind.

**Do not include real personal data** in your report. If a recording is needed to reproduce the issue, use a test device with dummy content.

## What to expect

This is a small project, so these are targets rather than guarantees:

| Step | Target |
|---|---|
| Acknowledge your report | Within 7 days |
| Initial assessment and severity | Within 14 days |
| Fix or mitigation plan | Depends on severity; critical issues are prioritized |
| Public disclosure | Coordinated with you; by default up to 90 days after the report, or sooner once a fix is released |

We will credit you in the release notes unless you prefer to remain anonymous.

## Scope

The following are examples of issues we consider in scope:

- **Code execution from untrusted files:** importing a recording or loading a task spec causes any content from the file to run (the project requires that this never happens, apart from setup/teardown commands the user has explicitly configured).
- **Path traversal or file overwrite** when importing, exporting, or deleting recordings.
- **Network exposure:** the local viewer or any service accepting connections from outside the local machine by default.
- **Privacy failures:** redaction that fails to mask content it was configured to mask; sensitive data written to logs, exports, or reports when it should not be.
- **Unexpected network activity or telemetry** in default configuration.
- **Safety-policy bypass:** actions or applications that are configured as blocked can still be performed.
- **Unsafe handling of secrets** such as keys or tokens in configuration or recordings.
- **Vulnerable or malicious dependencies** that affect droidtape users.

Out of scope (please report these to the relevant party instead):

- Vulnerabilities in the Android operating system, Android developer tools, or a device manufacturer's software.
- Vulnerabilities in third-party agents, models, or MCP clients that connect to droidtape.
- Issues that require an attacker to already have full control of your machine or an unlocked, debugging-enabled device.
- Behavior of an agent that you instructed to perform an action, where the tool did exactly what was configured.
- Findings from automated scanners without a demonstrated impact.

## Safe handling of recordings

Recordings are sensitive by nature. As a user:

- Treat recordings like screenshots of your device: they can contain messages, credentials, and personal data.
- Configure redaction before recording anything real, and use a dedicated test device or test accounts where possible.
- Do not commit recordings to version control. The repository's `.gitignore` excludes the default recordings folder.
- Review and redact a recording before sharing it in an issue or with anyone else.

## Safe harbor

We support good-faith security research. If you follow this policy, avoid accessing data that isn't yours, and do not disrupt other people's systems, we will not pursue or support legal action against you for your research.
