# Veloce OS requirements

This register maps the [README specification](../README.md) to the evidence needed for the first prototype and v1. Required checks block the relevant milestone. Stretch goals are reported separately. An unrun check stays **Not run**.

| ID | Priority | Acceptance evidence | Gate |
|---|---|---|---|
| BASE-001 | Required | Recreate a Debian 13 amd64 desktop from a versioned package manifest and setup instructions or script. LXQt sessions for Labwc, Wayfire and Xorg/Openbox boot, log in, lock and log out. Record image and package versions. | Stage 1 |
| PERF-001 | Required | Each of five idle samples is at or below 1,024 MiB on each supported reference configuration for both shipped appearance profiles. Follow the [README benchmark protocol](../README.md#what-the-1-gib-requirement-means) and retain raw samples with services enabled. | Milestone 1 baseline, Stages 5 and 8 shipped profiles |
| PERF-002 | Stretch | All five samples are at or below 800 MiB under the same conditions as PERF-001. | Report alongside PERF-001 |
| PERF-003 | Required | Record comparable idle CPU, wakeups, GPU, thermals and battery results. Use those results when choosing the compositor. | Stages 4 and 8 |
| CORE-001 | Required | Complete the [physical hardware checklist](hardware-test-matrix.md) for each supported device. Record hardware-specific limitations and evidence. | Stages 2 and 8 |
| APPS-001 | Required | Run the [application workflows](compatibility.md) from a clean baseline. Verify installations, applicable updates, and real usage across the supported browsers and optional app profiles. | Stage 3 |
| MEET-001 | Required | Install the official Zoom package, join a real meeting, and verify one application window and a selected display in the native client on Xorg. | Milestone 1 and Stages 3 and 8 |
| MEET-002 | Required | In Meet and Zoom Web, verify one browser tab, one application window and one selected display in both the shipped Firefox ESR and Chrome. Run the same tests in the installed Chrome-based Meet web app. Record results on Xorg and both Wayland candidates. | Milestone 1 baseline, Stages 3 and 8 v1 verification |
| MEET-003 | Required for a Wayland default | Pass MEET-002 and validate the Notes capture protection in NOTES-001 on the proposed default Wayland stack. Xorg can serve as a provisional prototype default while capture support is evaluated. Track browser and client gaps separately. Native Zoom application windows use the Xorg session per MEET-001. | Stage 4 |
| NOTES-001 | Required for v1 | Notes has a pin toggle for always-on-top placement. Notes stays visible to the presenter and is absent from all supported meeting captures, including full-display sharing, with either pin state. A session must establish protection before showing Notes. Run the [privacy checks](compatibility.md#notes-privacy-tests) on the shipped default session and on any other session that offers Notes. | Milestone 1 feasibility, Stage 4 session choice, Stages 5 and 8 v1 verification |
| SEC-001 | Required | Verify encrypted install and recovery, package trust, updates, firewall, session lock, permissions, and backup restore on disposable data. | Stages 6–8 |
| UX-001 | Required | Verify dock, Island, notifications and controls, keyboard navigation, reduced motion and crash isolation on the shipped profiles. | Stages 5 and 8 |
| REL-001 | Required | Record repeat boot and suspend/resume, reconnect, update and recovery results with no critical boot, data-loss or update failures. | Stages 6–8 |
| DIST-001 | Required | Boot a versioned live image and verify installer, bootloader, firmware, supported storage layouts, and recovery. Record build provenance and integrity. | Stages 7–9 |

MEET-003 is a proposed Stage 0 decision for review. Firefox ESR parity covers browser-tab, application-window and display capture. Verify true tab capture separately from sharing a browser window or a display. Log any unavailable mode as **Blocked** for that client and session, with the reason and a follow-up. The v1 criteria stay open until the required modes work or the owner explicitly revises scope.

## Stage 0 exit checklist

- [x] Public repository, README, ignore rules and draft original-material license exist.
- [x] Draft requirement IDs, priorities, acceptance criteria and test links.
- [x] Draft architecture decisions, compatibility scenarios, issue templates and security reporting policy.
- [x] Prepare a hardware inventory and driver checklist.
- [x] Identify and record the first physical reference laptop (HW-01).
- [ ] Review requirements and the test strategy, including the proposed meeting default gate.
- [x] Record the owner’s open public-release review items below.

## Public release review

| Open item | Evidence to record before public distribution |
|---|---|
| Project name | Owner records the naming review and decision before publishing a branded installer or release. |
| Original-material license | Owner obtains qualified review of the [draft license](../LICENSE) and records the outcome. |
| Third-party components | Owner records the bundled component inventory, attribution and source obligations where applicable, and any vendor permissions before distributing images. |
| Private vulnerability reporting | Maintainer enables and verifies a working private reporting route. |

Stage 1 can begin in a VM while the remaining hardware details are collected. The VM is suitable for setup and boot checks. Physical hardware is required for certification of the idle memory budget and laptop workflows.
