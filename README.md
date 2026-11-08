# droidtape

**Record, replay, and regression-test AI agents that operate Android devices.**

> ⚠️ **Status: pre-alpha.** Under active development. Nothing is usable yet, and APIs and file formats will change. Any package published under this name is a placeholder with no functionality.

## Why

AI agents that drive a phone (tapping, typing, reading the screen) are non-deterministic and hard to debug. When an agent fails, you can't see exactly what it saw or why it acted, and when you change a prompt or model you can't tell whether you made things better or worse.

droidtape gives you a "tape" of every agent run so you can inspect it, replay it, and test against it.

## What it will do

- **Record**: capture each step of an agent run: screenshot, UI tree, the agent's action, latency, and cost.
- **Replay**: feed a recorded run's observations back to your agent without a device, and detect the first point where the agent's behavior diverges. (Your agent may still call its model during replay.)
- **Regression-test**: run a task N times on a device or emulator and compare pass rate, steps, and cost against a baseline, with CI-friendly output.
- **Diff**: see exactly where two runs diverged.

Agents are meant to connect through a standard agent-facing interface, so existing agents can be recorded without code changes.

## Requirements

- Android SDK platform-tools (`adb`) installed and on your `PATH`
- An Android device (recommended) or emulator, with developer debugging enabled

## Quick start

Coming soon. Watch or star the repo to follow progress.

## Privacy note

Recordings contain screenshots and UI content from your device and may include personal information. Recordings are ignored by git by default; never commit recordings from a real account.

## Documentation

- [Software Requirements Specification](docs/SRS.md)
- [Open questions](docs/open-questions.md)
- [Design decisions](docs/decisions/)

## Contributing

Contributions are welcome. Please read [CONTRIBUTING.md](CONTRIBUTING.md) and our [Code of Conduct](CODE_OF_CONDUCT.md) first. Commits must be signed off (DCO). To report a security issue, see [SECURITY.md](SECURITY.md).

## License

Licensed under the [Apache License 2.0](LICENSE).
