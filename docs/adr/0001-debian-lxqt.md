# ADR 0001 · Debian base and LXQt desktop

**Status** — Proposed for Stage 0 review

Use Debian 13 (Trixie) on amd64 as the first prototype base, with LXQt and Debian-packaged components where practical. Record the installer image checksum, configured package sources, installed versions, and setup steps so someone can recreate and compare a build. Keep security updates enabled and record version changes when rerunning a benchmark.

Use the existing LXQt panel, session, file manager and system tools for the first build. Their memory, login, lock, network and audio behavior establish the baseline for later Qt/QML components. Use separate session-specific configuration to compare the desktops without changing the baseline services.

Stage 1 begins with a disposable VM. Compare on physical hardware before certifying performance, hardware support or a default session.

- [Debian 13 release information](https://www.debian.org/releases/trixie/)
- [LXQt Wayland sessions](https://lxqt-project.org/wiki/Wayland-Session)
- [Requirements](../requirements.md)
