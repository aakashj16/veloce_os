# Stage 2 · HW-01 hardware foundation

**Status** — Complete as of 2026-11-18 with graphics validation carried forward.

The owner confirmed that the applicable physical checks performed on the HP 15s-dr2007tx passed without errors. The test system was ARIA, an installed Debian 13 workstation running KDE Plasma on Wayland. Native Zoom passed microphone, speaker, webcam, application-window sharing and entire-display sharing. See the [hardware matrix](hardware-test-matrix.md) and the [ARIA field report](field-reports/aria-debian13-2026-11-17.md).

Veloce now uses **Debian 13 with KDE Plasma on Wayland** as its desktop baseline. [ADR 0002](adr/0002-desktop-sessions.md) records the decision.

## Follow-up

- Validate Intel UHD and NVIDIA MX130 drivers and hybrid graphics. Verify external-display output when a monitor is available.
- Capture the reproducible KDE image and baseline service configuration. Run the five-sample idle benchmark on HW-01 with normal services running. ARIA's reported idle usage was below 2 GiB RAM and 1% CPU. Stage 1's 725 MiB LXQt reading came from a VM. A matched benchmark is needed to establish relative memory usage and check the 1 GiB Standard target.
- Verify shipped browser and Zoom workflows, protected Notes capture, and release installation, encryption, backup and recovery in their roadmap stages. Use disposable data and a disposable target for recovery tests.
