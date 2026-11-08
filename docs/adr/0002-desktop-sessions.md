# ADR 0002 · Wayland candidates and Xorg session

**Status** — Proposed for Stage 0 review

Compare LXQt with Labwc and Wayfire using the same Debian base, services, hardware and measurement protocol. Record the compositor, portal backend, PipeWire, browser and native Zoom versions. Compare memory, CPU, GPU effects, battery, suspend/resume, scaling, accessibility, crash recovery and meeting capture before choosing the default.

Provide LXQt on Xorg with Openbox as a selectable login session for native Zoom window sharing. Test it as a complete desktop session. XWayland is available for compatible X11 apps within the Wayland candidates.

The stock Debian 13 wlr screen cast backend selects a display output. A candidate that cannot present an individual window in the web meeting clients fails the proposed MEET-003 gate. If both Wayland candidates miss that gate, use Xorg as the initial default and keep Wayland under evaluation. Verify native Zoom separately against the vendor's documented Xorg and Wayland capabilities.

- [Debian 13 portal backend manual](https://manpages.debian.org/trixie/xdg-desktop-portal-wlr/xdg-desktop-portal-wlr.5.en.html)
- [Zoom sharing documentation](https://support.zoom.com/hc/en/article?id=zm_kb&sysparm_article=KB0060596)
- [Meeting test cases](../compatibility.md#meeting-tests)
