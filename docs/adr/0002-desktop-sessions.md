# ADR 0002 · Wayland candidates and Xorg session

**Status** — Proposed for Stage 0 review

Compare LXQt with Labwc and Wayfire using the same Debian base, services, hardware and measurement protocol. Record the compositor, portal backend, PipeWire, browser and native Zoom versions. Compare memory, CPU, GPU effects, battery, suspend/resume, scaling, accessibility, crash recovery and meeting capture before choosing the default.

Provide LXQt on Xorg with Openbox as a selectable login session for native Zoom window sharing. Test it as a complete desktop session. XWayland is available for compatible X11 apps within the Wayland candidates.

The stock Debian 13 wlr screen cast backend selects a display output. A proposed Wayland default must pass the required Firefox ESR and Chrome meeting tests, including individual windows and whole applications. If both candidates miss the gate, use Xorg as the provisional default. Record any browser or client limitation separately and verify it on Xorg before attributing it to the Wayland stack. Verify native Zoom separately against the vendor's documented Xorg and Wayland capabilities.

- [Debian 13 portal backend manual](https://manpages.debian.org/trixie/xdg-desktop-portal-wlr/xdg-desktop-portal-wlr.5.en.html)
- [Zoom sharing documentation](https://support.zoom.com/hc/en/article?id=zm_kb&sysparm_article=KB0060596)
- [Meeting test cases](../compatibility.md#meeting-tests)
