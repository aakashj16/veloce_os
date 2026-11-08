# Reference hardware and test evidence

## First reference laptop

| Field | Inventory |
|---|---|
| Reference ID | HW-01 |
| Make and model | Pending selection |
| CPU and architecture | Pending |
| Installed RAM | Pending |
| Integrated and discrete GPU | Pending |
| Wi-Fi, Bluetooth and audio chipsets | Pending |
| Storage type and available test drive | Pending |
| Firmware version, UEFI and Secure Boot state | Pending |
| Internal display, scaling and external monitor connection | Pending |
| Webcam, wired headset and Bluetooth test devices | Pending |
| Baseline OS, test image and kernel | Pending |

For the first pass, the make and model, CPU, installed RAM and GPU are enough to identify a candidate. On Windows, `msinfo32` shows the system model, processor and memory. Task Manager's Performance tab shows the GPU. Record selected fields only. In a Debian live session, `lspci -nnk` and `lsusb` help identify the chipsets and loaded drivers. Use a VM or live USB for initial checks. Use a spare or disposable disk for installation and recovery tests.

## Physical test checklist

| Area | Checks | HW-01 |
|---|---|---|
| Graphics and display | Driver loads, native resolution, scaling, HDMI/USB-C display hotplug, per-monitor layout | Not run |
| Network and Bluetooth | Firmware and driver load, Wi-Fi connect/reconnect, Ethernet or adapter, headset pairing and profile switching | Not run |
| Audio and video | Speakers, internal and USB mic/camera, wired jack detection, headset routing and camera privacy controls | Not run |
| Power and input | Brightness and volume keys, touchpad, battery, lid close, repeated sleep/wake and thermal behavior | Not run |
| Storage and peripherals | USB, safe eject, NTFS/exFAT/ext4 access, printer and network shares as applicable | Not run |
| Boot and recovery | Live UEFI boot, Secure Boot behavior, encrypted install, update/reboot and restore on disposable data | Not run |
| Meetings | Run MEET-001 and MEET-002 with a second device and an external display | Not run |
| Performance | Reboot and collect all five idle samples per session with baseline services enabled | Not run |

For each run record the date, reference ID, commit/image hash, kernel, session, versions, expected result, observed result and evidence. Use **Pass**, **Fail**, **Blocked**, **Not applicable** or **Not run**. For blocked or inapplicable checks, record the reason. Before publishing evidence, remove serial numbers, usernames, network identifiers and credentials.

See the [complete hardware checklist](../README.md#5-complete-hardware-and-operating-system-test-matrix) for the full device and OS coverage.
