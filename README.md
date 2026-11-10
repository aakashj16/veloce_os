# Veloce OS

**A lightweight Linux desktop, built for real work.**

> **Status:** Planning / architecture specification (v0.2)  
> **Document date:** 2026-11-08  
> **Primary architecture:** x86-64 (`amd64`) laptops and desktops  
> **Foundation:** Debian 13 (Trixie), Linux kernel, LXQt, Qt/QML  
> **Primary performance goal:** **≤ 1 GiB (1,024 MiB) idle system memory** with the full baseline desktop and system services operational  
> **Repository:** `veloce_os` (public GitHub repository, source-available)  
> **Project type:** Custom Linux distribution and desktop experience — **not** a new operating-system kernel

## 1. Vision

Veloce OS aims to replace Windows for day-to-day computing and developer workflows without sacrificing essential hardware support, meeting software, security, or usability. It is intended for its creator **and anyone seeking an efficient, approachable desktop OS**, subject to the licensing and third-party software terms described below.

The ideal experience is:

- **Lightweight:** Minimize always-running processes and idle RAM/CPU consumption.
- **Complete:** Work reliably with laptop hardware, networking, meetings, browsers, messaging, productivity tools, and development tools.
- **Elegant:** macOS-inspired translucency, a floating dock, thoughtful animations, a compact top bar, and an expandable **Dynamic Island–style** status interface.
- **Reliable:** Sensible defaults, secure updates, backups, diagnostics, tested suspend/resume, and safe installation/recovery.
- **Maintainable:** Reuse upstream Linux components rather than rewriting proven subsystems; keep custom features modular and version-controlled.

**Important:** This is a project specification, **not a claim that a working ISO exists**. Every feature below is a requirement or planned integration until it passes the stated acceptance tests.

### Name and distribution notice

**Veloce OS is a working project name**, not yet cleared for public branding. Siemens has used the term **“Veloce OS”** for enterprise prototyping software. Because the repository is public, complete appropriate name/trademark/domain research **before publishing installers, logos or a branded release** and, if necessary, select a distinct public product name. Do not imply affiliation with Siemens. The visual design must be independently created and must not reproduce Apple trademarks or proprietary assets.

## 2. Non-negotiable success criteria

A milestone is complete only when its requirements are **tested and documented**, not merely installed.

| ID | Requirement | Release acceptance |
|---|---|---|
| PERF-001 | **Idle system memory ≤ 1,024 MiB** | Five consecutive measurements after a 10-minute idle settling period on each reference laptop/configuration; normal services enabled; no user apps open |
| PERF-002 | Preferred idle memory ≤ 800 MiB | Stretch goal; report actual measurements, do not claim until achieved |
| PERF-003 | Low idle CPU / battery impact | Capture CPU, wakeups, GPU use, thermals, power draw and battery runtime; establish baselines against installed Windows and/or stock Debian |
| CORE-001 | Complete laptop and desktop functionality | All applicable hardware tests in Section 5 pass on supported devices |
| APPS-001 | Real productivity | Firefox, Chrome, Edge and Brave compatibility; VLC; browser-only Telegram and WhatsApp; Zoom, VS Code, Python, SQL, Google Workspace, Microsoft 365 web, and optional offline office validated |
| MEET-001–003 | Meeting screen sharing | Test native Zoom on Xorg and Meet/Zoom Web by session, browser and capture source using the [meeting acceptance matrix](docs/compatibility.md#meeting-tests) |
| NOTES-001 | Private presenter notes | Verify the pin toggle and [capture exclusion](docs/compatibility.md#notes-privacy-tests), including on a shared display that locally shows Notes |
| SEC-001 | Secure and maintainable | Encryption, updates, firewall, permissions, backups and recovery tested |
| UX-001 | Functional elegant UI | Dock, notifications, system controls, accessibility basics and Island work without preventing core workflow |
| REL-001 | Boot, sleep, resume and updates work | Repeated end-to-end testing; no critical data-loss, boot or update regressions |
| DIST-001 | Installable distribution | Bootable live ISO; tested installer, bootloader, firmware and rollback/recovery instructions |

### What the 1 GiB requirement means

The target is **idle OS memory**, not the total RAM used while applications are running and **not** a claim that a machine with 1 GiB installed RAM can run Zoom, browsers, VS Code, and development tools.

- Measure a fresh, fully logged-in **Veloce Glass** session with Wi-Fi, Bluetooth, audio, notifications, file management, power management, firewall, session/lock services and ordinary background system services operational.
- No browser, Zoom, IDE, database server, media player, office suite, or messaging window open during the idle benchmark. Required services must not be disabled merely to pass the benchmark.
- Where possible, keep resource-intensive applications **installed but not running** until requested. Databases, container daemons, cloud syncing and indexing should be on-demand or explicitly enabled when needed.
- Use `MemTotal - MemAvailable` from `/proc/meminfo` as the **primary system-memory-used proxy**, converted to MiB; also collect `free -m`, per-process RSS/PSS (where available), kernel/slab stats, swap/zram state and GPU shared-memory caveats.
- Reboot between benchmark runs; wait 10 minutes after login, then take five samples at 60-second intervals. Record hardware, kernel, image hash, compositor, theme, screen resolution, network state, enabled services, and sample-level outputs.
- Compare equivalent configurations. A virtual machine is useful for early iteration **but is not sufficient** to certify the memory target on real laptops.
- If a full glass effect breaks the hard limit, optimize it or use a different implementation. **The requirement does not disappear.**

**Initial practical hardware planning assumption:** 4 GiB installed RAM may be enough to test a basic OS session, but **8–16 GiB or more** is preferable for modern browsers, video meetings, IDEs and development workloads. Final minimum hardware requirements will follow real testing, not guesswork.

## 3. Proposed architecture

```text
┌───────────────────────────────────────────────────────────────┐
│ VELOCE EXPERIENCE                                            │
│ Veloce Dock | Veloce Island | Control Center | Notifications │
│ Settings | Themes | First-run Welcome | Accessibility       │
│ Custom Qt/QML components, isolated and failure-tolerant      │
├───────────────────────────────────────────────────────────────┤
│ DESKTOP / COMPOSITOR                                         │
│ LXQt + Wayfire [candidate] / LXQt + Labwc [candidate]        │
│ Wayland + XWayland; LXQt + Xorg/Openbox compatibility session│
│ Display manager, lock screen, session and window management  │
├───────────────────────────────────────────────────────────────┤
│ APPLICATIONS                                                 │
│ Firefox | Chrome | Edge | Brave | Zoom | VLC                 │
│ Telegram Web | WhatsApp Web | Google Workspace | M365 web    │
│ VS Code | Python | SQL | Git | SSH | Docker | Office options │
│ File manager | Terminal | PDF | Archives | Google Drive      │
├───────────────────────────────────────────────────────────────┤
│ SYSTEM SERVICES                                              │
│ NetworkManager | BlueZ | PipeWire | WirePlumber | UPower     │
│ logind | libinput | portals | udev | notifications | polkit  │
│ fwupd (when supported) | firewall | package updates         │
├───────────────────────────────────────────────────────────────┤
│ DEBIAN 13 / LINUX                                            │
│ Linux kernel, drivers, APT, systemd, filesystem, EFI/GRUB    │
│ LUKS2 encryption, trusted firmware, storage and hardware I/O │
└───────────────────────────────────────────────────────────────┘
```

### Foundation decisions

| Layer | Proposed choice | Why / open question |
|---|---|---|
| Base | **Debian 13 Stable (Trixie)** | Security maintenance, predictable APT repository, broad hardware/software ecosystem; pin suite/version for builds |
| Architecture | **amd64 first** | Highest priority for common laptops, Zoom/VS Code and requested desktop apps; ARM64 is a later evaluation |
| Desktop | **LXQt** | Modular, Qt-based, adaptable, with Wayland compositor options |
| Compositor candidate A | **Wayfire** | Visual effects and animations suitable for glass design; must validate memory/GPU cost |
| Compositor candidate B | **Labwc** | Leaner baseline to benchmark against Wayfire; custom effects may require additional implementation |
| Compatibility session | **LXQt on Xorg**, likely Openbox | Essential fallback for Zoom application-window sharing and legacy apps; test explicitly |
| Custom interface | **Qt/QML** | Build isolated native-feeling widgets and UI rather than replacing every subsystem |
| Graphics compatibility | **XWayland** | Support for X11 applications within a Wayland session where compatible |
| Sound | **PipeWire + WirePlumber + ALSA** | Multimedia routing, microphones, jack, Bluetooth audio |
| Network | **NetworkManager + BlueZ** | Wi-Fi, wired Ethernet, Bluetooth connectivity |
| Packages | **APT** by default; optional **Flatpak** | Prefer verified upstream/official packages, explicit third-party repository trust |
| Image creation | **Debian `live-build`** | Configure repeatable ISO builds |
| Installer | **Calamares**, subject to integration tests | Graphical installation, encryption/partition and bootloader scenarios |

**Decision gate:** Do not lock in Wayfire (or Labwc) until both have been compared for RAM, idle CPU, GPU/blur impact, suspend/resume, screen sharing, multi-monitor behavior, accessibility, and crash recovery.

### Glass appearance without unnecessary weight

Two desktop profiles will be shipped:

1. **Veloce Glass (default):** Floating dock, translucency, gentle blur, rounded corners, minimal top bar, adaptive Island, short animations.
2. **Veloce Light:** Reduced blur and motion, identical basic controls and usability, useful on older GPUs or on battery.

Both must meet PERF-001 on designated supported hardware. Heavy GPU effects are never mandatory for notification delivery, audio controls or core desktop operation.

### Modular custom components

- **Veloce Dock:** Launch and pin applications, active app indicators, keyboard navigation, responsive layout, multi-monitor behavior.
- **Veloce Island:** Expands for volume, media, battery, Wi-Fi/Bluetooth events, screen capture feedback, timers/alerts, and call indicators **where event/permission APIs allow**.
- **Veloce Control:** Volume, input/output device, brightness, airplane mode, network, Bluetooth, battery and power profile controls.
- **Veloce Notify:** Native notifications with Do Not Disturb, action buttons where supported, history, privacy rules and accessibility.
- **Notes:** A local notes app with a pin toggle for always-on-top placement. In a supported session, you can read Notes over your demo while meeting attendees see the demo underneath, even during full-display sharing. See [protected Notes](docs/adr/0002-desktop-sessions.md#protected-notes-surface).
- **Veloce Settings:** Appearance, performance mode, power, devices, update status, privacy and accessibility preferences.
- **Veloce Welcome:** First boot setup, user language/keyboard, optional app installation and Google Drive integration.

Each component should degrade gracefully: if Island crashes, audio, networking, notifications and desktop login must continue to function.

## 4. Application compatibility, browser-first apps and packaging

All capabilities listed here are part of the **baseline compatibility scope**. To control the installation footprint and background processes, **a compatible application does not necessarily need to be preinstalled**. App availability on Linux is not the same as verified support on Veloce OS.

### Default vs on-demand installation

| Tier | Default applications / behavior | Reason |
|---|---|---|
| **Core image** | Firefox (initially Firefox ESR), VLC, terminal, file manager, PDF and archive tools, system controls | A useful desktop out of the box without four overlapping browser installations |
| **Optional browser installs** | Google Chrome, Microsoft Edge, Brave via official vendor-provided Linux packages/repositories | All three are **first-class tested options**, but no unnecessary installation or startup daemons |
| **Browser-only communication** | Telegram Web and WhatsApp Web opened in the user's preferred supported browser; optional web shortcuts | No native Telegram or WhatsApp client in v1; no separate always-running messaging processes |
| **Browser productivity** | Google Workspace and Microsoft 365 for the web through browser bookmarks/shortcuts | No native Google/Microsoft office package or background service required |
| **Optional offline productivity** | LibreOffice Writer, Calc and Impress (user-selected install) | Offline editing of common document formats without requiring the suite in the minimal image |
| **User/developer installations** | Zoom, VS Code, SQL tooling, Python/Jupyter extras, Git/SSH, optional Docker and local databases | Keep development/server components on demand; meet complete compatibility tests |

**Install policy:** Do not bundle proprietary Chrome, Edge, Brave, Zoom or other vendor binaries in a downloadable ISO until the vendor's redistribution terms are reviewed. Offer official download links or signed vendor repositories instead. Browser download and updates must remain possible after a clean install.

### Mandatory app and workflow matrix

| Application / workflow | Installation / access approach | Mandatory validation |
|---|---|---|
| **Mozilla Firefox** | Debian `firefox-esr` as initial default; evaluate Mozilla official APT for rapid release | Profiles, extension updates, media, file downloads, permissions, printing and web conferencing |
| **Google Chrome** | Official Google Linux `.deb` / vendor-managed updates (optional install) | Full browser lifecycle, security updates, hardware acceleration, microphone/webcam and web-office apps |
| **Microsoft Edge** | Official Edge Linux `.deb` / vendor-managed updates (optional install) | Same browser tests, Microsoft 365 web access, app shortcuts, and update integration |
| **Brave** | Official Brave signed APT repository (optional install) | Media, downloads, privacy settings, conferencing, updates, site compatibility |
| **Telegram** | **Telegram Web only** at `https://web.telegram.org/` in the chosen browser | Login, messaging, attachments, notifications when permitted, logout, session persistence; call features only where available in the chosen web version |
| **WhatsApp** | **WhatsApp Web only** at `https://web.whatsapp.com/` in the chosen browser | Device linking, chat, file transfers, browser permissions, supported notifications and calls where available |
| **VLC media player** | Debian `vlc` package, part of core image | Local MP3/MP4, volume, subtitles, file associations, fullscreen, hardware acceleration where available, removable media |
| **Google Workspace** | Web: Drive, Docs, Sheets, Slides, Gmail via supported browser | Authentication, file creation/editing, sharing, downloads/exports, printing, offline behavior clearly documented |
| **Microsoft 365** | Web: Word, Excel, PowerPoint, OneDrive, Outlook via supported browser | Authentication, edit/save, downloads, printing, advanced-feature limitations noted |
| **LibreOffice (optional)** | Debian packages installed on demand | Offline Writer/Calc/Impress, DOCX/XLSX/PPTX import/export; compatibility review for complex files |
| **Zoom** | Official Zoom Linux package, installed on demand | Sign-in, real meetings, camera, mic, headset switching, screen/window sharing in supported session |
| **VS Code** | Official Microsoft Linux package / signed repository, on demand | Extensions, Git, Python environment, terminal and remote development |
| **Python** | Debian Python plus `venv`, user environments, optional Jupyter | Isolated package installs, scripts, notebooks and plotting |
| **SQL** | SQL clients, optional DBeaver, PostgreSQL/MySQL clients and engines | Local/remote DB connectivity, SSH tunnels, credentials, imports/exports |
| **Terminal** | QTerminal | Tabs, Unicode, SSH, copy/paste, keyboard shortcuts |
| **Files** | PCManFM-Qt | Drag/drop, removable storage, file permissions and network shares |
| **Git / SSH / Docker** | Git and SSH tools; Docker daemon optional/on-demand | Repository workflows, key storage, containers and stopped-daemon RAM behavior |
| **Google Drive files** | Browser by default; optional vetted `rclone` or filesystem integration | Browser read/write, optional mount sign-in and error recovery |
| **Everyday productivity** | Lightweight PDF viewer, archives, screenshots, clipboard, printing | Everyday file, PDF and sharing operations |

### Web messaging: strictly browser-only in v1

- **Telegram:** `https://web.telegram.org/`.
- **WhatsApp:** `https://web.whatsapp.com/`.
- Both must open in Firefox, Chrome, Edge and Brave as supported by their current websites. Provide optional bookmarks or browser-managed standalone shortcuts **without packaging a native app or separate Electron wrapper**.
- Browser notifications require user permission and browser processes; they are **not** expected to work when all relevant browser processes are closed.
- Validate messaging, attachments and login sessions. Calling and screen-sharing capabilities are website/browser/account dependent, and Veloce does not promise desktop-native feature parity.
- Do not include login tokens, cookies, credentials or personal account configuration in images or repositories.
- A **future post-v1 evaluation** may consider optional Telegram Desktop or native messaging integrations; these are **explicitly excluded from the initial release**.

### Office productivity policy

- **Online:** Google Workspace works through modern browsers; Microsoft 365 for the web provides Word, Excel and PowerPoint functionality. Each service may require a user account or subscription.
- **Offline:** LibreOffice is the initial optional office suite. Complex Microsoft Excel macros, add-ins, advanced formatting and Power Query are **not** guaranteed to behave like native Windows Excel.
- There is **no officially supported native Microsoft Office desktop suite for Linux** in this plan. Don't present a Wine workaround as full compatibility.
- Google Drive **for desktop** is not officially offered on Linux; use the website or a separately tested third-party mount solution.
- Benchmark disk consumption separately for the minimal base and each installed browser/office profile. Optional applications must not add always-running daemons unless the user enables them.

### Meeting screen sharing and the Xorg session

As documented by Zoom, on Linux **Wayland sessions support entire-desktop or whiteboard sharing but not specific application-window sharing**; application-window sharing requires Xorg. Therefore:

- Provide and test the primary **Wayland** desktop with XDG Desktop Portal + PipeWire integration.
- Provide and test an **Xorg session** for native Zoom application-window sharing.
- Verify browser tab, one application window, and selected display sharing in Google Meet and Zoom Web with both Firefox ESR and Chrome. Test the same modes in the installed Chrome-based Meet web app. For native Zoom, test an individual window and a selected display on Xorg.
- Compare both Wayland candidates with the actual portal backend. A Wayland session must pass the browser meeting matrix and the Notes privacy checks before it can become the default. See [meeting and Notes requirements](docs/requirements.md) for the session criteria.
- Certify real meetings with microphones, webcams, Bluetooth headsets, multiple monitors, share audio, join/leave and suspend/resume.
- Do not describe Zoom as “fully working” until the required scenarios pass.

## 5. Complete hardware and operating-system test matrix

Use **PASS / FAIL / BLOCKED / NOT APPLICABLE**, record test machine and kernel version, and attach evidence/logs for failures.

### Connectivity and external devices

- [ ] Wi-Fi: scan, connect/reconnect, WPA2/WPA3 where hardware supports it, suspend/resume
- [ ] Ethernet and USB network adapters
- [ ] Bluetooth: pair/unpair, mice, keyboards, headphones, file/device behavior where supported
- [ ] USB-A / USB-C: flash drives, hubs, phones, webcam, keyboard/mouse, safe eject
- [ ] External SSDs and drives: read/write **NTFS**, **exFAT**, **ext4** as applicable
- [ ] Display output: HDMI/DisplayPort/USB-C, multi-monitor, scaling and hotplug
- [ ] Network shares and Google Drive integration

### Audio, video and meetings

- [ ] Internal speakers and hardware volume/mute keys
- [ ] 3.5 mm headphone/microphone jack, jack detection and headset mic routing
- [ ] Bluetooth audio profile changes and microphone selection
- [ ] Built-in webcam and privacy switch/permissions
- [ ] Internal/external microphones and input levels
- [ ] Browser audio/video calls; Zoom meeting join/host and screen sharing
- [ ] Notification sounds, media control keys and microphone/camera indicators

### Laptop ergonomics and performance

- [ ] Touchpad tap, two-finger scroll, gestures and pointer settings
- [ ] Keyboard shortcuts, function keys and keyboard-layout switching
- [ ] Brightness controls, dimming, external display backlight support when available
- [ ] Battery state, charging status, battery saver and power profiles
- [ ] Lid-close, sleep/suspend, resume and repeated wake cycles
- [ ] Thermal behavior and fan management through safe kernel/firmware interfaces
- [ ] High-DPI and fractional scaling, external monitor arrangements
- [ ] Screenshot, clipboard history, screen locking and notifications
- [ ] Performance after long uptime, low-storage and low-memory behavior

### Core OS, data and security

- [ ] Encrypted installation with **LUKS2** and tested unlock/recovery procedure
- [ ] Separate user/admin privileges with `sudo`/polkit and safe permission prompts
- [ ] Firewall enabled with sensible default inbound policy
- [ ] Signed package sources; update install, reboot and recovery from failed update
- [ ] Session screen lock, privacy and secure handling of credentials/secrets
- [ ] Disk health, system logs, crash reporting and diagnostic export
- [ ] Backup and restore of a sample user profile, plus bootable rescue path
- [ ] Firmware update support where supported by the manufacturer / LVFS
- [ ] UEFI/GRUB boot, Windows coexistence/dual boot if selected
- [ ] Secure Boot behavior documented and validated, without assuming all hardware works
- [ ] Accessibility: keyboard-only navigation, readable scaling and screen-reader feasibility assessment

**Support policy:** A hardware-dependent test may legitimately fail on a specific device because of proprietary firmware or unsupported drivers. List supported laptop models and any limitations explicitly; do not claim universal compatibility.

## 6. Performance and reliability engineering

### Metrics to track for every tagged build

| Dimension | Measurement | Target / status |
|---|---|---|
| RAM at idle | `MemTotal - MemAvailable` in MiB | **Hard: ≤ 1,024 MiB**; stretch ≤ 800 MiB |
| Idle CPU | Per-core and whole-system CPU utilization | Record baseline; improvement over earlier release |
| CPU wakeups / GPU | Powertop or equivalent, compositor GPU/render metrics | Minimize overhead; no fixed number before hardware benchmark |
| Cold boot to usable desktop | Timestamp from power/boot to usable login/session | Baseline first; optimization later |
| Resume latency | Wake to interactive session/network/audio | Baseline and regression checks |
| Battery use | Discharge rate and repeatable browser/meeting tests | Better than unoptimized baseline where feasible |
| Disk footprint | Minimal OS, base applications, full optional development profile | Publish separate sizes; no fabricated target |
| Reliability | Session crash, failed updates, data integrity, suspend loop | No critical release blockers |
| App compatibility | Installation + real workflows + updates | Mandatory apps meet acceptance checklist |

### Suggested benchmark commands (development only)

```bash
# Memory proxy and swap status
free -m
awk '/MemTotal:|MemAvailable:|SwapTotal:|SwapFree:/ {print}' /proc/meminfo

# Top resident-memory processes
ps -eo pid,comm,rss,%cpu --sort=-rss | head -n 25

# Services and session state
systemctl --failed
loginctl session-status

# Hardware identifiers (sanitize before posting logs publicly)
lspci -nn
lsusb
```

Future `scripts/benchmarks/` tooling should automate measurements into CSV/JSON with comparable metadata, verify the five-sample rule, and fail CI/hardware gates when the budget is exceeded. Do not publish serial numbers, Wi-Fi identifiers, personal usernames or credentials in test artifacts.

### Performance policy

1. Reduce unnecessary background services **without** disabling security updates, firewall, audio, networking, or notifications.
2. Start developer tools and local databases on demand.
3. Avoid duplicate tray, notification, wallpaper, compositor and update daemons.
4. Prefer native, audited system services over maintaining custom replacements.
5. Profile Qt/QML components and compositing effects before shipping.
6. Optimize power, responsiveness and reliability **alongside** RAM; low memory alone is not success.

## 7. Development roadmap and stage gates

**Stage status:** Stage 0 is **COMPLETE** as of 2026-11-09. Stage 1 is **IN PROGRESS**. Stages 2–10 are **PLANNED**. Dates are deliberately not estimated until hardware access, contributors and prototype results are known. Do not mark steps complete without evidence.

### Stage 0 — Requirements, naming and project setup

**Goal:** Convert the vision into a testable, version-controlled engineering project.

**Deliverables**
- [x] Public GitHub repository **`veloce_os`**, README, `.gitignore`, and draft original-material license
- [x] Review the [security reporting policy](SECURITY.md) and issue templates
- [x] Record the prototype architecture decisions (Debian 13, LXQt, Wayfire and Labwc trials, and Xorg compatibility)
- [x] Record and review requirement IDs, priorities, acceptance criteria and test links
- [x] Identify HW-01 and document the baseline hardware inventory and driver compatibility checklist
- [x] Document supported-use scenarios for four browsers, browser-only Telegram/WhatsApp, VLC, office apps, meeting clients, Notes, Python/SQL and VS Code
- [x] Record the [open public-release reviews](docs/requirements.md#public-release-review) for branding, the draft license and third-party distribution

**Exit gate met:** The requirements and test strategy were reviewed, and HW-01 is the first physical reference laptop. See the [Stage 0 checklist](docs/requirements.md#stage-0-exit-checklist), [hardware inventory](docs/hardware-test-matrix.md) and architecture decisions for the [base desktop](docs/adr/0001-debian-lxqt.md) and [session options](docs/adr/0002-desktop-sessions.md).

### Stage 1 — Minimal Debian desktop proof of concept

**Goal:** Demonstrate a minimal, usable graphical Debian desktop before building custom UI.

**Status** — Complete. The [Stage 1 guide](docs/stage-1.md) records the Debian 13 setup, results for all three sessions, and known VM issues.

**Deliverables**
- [x] Reproducible Debian 13 amd64 minimal installation in a virtual machine
- [x] LXQt + Wayfire trial profile
- [x] LXQt + Labwc trial profile
- [x] LXQt + Xorg/Openbox compatibility session
- [x] Login, logout, session lock, basic file manager, terminal and network connection
- [x] Initial hardware-free memory, CPU and boot baselines

**Exit gate passed.** All three sessions passed boot and desktop checks, and their initial measurements and known blockers are documented. Stage 2 starts with LXQt on Xorg/Openbox as the hardware baseline. The default session decision follows the Stage 4 evaluation.

### Stage 2 — Hardware foundation and system services

**Goal:** Make the desktop useful and dependable on physical laptops.

**Deliverables**
- [ ] NetworkManager/BlueZ configuration; Wi-Fi, Ethernet and Bluetooth testing
- [ ] PipeWire/WirePlumber/ALSA for speakers, headphone jack, Bluetooth audio and mic
- [ ] Touchpad, keyboard/function keys, brightness, volume and monitor support
- [ ] Suspend, lid-close, resume, battery telemetry, thermal and power profiles
- [ ] USB, storage, NTFS/exFAT, network shares and device permissions
- [ ] Notification service, screen locker, login/display manager and basic accessibility
- [ ] Device-specific driver/firmware gap list and tested recovery path

**Exit gate:** All applicable baseline hardware checks pass on at least one reference laptop; unsupported hardware issues documented.

### Stage 3 — Browsers, communication and developer applications

**Goal:** Match the real Windows productivity workflow, not just app launch.

**Deliverables**
- [ ] Firefox ESR as default; official Chrome, Edge and Brave optional installations with verified updates
- [ ] **Telegram Web and WhatsApp Web only**; messaging/attachment/browser-notification checks on supported browsers
- [ ] VLC included: MP3/MP4 playback, subtitles, file association, media keys
- [ ] Google Workspace and Microsoft 365 web workflows, plus optional LibreOffice offline workflows
- [ ] Official Zoom install and real meeting tests for [native and browser clients](docs/compatibility.md#meeting-tests)
- [ ] VS Code + extensions; Git, SSH, Python venv, pip and Jupyter workflows
- [ ] SQL GUI, PostgreSQL/MySQL clients, optional local databases and Docker
- [ ] PDF viewer, archive tool, printing, screenshots and clipboard history
- [ ] Google Drive web access; optional vetted `rclone` authentication/browse/read/write integration

**Exit gate:** All four browsers install and update correctly, VLC plays local files, Telegram and WhatsApp function in-browser, online office works, optional LibreOffice handles common files, and Zoom session limitations are documented and tested.

### Stage 4 — Performance baseline and compositor selection

**Goal:** Prove feasibility of the **≤ 1 GiB idle RAM** requirement with full core services.

**Deliverables**
- [ ] Standardized idle benchmark harness and clean-run procedure
- [ ] Five-sample measurements on Wayfire, Labwc and X11 baseline sessions
- [ ] CPU, GPU/blur, battery, suspend and multi-monitor comparisons
- [ ] Services/startup review; remove unnecessary daemons safely
- [ ] Architecture decision: primary compositor and custom-component constraints, including capture protection for Notes
- [ ] Performance regression checks prepared for subsequent builds

**Exit gate:** Chosen baseline meets ≤ 1,024 MiB idle on reference hardware, **or** documented optimization plan plus explicit stage failure; no false success claims.

### Stage 5 — Veloce visual experience

**Goal:** Build the polished macOS-inspired experience on the validated foundation.

**Deliverables**
- [ ] Visual design system: typography, original icons, spacing, blur and colors
- [ ] Floating dock, application switcher, launcher and window behaviors
- [ ] Dynamic Island-style status widget driven by real event integrations
- [ ] Unified controls: Wi-Fi, Bluetooth, audio, mic, battery, display brightness
- [ ] Notifications, Do Not Disturb, privacy indicators and action handling
- [ ] Settings and first-run onboarding
- [ ] Notes app with pin toggle and verified capture exclusion on supported sessions
- [ ] Glass and Light appearance profiles; accessibility/reduced-motion support
- [ ] Multi-monitor, scaling, keyboard accessibility and crash isolation tests

**Exit gate:** All core controls work and **Veloce Glass stays ≤ 1,024 MiB at idle** on supported reference hardware; no unacceptable GPU or battery regression.

### Stage 6 — Security, data protection and long-term maintenance

**Goal:** Turn a functional desktop into an operating system that is safe to rely on.

**Deliverables**
- [ ] LUKS2 full-disk encryption pathway and secure recovery instructions
- [ ] Firewall, package signatures, trusted repositories, permission prompts and secrets policy
- [ ] Security update mechanism, sensible update cadence and failure handling
- [ ] Backup/restore solution and documented filesystem recovery tests
- [ ] Secure Boot / signed boot support investigation and device matrix
- [ ] Dependency inventory, component-by-component license notices, GPL/copyleft compliance and proprietary-app redistribution review
- [ ] Privacy, logging, diagnostic export and vulnerability disclosure policy
- [ ] Crash, low-disk, offline and low-memory resilience tests

**Exit gate:** No critical security/data-loss blockers; recovery drill passes; known limitations and license obligations documented.

### Stage 7 — Image construction, installation and recovery

**Goal:** Package Veloce as an installable distribution instead of a set of manual tweaks.

**Deliverables**
- [ ] Version-pinned `live-build` configuration and package manifests
- [ ] Reproducible build scripts and checksums; compare rebuild outputs and document differences
- [ ] Live USB boot on UEFI target hardware; non-destructive trial mode
- [ ] Graphical installer (Calamares candidate), language/timezone/keyboard setup
- [ ] Encrypted install, EFI partitions, bootloader and supported dual-boot scenarios
- [ ] Recovery environment, logs, backup/restore and installation error handling
- [ ] Installer QA for empty disk, pre-existing Linux and Windows coexistence

**Exit gate:** Bootable ISO produced with verified integrity; install/reboot/recovery succeeds on reference test device. **Never auto-format a user's existing Windows drive.**

### Stage 8 — Real-laptop alpha and regression testing

**Goal:** Validate Veloce as a serious replacement for normal Windows work.

**Deliverables**
- [ ] Hardware test matrix populated with logs for reference laptops
- [ ] Extended suspend/resume, Wi-Fi reconnect, battery and thermals tests
- [ ] Repeatable Zoom meetings, Firefox/Chrome/Edge/Brave sessions, VLC, web messaging, web office, optional LibreOffice, SQL/Python/VS Code work
- [ ] External displays, USB peripherals, headset and audio switching tests
- [ ] Update, rollback/recovery and power-interruption tests
- [ ] Performance trend report and blockers list
- [ ] User experience review and accessibility improvements

**Exit gate:** No critical regression or data-loss defects; performance target met; Windows remains available until confirmed safe to replace.

### Stage 9 — Beta release, documentation and maintenance

**Goal:** Ship a clearly labeled beta only after safety and compatibility gates pass.

**Deliverables**
- [ ] Versioned beta ISO, checksums, release notes and supported-hardware list
- [ ] Installation, dual boot, app setup and troubleshooting documentation
- [ ] Public issue/bug-report templates and reproducible diagnostics instructions
- [ ] Third-party attribution, corresponding-source/copyleft compliance, public repository license review and security guidance
- [ ] Updates/testing policy, release cadence and upgrade compatibility plan
- [ ] Distribution name clearance before publishing branding or installers widely

**Exit gate:** Transparent beta release with reproducible artifacts, documented hardware coverage and known limitations. **General availability is a separate decision.**

### Stage 10 — Stable-release candidate and ongoing operations

**Goal:** Earn the designation of a dependable Windows replacement.

**Deliverables**
- [ ] Stable upgrade and disaster-recovery strategy
- [ ] Proven security patch path and third-party app update testing
- [ ] Multi-device regression suite and supported laptop certification criteria
- [ ] Final user documentation, accessibility and localization plans
- [ ] Stable release readiness review, long-term issue ownership and support boundaries

**Exit gate:** Full baseline scope works on the explicitly supported hardware list, without violating the memory/security/reliability requirements.

## 8. Milestone 1: the 1 GiB desktop

This is the **first build we should implement** — **not** a custom glass desktop yet.

- [ ] M1-01: Create repository structure and capture hardware inventory.
- [ ] M1-02: Install Debian 13 minimally in a VM or spare/test drive.
- [ ] M1-03: Build LXQt + Wayfire and LXQt + Labwc candidate sessions.
- [ ] M1-04: Configure an Xorg session with reliable login/logout.
- [ ] M1-05: Configure Wi-Fi/networking, Bluetooth, PipeWire audio, notification and power services.
- [ ] M1-06: Run repeatable idle RAM/CPU benchmarks on all sessions.
- [ ] M1-07: Validate Firefox, optional Chrome/Edge/Brave, VLC, and Telegram/WhatsApp Web in browsers.
- [ ] M1-08: Run the [meeting matrix](docs/compatibility.md#meeting-tests) for native Zoom and web meeting clients. Verify window and display sharing in native Zoom on Xorg. Test Firefox ESR and Chrome for tab, window and display sharing in the web clients. Record Wayland results per candidate and prototype the [protected Notes surface](docs/adr/0002-desktop-sessions.md#protected-notes-surface) using dummy content.
- [ ] M1-09: Test Google Workspace, Microsoft 365 web, optional LibreOffice, VS Code, Python venv and a SQL client.
- [ ] M1-10: Record failures, choose architecture and define a fix backlog.

**Definition of done:** A bootable prototype with a reproducible configuration, working essential applications, documented hardware limitations and a measured idle memory result. A passing result requires **≤ 1,024 MiB**, not an assumption.

## 9. Proposed repository structure

```text
veloce_os/
├── README.md
├── .gitignore
├── LICENSE                         # Draft: restricts Veloce-owned materials only
├── CONTRIBUTING.md                 # Planned
├── SECURITY.md                     # Planned
├── docs/
│   ├── architecture.md
│   ├── requirements.md
│   ├── adr/                         # Architecture decision records
│   ├── performance-budget.md
│   ├── benchmarks.md
│   ├── hardware-test-matrix.md
│   ├── compatibility.md
│   ├── installation.md
│   ├── troubleshooting.md
│   └── releases.md
├── desktop/
│   ├── dock/
│   ├── island/
│   ├── control-center/
│   ├── notifications/
│   ├── settings/
│   ├── welcome/
│   └── themes/
├── system/
│   ├── packages/
│   ├── sessions/
│   ├── networking/
│   ├── audio/
│   ├── power/
│   ├── security/
│   ├── storage/
│   └── defaults/
├── applications/
│   ├── browsers/
│   ├── web-messaging/
│   ├── office/
│   ├── media/
│   ├── developer-tools/
│   └── web-apps/
├── image/
│   ├── live-build/
│   ├── installer/
│   └── boot/
├── scripts/
│   ├── bootstrap/
│   ├── benchmarks/
│   ├── diagnostics/
│   └── release/
├── tests/
│   ├── hardware/
│   ├── applications/
│   ├── performance/
│   ├── security/
│   ├── installation/
│   └── regression/
└── .github/
    ├── ISSUE_TEMPLATE/
    └── workflows/
```

*This tree is the target repository design. It does **not** mean these directories or implementations already exist.*

## 10. Development workflow and quality bar

1. **Specify:** Link each issue/feature to a requirement ID and acceptance test.
2. **Implement:** Prefer upstream packages/configuration; add custom Qt/QML code only where needed.
3. **Review:** Check security, service overhead, desktop failure isolation and accessibility.
4. **Test:** Validate app and hardware behavior; automate what can run in CI, retain real-device checks.
5. **Benchmark:** Record idle memory and power results for every desktop integration milestone.
6. **Package:** Build versioned packages/images with manifests, checksums and clear provenance.
7. **Release:** Publish only after stage gates pass; document untested hardware and third-party app limitations.

**Proposed CI checks:** shell lint, QML/Qt checks, config validation, package integrity, unit tests, security/dependency scanning, ISO build smoke test and VM boot test. Physical Wi-Fi, Bluetooth, battery, Zoom, audio jack, display and USB testing remains outside ordinary hosted GitHub CI.

**Branch policy (proposed):** `main` maintained by the project owner/authorized maintainers; changes land only with owner approval. Public visibility does not give others write access. Feature branches and reviewed PRs may be used internally; third-party changes are not accepted without explicit agreement. Tagged milestone releases (`v0.1.0-alpha`, etc.) only when justified by a working artifact.

## 11. Release risks and mitigation

| Risk | Impact | Mitigation |
|---|---|---|
| Veloce name may conflict with an existing software product | Branding/public distribution risk | Maintain as codename until reviewed; rename before public release if needed |
| Glass compositor exceeds 1 GiB target | Violates primary goal | Benchmark Wayfire vs Labwc; simplify effects; optimize QML; keep Light profile |
| Zoom functionality differs across Wayland/Xorg | Meeting workflow failure | Ship documented X11 compatibility session; real meeting test gate |
| Telegram/WhatsApp are browser-only in v1 | Browser-dependent calls and notifications | Test current web services; optional browser shortcuts; revisit native clients after v1 |
| Vendor-specific laptop drivers/firmware | Wi-Fi/audio/sleep failure | Maintain hardware compatibility list and test real devices before making claims |
| External packages introduce update/security issues | Reliability/security impact | Use verified repositories and explicit upgrade testing |
| Microsoft Office desktop not supported on Linux | Advanced spreadsheet/Word feature gap | Microsoft 365 web + optional LibreOffice; document advanced-file limitations |
| Restrictive original-code license vs GPL/third-party packages | Legal/release compliance risk | Restrict only original Veloce assets; honor upstream licenses, source obligations and vendor distribution terms |
| Dual boot/installer errors | Data-loss risk | Backups, live USB and non-destructive tests; never auto-partition existing user drives |
| Idle benchmark gamed by turning off critical services | Misleading result | Lock required service checklist and record session state for every sample |
| Too much custom code | Maintenance burden | Integrate mature upstream components; modular interfaces and small custom surface |
| Secure Boot/graphics compatibility | Install failure | Test supported boot paths; document restrictions rather than silently disabling defenses |

## 12. Scope and exclusions

### Included in baseline scope

- Fully working baseline desktop and essential laptop hardware services.
- Firefox, Chrome, Edge and Brave support; browser-only Telegram and WhatsApp; VLC; Zoom, VS Code, Python and SQL workflows.
- Google Workspace and Microsoft 365 web; optional LibreOffice; file manager, terminal, screenshots, clipboard, documents, USB/external drives and Google Drive access.
- Bluetooth, Wi-Fi, audio jack, webcam/microphone, brightness, volume, notifications, touchpad, function keys, battery/suspend and external displays.
- Security, updates, encryption, backups, diagnostics, graphical installation and recovery.
- Mac-inspired original glass interface, floating dock, Dynamic Island–style controls and Light performance profile.
- Notes with optional always-on-top placement and capture exclusion on verified sessions.
- Reproducible builds, performance tests and published supported-hardware limitations.

### Not planned for v1 unless evidence changes priorities

- Sharing all windows of an application as a group or sharing one tab inside a desktop application.
- Writing a kernel, Wi-Fi or GPU drivers from scratch.
- Running Windows-only `.exe` applications natively without compatibility layers.
- Native desktop Microsoft Office parity, especially advanced Excel/VBA/Power Query features.
- Universal compatibility with every laptop, especially unsupported proprietary hardware.
- Any native Telegram or WhatsApp application in v1, or an unofficial/impersonated WhatsApp client.
- Promising identical macOS visuals, behavior or access to Apple's proprietary services.
- Replacing upstream security, audio, networking, boot or storage services with untested custom implementations.

## 13. Helpful upstream references

- [Debian 13 (Trixie) release and support](https://www.debian.org/releases/trixie/)
- [LXQt Wayland session and supported compositors](https://lxqt-project.org/wiki/Wayland-Session)
- [Debian `live-build`](https://packages.debian.org/trixie/live-build)
- [Official Firefox Linux install documentation](https://support.mozilla.org/en-US/kb/install-firefox-linux)
- [Google Chrome for Linux](https://support.google.com/chrome/answer/95346)
- [Microsoft Edge for Linux](https://www.microsoft.com/edge/download)
- [Brave Linux installation](https://brave.com/linux/)
- [VideoLAN VLC for Debian](https://www.videolan.org/vlc/download-debian.html)
- [Telegram Web](https://web.telegram.org/)
- [WhatsApp Web](https://web.whatsapp.com/)
- [Google Docs/Sheets/Slides browser requirements](https://support.google.com/docs/answer/2375082)
- [Microsoft 365 for the web](https://support.microsoft.com/en-us/office/get-started-with-microsoft-365-for-the-web-in-microsoft-365)
- [LibreOffice](https://www.libreoffice.org/download/)
- [Zoom screen-sharing documentation](https://support.zoom.com/hc/en/article?id=zm_kb&sysparm_article=KB0060596)
- [Siemens Veloce platform — naming review reference](https://www.siemens.com/en-us/products/ic/hav/veloce/)
- [Siemens document using “Veloce OS” — naming review reference](https://static.sw.cdn.siemens.com/siemens-disw-assets/public/6TeATy2I6BlXDr3txgEWaK/en-US/Siemens-SW_Veloce-Primo-Enterprise-prototyping-solution_DS-83490-rev3.pdf)

## 14. Licensing and public repository policy

**Veloce OS is intended to be free to install and use, but its original Veloce-authored material is not being released under an open-source license.** The custom `LICENSE` in this repository is a **draft source-available / no-unauthorized-modification-and-commercialization license** and should be reviewed by a qualified lawyer before a public ISO or installer release.

- Anyone may view the public repository (and use GitHub's normal view/fork features under GitHub's terms). Access to the repo does **not** authorize public distribution of modified Veloce-branded products.
- Users may install and run **official, unmodified Veloce distributions** at no charge, including as a workstation for professional work. This does not confer a right to sell, repackage, monetize, distribute unofficial releases, or commercially exploit Veloce-original code, designs or branding.
- Only the owner or authorized maintainers may publish **official Veloce OS releases and updates**. This does **not** stop users from installing security updates, applications or changing their own settings.
- **Third-party components retain their own licenses.** In particular, GPL-licensed Linux/Debian components may carry rights to modify, redistribute or sell them; Veloce cannot withdraw those rights. GPL source and notice obligations must be satisfied when distributing relevant binaries. Vendor browsers/Zoom may have separate redistribution restrictions.
- The custom license covers only copyrightable Veloce-original materials for which the project owner actually holds the rights. It does not supersede license conditions for Debian, Linux, LXQt, VLC, LibreOffice, browsers, media codecs or other third-party programs.
- Before distributing an ISO, produce `THIRD_PARTY_NOTICES.md` (or equivalent) listing bundled components, exact licenses, source links/corresponding-source offers where required, and redistribution permissions. Branding/trademark clearance remains open.

**Important distinction:** This is **source-available**, not an OSI-approved open-source project. A restrictive project license cannot make the *entire Debian-derived OS* non-modifiable or prohibit permitted GPL redistribution. Protect original code and branding within the scope of applicable law and upstream license obligations.

## 15. Current project state

| Area | Status |
|---|---|
| Vision / requirements | **Drafted** |
| Architecture | **Proposed — not validated** |
| Brand clearance | **Open — resolve before branded ISO distribution** |
| GitHub repository | **Public `veloce_os` repository created** |
| Stage 0 | **Complete. Prototype scope and test strategy reviewed. HW-01 selected.** |
| Original-code LICENSE | **Draft available; counsel review required before public binary release** |
| Third-party attribution audit | **Not started** |
| Prototype implementation | **Stage 1 in progress. VM setup guide and Xorg package manifest available. Session testing pending.** |
| Reference hardware testing | **Not started** |
| 1 GiB benchmark result | **Not yet measured** |
| Installer / live ISO | **Not built** |

---

**Veloce OS — light on resources, complete where it matters.**

*This README is a technical plan and working product specification. It will be revised as real builds and benchmark evidence replace assumptions.*
