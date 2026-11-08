# Application compatibility

All scenarios are planned. Record the build, hardware, app version, session, expected and observed results, and any known limitations.

| Workflow | Minimum validation |
|---|---|
| Browsers | Firefox ESR ships by default. Install Chrome, Edge and Brave on demand through their official paths. Verify update, downloads, media, permissions, and webcam/microphone access for each. Chrome is the reference for the full meeting matrix. Record screen-sharing results separately for Firefox ESR, Edge and Brave. |
| Telegram and WhatsApp | Use the official web apps. On each supported browser, verify sign-in or device linking, messaging, attachments, notifications with permission and session persistence. Record account-dependent call behavior separately. |
| VLC | Open a local MP4 and MP3, toggle subtitles and fullscreen, and open a media file from removable storage. |
| Google Workspace and Microsoft 365 web | Sign in, edit, save, share and download or export a sample document, sheet and presentation. |
| LibreOffice | In the optional offline profile, open and export sample DOCX, XLSX and PPTX files. Record any formatting or feature differences. |
| VS Code, Python and SQL | Open a Git repository in VS Code, use its terminal, run a script in a Python virtual environment, open a notebook, and run a query against a test database with the selected SQL client. |
| Everyday tools and optional services | Verify PDF, archives, printing, screenshots, clipboard, Drive web and SSH. In optional profiles, test any enabled Drive mount, container service or local database and record its startup behavior. |

## Meeting tests

Run a real test meeting with presenter permissions and a receiver on a second device. Test Xorg/Openbox, Labwc and Wayfire separately. The entries below specify what to verify, with all results initially **Not run**.

| Client | Tab | Application window | Selected display |
|---|---|---|---|
| Google Meet in Chrome | Required on the certified session | Required on the certified session | Required on the certified session |
| Google Meet installed Chrome web app | Required on the certified session | Required on the certified session | Required on the certified session |
| Zoom Web in Chrome | Required on the certified session | Required on the certified session | Required on the certified session |
| Native Zoom on Xorg | Use the browser workflow for a tab picker | Required | Required |
| Native Zoom on either Wayland candidate | Use the browser workflow for a tab picker | Vendor documents Xorg for this workflow | Verify and record |

For MEET-002, Xorg is the first path to certify. A proposed Wayland default must pass the same browser matrix under MEET-003.

1. **Tab** — Share a demo BigQuery tab. Switch locally to a second tab and have the receiver confirm that only the selected tab is visible.
2. **Application window** — Open two VS Code windows with different dummy content. Share one and have the receiver confirm that the other window stays private.
3. **Display** — Connect a second monitor. Share one display and verify the second display stays private.
4. **Capture controls** — Check cancel, start and stop. Verify that cancelling leaves no capture active and that the receiver stops seeing the stream when sharing ends.
5. **Meeting basics** — Check join and leave, mic, camera, output selection and headset switching. Record shared tab or desktop audio separately from microphone audio.
6. **Zoom installation** — On a clean baseline install the official amd64 Debian package on demand. Verify app launch, sign-in or meeting join, and the documented update path.

Record the presenter and receiver result per case. Capture the session type, compositor and portal version, browser/app version, monitor layout and a brief symptom or sanitized log for any failure. If a host policy prevents joining in the browser or sharing, record **Blocked** and rerun with a test meeting that permits the workflow.

- [Meet presentation modes](https://support.google.com/meet/answer/9308856)
- [Meet installable web app](https://support.google.com/meet/answer/10708569)
- [Zoom Linux installation](https://support.zoom.com/hc/en/article?id=zm_kb&sysparm_article=KB0063458)
- [Native and web Zoom sharing](https://support.zoom.com/hc/en/article?id=zm_kb&sysparm_article=KB0060596)
