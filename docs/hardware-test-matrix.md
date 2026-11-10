# Reference hardware and test evidence

## First reference laptop

| Field | Inventory |
|---|---|
| Reference ID | HW-01 |
| Make and model | HP laptop selected. Exact system model pending. |
| CPU and architecture | Intel Core i5-1035G1, x86-64 (amd64) |
| Installed RAM | 16 GB |
| Integrated and discrete GPU | Intel UHD Graphics and NVIDIA GeForce MX130 (2 GB) |
| Wi-Fi, Bluetooth and audio chipsets | Pending |
| Storage type and available test drive | 1.14 TB total reported by Windows. Drive types, physical disk layout and disposable test drive pending. |
| Firmware version, UEFI and Secure Boot state | Pending |
| Internal display, scaling and external monitor connection | No pen or touch input reported. Resolution, scaling and external monitor connection pending. |
| Webcam, wired headset and Bluetooth test devices | Pending |
| Baseline OS, test image and kernel | 64-bit Windows 11 work laptop. Windows build, Debian test image and kernel pending. |

The first physical reference laptop is selected. Use `msinfo32` in Windows to record the exact System Model before publishing device-specific compatibility results. Record selected fields only. In a Debian live session, `lspci -nnk` and `lsusb` help identify the chipsets and loaded drivers. See the [Stage 2 guide](stage-2.md) for the Windows inventory steps, test media and first native checks. Use a disposable target for installation and recovery tests.

## Physical test checklist

| Area | Checks | HW-01 |
|---|---|---|
| Graphics and display | Driver loads, native resolution, scaling, HDMI/USB-C display hotplug, per-monitor layout | Not run |
| Network and Bluetooth | Firmware and driver load, Wi-Fi connect/reconnect, Ethernet or adapter, headset pairing and profile switching | Not run |
| Audio and video | Speakers, internal and USB mic/camera, wired jack detection, headset routing and camera privacy controls | Not run |
| Power and input | Brightness and volume keys, touchpad, battery, lid close, repeated sleep/wake and thermal behavior | Not run |
| Storage and peripherals | USB, safe eject, NTFS/exFAT/ext4 access, printer and network shares as applicable | Not run |
| Boot and recovery | Live UEFI boot, Secure Boot behavior, encrypted install, update/reboot and restore on disposable data | Not run |
| Meetings | Run MEET-001, MEET-002 and NOTES-001 with a second device and an external display. Record browser and session results. | Not run |
| Performance | Reboot and collect all five idle samples per session with baseline services enabled | Not run |

For each run record the date, reference ID, commit/image hash, kernel, session, versions, expected result, observed result and evidence. Use **Pass**, **Fail**, **Blocked**, **Not applicable** or **Not run**. For blocked or inapplicable checks, record the reason. Before publishing evidence, remove serial numbers, usernames, network identifiers and credentials.

See the [complete hardware checklist](../README.md#5-complete-hardware-and-operating-system-test-matrix) for the full device and OS coverage.
