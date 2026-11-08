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
| MEET-002 | Required | In Meet in Chrome, the installed Meet web app, and Zoom Web in Chrome, verify one browser tab, one application window and one selected display on Xorg. Test the same combinations on both Wayland candidates and document the available sources. | Milestone 1 and Stages 3 and 8 |
| MEET-003 | Required for a Wayland default | Pass the MEET-002 web-client scenarios on the proposed default Wayland stack before certifying it as the default. If neither candidate passes, keep Xorg as the proposed default until a Wayland candidate passes. Native Zoom application windows use the Xorg session per MEET-001. | Stage 4 |
| SEC-001 | Required | Verify encrypted install and recovery, package trust, updates, firewall, session lock, permissions, and backup restore on disposable data. | Stages 6–8 |
| UX-001 | Required | Verify dock, Island, notifications and controls, keyboard navigation, reduced motion and crash isolation on the shipped profiles. | Stages 5 and 8 |
| REL-001 | Required | Record repeat boot and suspend/resume, reconnect, update and recovery results with no critical boot, data-loss or update failures. | Stages 6–8 |
| DIST-001 | Required | Boot a versioned live image and verify installer, bootloader, firmware, supported storage layouts, and recovery. Record build provenance and integrity. | Stages 7–9 |

MEET-003 is a proposed Stage 0 decision for review. App-wide sharing that automatically follows every window of a program is a separate, open scope decision. The required scenarios cover a single selected application window.

## Stage 0 exit checklist

- [x] Public repository, README, ignore rules and draft original-material license exist.
- [x] Draft requirement IDs, priorities, acceptance criteria and test links.
- [x] Draft architecture decisions, compatibility scenarios, issue templates and security reporting policy.
- [x] Prepare a hardware inventory and driver checklist.
- [ ] Identify and record the first physical reference laptop.
- [ ] Review requirements and the test strategy, including the proposed meeting default gate.
- [x] Record the owner’s open public-release review items below.

## Public release review

| Open item | Evidence to record before public distribution |
|---|---|
| Project name | Owner records the naming review and decision before publishing a branded installer or release. |
| Original-material license | Owner obtains qualified review of the [draft license](../LICENSE) and records the outcome. |
| Third-party components | Owner records the bundled component inventory, attribution and source obligations where applicable, and any vendor permissions before distributing images. |
| Private vulnerability reporting | Maintainer enables and verifies a working private reporting route. |

Stage 1 can begin in a VM while the hardware entry is being completed. The VM is suitable for setup and boot checks. Physical hardware is required for certification of the idle memory budget and laptop workflows.
