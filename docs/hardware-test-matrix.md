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
| Baseline OS, test image and kernel | 64-bit Windows 11 work laptop. Windows build, Debian test image and kernel pending. |

The inventory records Windows observations. Confirm the active drivers and unresolved device IDs from the Debian live session. UEFI HTTP Boot support remains unconfirmed for this SKU. In a Debian live session, `lspci -nnk` and `lsusb` help identify the chipsets and loaded drivers. See the [Stage 2 guide](stage-2.md) for the Windows inventory steps, test media and first native checks. Use a disposable target for installation and recovery tests.

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
