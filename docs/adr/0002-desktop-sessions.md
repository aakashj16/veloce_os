# ADR 0002 · KDE Plasma on Wayland

**Status** — Accepted on 2026-11-18.

Build Veloce on Debian 13 with KDE Plasma and KWin on Wayland. Native Zoom Workplace 7.2.5 passed individual application-window and entire-display sharing on ARIA, the physical HW-01 reference laptop. The owner subsequently confirmed the physical checks passed with graphics validation still open. See the [ARIA field report](../field-reports/aria-debian13-2026-11-17.md).

The [Stage 1 VM trials](../stage-1.md) preserve the LXQt/Openbox, Labwc and Wayfire results. Validate Veloce's KDE configuration with the [meeting matrix](../compatibility.md#meeting-tests) and run the reference-hardware benchmark with normal services enabled. Standard and Enhanced both use the KDE baseline.

# ADR 0002 · KDE Plasma on Wayland

**Status** — Accepted on 2026-11-18.

Build Veloce on Debian 13 with KDE Plasma and KWin on Wayland. Native Zoom Workplace 7.2.5 passed individual application-window and entire-display sharing on ARIA, the physical HW-01 reference laptop. The owner subsequently confirmed the physical checks passed with graphics validation still open. See the [ARIA field report](../field-reports/aria-debian13-2026-11-17.md).

The [Stage 1 VM trials](../stage-1.md) preserve the LXQt/Openbox, Labwc and Wayfire results. Validate Veloce's KDE configuration with the [meeting matrix](../compatibility.md#meeting-tests) and run the reference-hardware benchmark with normal services enabled. Standard and Enhanced both use the KDE baseline.

## Protected Notes surface

Prototype compositor-owned exclusion on Wayland. The compositor should render Notes for the local display and omit its surface and decorations when composing frames for capture. Omit Notes from source lists and previews. The app's pin toggle changes only local stacking. Establish protection before rendering Notes and cover every capture route available to supported meeting clients in that session.

Treat unverified sessions as unavailable for Notes, including stock Xorg/Openbox. Evaluate an enforceable protected path separately if Notes is needed in the Xorg compatibility session. Showing a warning or trying to hide Notes after sharing starts leaves a risk of exposing the first frame. The protection must hold during session and capture transitions.

- [Debian 13 portal backend manual](https://manpages.debian.org/trixie/xdg-desktop-portal-wlr/xdg-desktop-portal-wlr.5.en.html)
- [Zoom sharing documentation](https://support.zoom.com/hc/en/article?id=zm_kb&sysparm_article=KB0060596)
- [XDG ScreenCast portal interface](https://flatpak.github.io/xdg-desktop-portal/docs/doc-org.freedesktop.portal.ScreenCast.html)
- [Meeting test cases](../compatibility.md#meeting-tests)
- [Notes privacy checks](../compatibility.md#notes-privacy-tests)
