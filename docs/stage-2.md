# Stage 2 · HW-01 hardware foundation

**Status** — In progress. The existing Windows inventory has been reviewed. The remaining device inventory and all native Linux checks are pending.

Use Debian 13 amd64 and **LXQt on Xorg with Openbox** as the Stage 2 baseline. [Draft PR #2](https://github.com/aakashj16/veloce_os/pull/2) records the Stage 1 VMware results. The 725 MiB Xorg reading is a VM reference. Record live-session measurements separately from installed-system acceptance. The production session decision is tracked in [ADR 0002](adr/0002-desktop-sessions.md).

## 1. Complete the Windows inventory

The [hardware matrix](hardware-test-matrix.md#first-reference-laptop) records an HP laptop with Windows 11, an Intel Core i5-1035G1, 16 GB RAM, Intel UHD Graphics and an NVIDIA GeForce MX130 with 2 GB VRAM. Windows reported 1.14 TB of total storage. This is a suitable amd64 reference candidate. Prioritize Wi-Fi and graphics, including which GPU drives the external display. The exact model, storage layout and device chipsets still need to be identified.

1. Press **Win + R**, enter `msinfo32`, and copy **System Model**, **BIOS Version/Date**, **BIOS Mode**, **Secure Boot State** and **Version** from System Summary. Copy only those fields.
2. Open regular **Windows PowerShell** and run these read-only queries. They show device names and storage types.

   ```powershell
   Get-CimInstance -ClassName Win32_PnPEntity -Filter "Present = TRUE" |
       Where-Object { $_.PNPClass -in @('Net', 'Bluetooth', 'MEDIA', 'Camera', 'Image') } |
       Sort-Object PNPClass, Name |
       Format-Table PNPClass, Name -AutoSize

   Get-PhysicalDisk |
       Format-Table MediaType, BusType, @{Name='SizeGiB'; Expression={[math]::Round($_.Size / 1GB, 1)}} -AutoSize
   ```

   Keep the physical Wi-Fi, Ethernet, Bluetooth, audio and camera entries. For a generic name, use **Device Manager → Properties → Details → Hardware Ids** and record the vendor/device or vendor/product pair when shown (for example, `VEN_8086&DEV_XXXX`). Keep machine names, device instance paths, serial numbers, MAC addresses, SSIDs, credentials and recovery keys out of shared evidence.

3. Record the laptop panel resolution and scaling from **Settings → System → Display**, plus the external monitor port and available accessories for later testing.

## 2. Choose a test route

| Route | Use in Stage 2 |
|---|---|
| Windows inventory and the existing VMware guest | Start immediately without removable media. Mount the live ISO as a virtual DVD to rehearse boot and session checks. VMware and WSL 2 provide guest environments. Native HP driver, power and display results require a physical boot. |
| One-time UEFI boot from removable media | Preferred for native testing if permitted on the work laptop. Use a blank **8 GB USB stick** (about **7.45 GiB** nominal) or an approved spare external USB drive. Debian's LXQt live ISO is approximately 3.7 GiB. Writing the image replaces the target drive's contents. |
| UEFI HTTP Boot | A no-stick native option on supported firmware. It requires model-specific support and an approved network boot setup. Consider it after identifying the exact HP model. |

Use the current [Debian 13 LXQt amd64 live ISO](https://cdimage.debian.org/debian-cd/current-live/amd64/iso-hybrid/) and [verify it using Debian's signed checksums](https://www.debian.org/CD/verify). Record the image filename and hash. Rehearse in VMware before writing removable media. The [Debian 13.7 LXQt live package list](https://cdimage.debian.org/debian-cd/current-live/amd64/iso-hybrid/debian-live-13.7.0-amd64-lxqt.iso.packages) includes Xorg, Xfwm4 and PulseAudio. Use the stock image for the initial hardware check. Prepare and verify Openbox and PipeWire/WirePlumber in the VM before using an image for the Stage 2 session and audio-stack checks.

Before booting external media, confirm the work device permits it and check that any BitLocker recovery process is available through the device owner or IT. Keep Secure Boot, BitLocker, permanent boot order and the Windows boot configuration as configured. Use the model's one-time boot menu (typically **Esc → F9** on HP). Choose the live session, leave the internal drives unmounted and skip the installer. If the media is blocked or a firmware change is required, stop and record the blocker. After the test, shut down, remove the media and confirm normal Windows startup.

## 3. Run the first native checks

1. Record boot success, the image hash, `uname -r`, and the session. Run `printf '%s / %s\n' "$XDG_CURRENT_DESKTOP" "$XDG_SESSION_TYPE"` and `pgrep -x openbox`. Record **LXQt / x11** and a running Openbox process before counting the run as the Stage 2 baseline. If the stock live image starts with Xfwm4, record the result as a stock live hardware check. For the baseline image, use the [Stage 1 Xorg package list](https://github.com/aakashj16/veloce_os/blob/eb76c0fe778faebd525d7efefd1ca6c1d9d7cd37/system/packages/stage1-xorg.txt) to prepare and verify the Openbox session in the VM before the next physical run.
2. Use `lspci -nnk` and `lsusb` to identify devices and loaded PCI drivers. Check native panel resolution, brightness and an external monitor, Wi-Fi connect/reconnect, Bluetooth pairing, speakers, microphone, webcam and touchpad. If initial checks pass, test lid-close and repeated suspend/resume, then recheck Wi-Fi, display and audio.
3. Record per-device results in the [hardware matrix](hardware-test-matrix.md#physical-test-checklist). Use an approved disposable external device for filesystem write/eject tests. Defer encrypted installation and recovery to a disposable target. Track live-image and installed-system memory measurements separately using the [performance protocol](../README.md#what-the-performance-budgets-mean).

References for the test paths include [Debian 13 release notes for live and HTTP boot](https://www.debian.org/News/2025/20250809), [HP's one-time USB boot instructions](https://www.hp.com/my-en/tech-takes/software/how-to/how-to-boot-from-usb-drive-on-windows-10-pcs.html), [Microsoft's BitLocker guidance](https://learn.microsoft.com/en-us/windows/security/information-protection/bitlocker/bitlocker-key-management-faq) and [Microsoft's WSL 2 FAQ](https://learn.microsoft.com/en-us/windows/wsl/faq).
