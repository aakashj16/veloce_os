# Application compatibility

All scenarios are planned. Record the build, hardware, app version, session, expected and observed results, and any known limitations.

| Workflow | Minimum validation |
|---|---|
| Browsers | Firefox ESR ships by default. Install Chrome, Edge and Brave on demand through their official paths. Verify update, downloads, media, permissions, and webcam/microphone access for each. Firefox ESR and Chrome both run the full meeting matrix. Record screen-sharing results separately for Edge and Brave. |
| Telegram and WhatsApp | Use the official web apps. On each supported browser, verify sign-in or device linking, messaging, attachments, notifications with permission and session persistence. Record account-dependent call behavior separately. |
| VLC | Open a local MP4 and MP3, toggle subtitles and fullscreen, and open a media file from removable storage. |
| Google Workspace and Microsoft 365 web | Sign in, edit, save, share and download or export a sample document, sheet and presentation. |
| LibreOffice | In the optional offline profile, open and export sample DOCX, XLSX and PPTX files. Record any formatting or feature differences. |
| VS Code, Python and SQL | Open a Git repository in VS Code, use its terminal, run a script in a Python virtual environment, open a notebook, and run a query against a test database with the selected SQL client. |
| Everyday tools and optional services | Verify PDF, archives, printing, screenshots, clipboard, Drive web and SSH. In optional profiles, test any enabled Drive mount, container service or local database and record its startup behavior. |

## Meeting tests

Run a real test meeting with presenter permissions and a receiver on a second device. Record results on Xorg/Openbox, Labwc and Wayfire. Start with the exact Firefox ESR build Veloce ships. All cases start as **Not run**.

| Client | One browser tab | One application window | Selected display |
|---|---|---|---|
| Google Meet in Firefox ESR | Required | Required | Required |
| Zoom Web in Firefox ESR | Required | Required | Required |
| Google Meet in Chrome | Required | Required | Required |
| Installed Google Meet web app in Chrome | Required | Required | Required |
| Zoom Web in Chrome | Required | Required | Required |
| Native Zoom on Xorg | Use a web client for a tab picker | Required | Required |
| Native Zoom on either Wayland candidate | Use a web client for a tab picker | Vendor documents Xorg for this workflow | Verify and record |

Xorg is the first session to validate. A Wayland default must pass the same web-client requirements under MEET-003. Changing sessions can affect which windows are offered. Firefox ESR's tab picker and the meeting service also need to support the tab workflow.

1. **Browser tab** — Share a demo BigQuery tab. Switch locally to a second tab and have the receiver confirm that the original selected tab is the only content visible. Run this test independently in Firefox ESR and Chrome. Moving the demo into its own window is a window-sharing workaround.
2. **Application window** — Open two VS Code windows with different dummy content. Share one and have the receiver confirm that the other window stays private. Verify that switching editor tabs inside the selected window follows the displayed window.
3. **Selected display** — Connect a second monitor. Share one display and verify the second display stays private.
4. **Capture controls** — Check cancel, start and stop. Verify that cancelling leaves no capture active and that the receiver stops seeing the stream when sharing ends.
5. **Meeting basics** — Check join and leave, mic, camera, output selection and headset switching. Record shared tab or desktop audio separately from microphone audio.
6. **Zoom installation** — On a clean baseline install the official amd64 Debian package on demand. Verify app launch, sign-in or meeting join, and the documented update path.

Record the presenter and receiver result per case. Capture the session type, compositor and portal version, browser/app version, monitor layout and a brief symptom or sanitized log for any failure. If host policy blocks sharing, rerun with a test meeting that permits it. If the tested client or session lacks a required capture mode, log the affected mode as **Blocked** and identify the layer involved. A first prototype can still measure and document the gap.

Google documents a Chrome tab choice for Meet. Zoom's Linux desktop instructions describe individual application windows and a desktop. Verify true Firefox ESR tab capture in the shipped build before promising it.

- [Meet presentation modes](https://support.google.com/meet/answer/9308856)
- [Meet installable web app](https://support.google.com/meet/answer/10708569)
- [Zoom Linux installation](https://support.zoom.com/hc/en/article?id=zm_kb&sysparm_article=KB0063458)
- [Native and web Zoom sharing](https://support.zoom.com/hc/en/article?id=zm_kb&sysparm_article=KB0060596)
- [Web screen capture source types](https://www.w3.org/TR/screen-capture/#display-surfaces)

## Notes privacy tests

Run these tests with dummy notes and a receiver on a second device. Certify Notes on the v1 default session. Test each meeting client and capture path offered on any session where Notes is available, including alternate capture paths that the client can invoke.

1. **Pin toggle** — Pin Notes and focus another app. Notes stays above that app locally. Unpin Notes and verify it follows normal window stacking. The same capture protection applies in both states.
2. **Source picker** — Verify that Notes cannot be selected or previewed as a share source. Open a second capture picker during a share and check it too.
3. **Full display** — Start sharing the same display where Notes is visibly over the demo. The receiver should see the demo underneath with no Notes text, title, or window pixels. Test both pin states, opening and moving Notes during sharing, and moving it between displays.
4. **Window and tab** — Share a window and a browser tab while Notes overlaps them locally. Check that Notes stays absent from the received stream.
5. **Transitions and failure** — Inspect received frames at the start, stop, and restart of sharing, and while the pin state or monitor configuration changes. Simulate loss of capture protection and verify that Notes cannot appear in an unfiltered frame. On an uncertified session, verify that Notes cannot display its contents.

Record separate pass/fail results for local visibility and receiver privacy. An unexpected Notes pixel is a failure. Verify after compositor, portal, or meeting client changes that affect capture.
