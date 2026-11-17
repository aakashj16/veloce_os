# ADR 0003 · Original visual identity and independent performance modes

**Status — Accepted product direction (2026-11-10); performance and implementation unverified.**

## Context

The original roadmap proposed a macOS-inspired glass interface with blur as a central feature and Veloce Glass/Light as appearance profiles. The product goal has evolved: Veloce should be a distinctive, modern, lightweight productivity desktop rather than a macOS reproduction. Stage 1 testing favored the simplicity and reliability of LXQt on Xorg/Openbox over the tested Wayland sessions; no production session or physical-hardware performance gate has yet been certified.

## Decision

1. **One OS, two independent performance modes.** **Standard** is the efficient baseline, with a hard idle system-memory ceiling of **1,024 MiB (1 GiB)** and a non-blocking stretch goal of 800 MiB. **Enhanced** adds optional subtle effects, with a preferred optimization target ≤ 1,024 MiB and an absolute idle ceiling of **2,048 MiB (2 GiB)**. Both deliver the same core capabilities. Enhanced is not a separate installer or distribution.
2. **Two independent visual themes.** **Graphite** is a matte charcoal/dark theme; **Pearl** is a light neutral theme. Both appear in either Standard or Enhanced. Theme selection must not implicitly change the performance mode.
3. **Original matte-first design.** Achieve visual polish through typography, spacing, original icons, precise color and contrast, layering, subtle borders, rounded surfaces where supported, dock and status hub. Avoid live blur as a dependency, pervasive translucency, and proprietary Apple designs or assets. Keep effects optional, short and reducible; respect accessibility settings.
4. **Provisional display baseline.** Use LXQt + Xorg/Openbox as the first implementation candidate based on Stage 1 observations, especially session dependability and native Zoom application-window sharing. Keep LXQt + Labwc and Wayfire as test candidates; Wayfire is not required by Enhanced. The production default remains subject to Stage 4/8 evidence and the meeting/privacy acceptance gates.
5. **Hardware planning, not certification.** Standard should remain practical on 4 GiB installed-memory devices where supported; Enhanced should primarily target 8 GiB+ machines. Idle system-memory consumption must not be confused with installed RAM. CPU/GPU, frame times, thermal behavior and battery matter alongside memory. Only real-device test results establish compatibility.
6. **No silent downgrades or misleading claims.** Unsupported effects should degrade safely; document fallback behavior. Do not disable critical services to meet budgets. All four mode × theme combinations must be validated under the same reproducible five-sample measurement method in the README.

## Alternatives considered

- **macOS-style glass as core identity:** Rejected due to higher implementation, graphics, battery, maintainability and brand-similarity costs without equivalent user benefit.
- **Separate Standard and Enhanced distributions:** Rejected due to packaging/update fragmentation and duplicated maintenance.
- **Wayfire mandatory for Enhanced:** Rejected; matte appearance does not require a blur-oriented Wayland compositor.
- **One fixed appearance for all devices:** Rejected; configurable performance with consistent themes better serves both older and newer laptops.

## Consequences

- Update requirements, README, Stage 4 and 5 gates, risk register and future mockups to the new terms.
- Replace references to “Veloce Glass/Light” as shipping profiles with Standard/Enhanced; Graphite/Pearl are theme names.
- Retain existing functional requirements, including screen sharing, protected Notes **as an unverified v1 gate**, developer apps, and security.
- Do not retroactively mark untested milestones complete. Stage 2 can begin once the documentation revision is reviewed and accepted; physical laptop acceptance still requires hardware evidence.

## Measurement and release gates

- Standard: five idle samples **each ≤ 1,024 MiB** on supported hardware, both themes; 800 MiB stretch.
- Enhanced: five idle samples **each ≤ 2,048 MiB**, both themes; aim ≤ 1,024 MiB when feasible.
- Test after cold reboot, 10-minute idle stabilization, one sample per minute with complete baseline services operational. Record session, hardware, mode, theme, display configuration, CPU/GPU/battery metrics and sample data.
- UX switching, keyboard access, reduced motion and effect fallback must pass. No results are certified as of this ADR.

See [README](../../README.md), [requirements](../requirements.md), [ADR 0002](0002-desktop-sessions.md) and [hardware matrix](../hardware-test-matrix.md).
