# Reference hardware and test evidence

## First reference laptop

| Field | Inventory |
|---|---|
| Reference ID | HW-01 |
| Make and model | [HP Laptop 15s-dr2007tx](https://support.hp.com/in-en/product/troubleshooting/hp-15s-dr2000-laptop-pc-series/model/38462190?sku=191F4PA), product SKU `191F4PA#ACJ`. Windows System Model `HP Laptop 15s-dr2xxx`. |
| CPU and architecture | Intel Core i5-1035G1, x86-64 (amd64) |
| Installed RAM | 16 GB |
| Integrated and discrete GPU | Intel UHD Graphics and NVIDIA GeForce MX130 (2 GB) |
| Wi-Fi | Intel Wi-Fi 6 AX201 160 MHz |
| Ethernet | Realtek PCIe GbE Family Controller. Exact controller ID pending. |
| Bluetooth | Intel Wireless Bluetooth. Exact radio ID pending. |
| Audio | Intel Smart Sound Technology (SST), Realtek Audio and Intel Display Audio. Exact Realtek codec pending. |
| Storage type and available test drive | Windows reports a 238.5 GiB NVMe SSD and a 931.5 GiB HDD with bus type shown as RAID. Disposable test drive pending. |
| Firmware version, UEFI and Secure Boot state | Insyde F.68 (2024-10-14), UEFI, Secure Boot On. |
| Internal display, scaling and external monitor connection | 1920 × 1080 at 125% scaling in Windows. No pen or touch input reported. External-monitor connection pending. |
| Webcam, wired headset and Bluetooth test devices | HP TrueVision HD Camera. Bluetooth earbuds, mouse, keyboard and speaker are listed in Windows. Wired headset availability pending. |
| Baseline OS, test image and kernel | Windows 11 was the original inventory baseline; **Debian 13 KDE Plasma/Wayland now runs on personally owned HW-01 (ARIA)**. Veloce image/commit and kernel benchmark metadata remain pending. |

The inventory combines historical Windows observations with [ARIA’s Debian 13 field report](field-reports/aria-debian13-2026-11-17.md). The owner reported that all applicable physical checks performed passed at Stage 2 closeout, with graphics validation pending. **November 23–24 follow-up:** the MX130 `nouveau` boot hang now has a [documented Intel-only recovery workaround](troubleshooting.md); this is not a graphics validation pass. Individual driver logs and release acceptance evidence remain to be collected where needed.

## Physical test checklist

| Area | Checks | HW-01 |
|---|---|---|
| Graphics and display | Intel UHD, NVIDIA MX130, hybrid graphics and external-monitor output | **Partial:** Intel `i915` works; MX130 `nouveau` boot hangs on 6.12.118; [documented GRUB blacklist workaround](troubleshooting.md) works. NVIDIA/hybrid and external monitor remain unvalidated. |
| Network and Bluetooth | Firmware and driver load, Wi-Fi connect/reconnect, Ethernet or adapter, headset pairing and profile switching | Pass (owner-reported for checks performed) |
| Audio and video | Speakers, internal and USB mic/camera, wired jack detection, headset routing and camera privacy controls | Pass (owner-reported for checks performed) |
| Power and input | Brightness and volume keys, touchpad, battery, lid close, repeated sleep/wake and thermal behavior | Pass (owner-reported for checks performed) |
| Storage and peripherals | USB, safe eject, NTFS/exFAT/ext4 access, printer and network shares as applicable | Pass (owner-reported for checks performed) |
| Boot and recovery | Live UEFI boot, Secure Boot behavior, encrypted install, update/reboot and restore on disposable data | Release recovery pending |
| Meetings | Run MEET-001, MEET-002 and NOTES-001 with a second device and an external display. Record browser and session results. | Native Zoom window and display sharing passed on ARIA. Browser, multi-monitor and Notes checks remain open. |
| Performance | Reboot and collect all five idle samples per session with baseline services enabled | Not run |

For each run record the date, reference ID, commit/image hash, kernel, session, versions, expected result, observed result and evidence. Use **Pass**, **Fail**, **Blocked**, **Not applicable** or **Not run**. For blocked or inapplicable checks, record the reason. Before publishing evidence, remove serial numbers, usernames, network identifiers and credentials.

See the [complete hardware checklist](../README.md#5-complete-hardware-and-operating-system-test-matrix) for the full device and OS coverage.
