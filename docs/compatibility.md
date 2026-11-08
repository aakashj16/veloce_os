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

| Client | One browser tab | One application window | Whole application | Selected display |
|---|---|---|---|---|
| Google Meet in Firefox ESR | Required | Required | Required | Required |
| Zoom Web in Firefox ESR | Required | Required | Required | Required |
| Google Meet in Chrome | Required | Required | Required | Required |
| Installed Google Meet web app in Chrome | Required | Required | Required | Required |
| Zoom Web in Chrome | Required | Required | Required | Required |
| Native Zoom on Xorg | Use a web client for a tab picker | Required | Required | Required |
| Native Zoom on either Wayland candidate | Use a web client for a tab picker | Vendor documents Xorg for this workflow | Evaluate on Xorg first | Verify and record |

Xorg is the first session to validate. A Wayland default must pass the same web-client requirements under MEET-003. Changing sessions can affect which windows are offered. Firefox ESR's tab picker and the meeting service also need to support the tab workflow.

1. **Browser tab** — Share a demo BigQuery tab. Switch locally to a second tab and have the receiver confirm that the original selected tab is the only content visible. Run this test independently in Firefox ESR and Chrome. Moving the demo into its own window is a window-sharing workaround.
2. **Application window** — Open two VS Code windows with different dummy content. Share one and have the receiver confirm that the other window stays private. Verify that switching editor tabs inside the selected window follows the displayed window.
3. **Whole application** — Select VS Code once with two visible VS Code windows open. Have the receiver confirm that both windows can be presented as you switch between them, while an unrelated file manager stays private. Record whether newly opened VS Code windows are included and how the user can tell. A selected monitor or one VS Code window does not pass this case.
4. **Selected display** — Connect a second monitor. Share one display and verify the second display stays private.
5. **Optional desktop app tab** — If an app exposes supported capture of one editor or document tab, verify that switching to another tab locally does not change the receiver's view. Treat this separately from browser-tab and window capture.
6. **Capture controls** — Check cancel, start and stop. Verify that cancelling leaves no capture active and that the receiver stops seeing the stream when sharing ends.
7. **Meeting basics** — Check join and leave, mic, camera, output selection and headset switching. Record shared tab or desktop audio separately from microphone audio.
8. **Zoom installation** — On a clean baseline install the official amd64 Debian package on demand. Verify app launch, sign-in or meeting join, and the documented update path.

Record the presenter and receiver result per case. Capture the session type, compositor and portal version, browser/app version, monitor layout and a brief symptom or sanitized log for any failure. If host policy blocks sharing, rerun with a test meeting that permits it. If the tested client or session lacks a required capture mode, log the affected mode as **Blocked** and identify the layer involved. A first prototype can still measure and document the gap.

Google documents a Chrome tab choice for Meet. Zoom's Linux desktop instructions describe individual application windows and a desktop. Treat true Firefox ESR tab capture and application-wide capture as capabilities to verify before promising them.

- [Meet presentation modes](https://support.google.com/meet/answer/9308856)
- [Meet installable web app](https://support.google.com/meet/answer/10708569)
- [Zoom Linux installation](https://support.zoom.com/hc/en/article?id=zm_kb&sysparm_article=KB0063458)
- [Native and web Zoom sharing](https://support.zoom.com/hc/en/article?id=zm_kb&sysparm_article=KB0060596)
- [Web screen capture source types](https://www.w3.org/TR/screen-capture/#display-surfaces)
