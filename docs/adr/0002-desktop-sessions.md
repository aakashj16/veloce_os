# ADR 0002 · Wayland candidates and Xorg session

**Status** — Evaluation plan accepted. Xorg/Openbox favored as the provisional Stage 1 baseline (2026-11-10); final production default pending Stage 4/8 evidence.

Initial Stage 1 functional testing found LXQt + Xorg/Openbox simpler and more dependable than the evaluated Labwc and Wayfire sessions. That observation is not a certification of performance or universal stability. Use Xorg/Openbox as the provisional desktop baseline. Continue comparing LXQt with Labwc and Wayfire using the same Debian base, services, hardware and measurement protocol. Record the compositor, portal backend, PipeWire, browser and native Zoom versions. Compare memory, CPU, GPU effects, battery, suspend/resume, scaling, accessibility, crash recovery and meeting capture before choosing the default.

Provide LXQt on Xorg with Openbox as a selectable login session for native Zoom window sharing, testing it as a complete desktop session. Standard and Enhanced are **performance modes**, not compositors or separate display stacks. Enhanced does not require Wayfire; design is matte-first with optional refined effects. See [ADR 0003](0003-visual-identity-and-performance-modes.md). XWayland is available for compatible X11 apps within the Wayland candidates.

The stock Debian 13 wlr screen cast backend selects a display output. A proposed Wayland default must pass the required Firefox ESR and Chrome meeting tests and the Notes privacy checks. Xorg is the provisional prototype default while those checks are open; no change to the protected Notes security gate is implied. Record browser or client limitations separately and verify them on Xorg before attributing them to Wayland. Verify native Zoom against the vendor's documented session capabilities.

## Protected Notes surface

Prototype compositor-owned exclusion on Wayland. The compositor should render Notes for the local display and omit its surface and decorations when composing frames for capture. Omit Notes from source lists and previews. The app's pin toggle changes only local stacking. Establish protection before rendering Notes and cover every capture route available to supported meeting clients in that session.

Treat unverified sessions as unavailable for Notes, including stock Xorg/Openbox. Evaluate an enforceable protected path separately if Notes is needed in the Xorg compatibility session. Showing a warning or trying to hide Notes after sharing starts leaves a risk of exposing the first frame. The protection must hold during session and capture transitions.

- [Debian 13 portal backend manual](https://manpages.debian.org/trixie/xdg-desktop-portal-wlr/xdg-desktop-portal-wlr.5.en.html)
- [Zoom sharing documentation](https://support.zoom.com/hc/en/article?id=zm_kb&sysparm_article=KB0060596)
- [XDG ScreenCast portal interface](https://flatpak.github.io/xdg-desktop-portal/docs/doc-org.freedesktop.portal.ScreenCast.html)
- [Meeting test cases](../compatibility.md#meeting-tests)
- [Notes privacy checks](../compatibility.md#notes-privacy-tests)
