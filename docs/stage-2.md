# Stage 2 · HW-01 hardware foundation

**Status** — In progress. The Windows hardware inventory identifies the HP 15s-dr2007tx, BIOS Insyde F.68, UEFI mode and Secure Boot On. Native Linux checks are pending.

Use Debian 13 amd64 and **LXQt on Xorg with Openbox** as the Stage 2 baseline. [Draft PR #2](https://github.com/aakashj16/veloce_os/pull/2) records the Stage 1 VMware results. The 725 MiB Xorg reading is a VM reference. Record live-session measurements separately from installed-system acceptance. The production session decision is tracked in [ADR 0002](adr/0002-desktop-sessions.md).

## 1. Complete the Windows inventory

The [hardware matrix](hardware-test-matrix.md#first-reference-laptop) records the Windows evidence for both GPUs, Intel AX201 Wi-Fi, Realtek Ethernet, Intel Bluetooth, Intel SST/Realtek audio, the webcam and both internal drives. The inventory is sufficient to start live testing. Capture exact device IDs during the live session if needed for troubleshooting.

1. For future inventories, press **Win + R**, enter `msinfo32`, and record the System Model, System SKU, BIOS Version/Date, BIOS Mode, Secure Boot State and Windows Version.
2. Open regular **Windows PowerShell** and run these read-only queries. They show device names and storage types.

   ```powershell
   Get-CimInstance -ClassName Win32_PnPEntity -Filter "Present = TRUE" |
       Where-Object { $_.PNPClass -in @('Net', 'Bluetooth', 'MEDIA', 'Camera', 'Image') } |
       Sort-Object PNPClass, Name |
       Format-Table PNPClass, Name -AutoSize

   Get-PhysicalDisk |
       Format-Table MediaType, BusType, @{Name='SizeGiB'; Expression={[math]::Round($_.Size / 1GB, 1)}} -AutoSize
   ```

   Keep the physical Wi-Fi, Ethernet, Bluetooth, audio and camera entries. If a Windows hardware ID is needed, expand the device category in Device Manager, right-click the individual device, then select **Properties → Details → Hardware Ids**. Record the vendor/device or vendor/product pair when shown (for example, `VEN_8086&DEV_XXXX`). Keep machine names, device instance paths, serial numbers, MAC addresses, SSIDs, credentials and recovery keys out of shared evidence.

3. Record the laptop panel resolution and scaling from **Settings → System → Display**, plus the external monitor port and available accessories for later testing.

## 2. Choose a test route

| Route | Use in Stage 2 |
|---|---|
| Windows inventory and the existing VMware guest | Start immediately without removable media. Mount the live ISO as a virtual DVD to rehearse boot and session checks. VMware and WSL 2 provide guest environments. Native HP driver, power and display results require a physical boot. |
| One-time UEFI boot from removable media | Preferred for native testing if permitted on the work laptop. Use a blank **8 GB USB stick** (about **7.45 GiB** nominal) or an approved spare external USB drive. Debian's LXQt live ISO is approximately 3.7 GiB. Writing the image replaces the target drive's contents. |
| UEFI HTTP Boot | A no-stick native option on supported firmware. HP documents network boot generally, but HTTP Boot support for HW-01 is unconfirmed. Use the removable-media route for native testing unless HTTP Boot support and an approved network setup are confirmed. |

Use the current [Debian 13 LXQt amd64 live ISO](https://cdimage.debian.org/debian-cd/current-live/amd64/iso-hybrid/) and [verify it using Debian's signed checksums](https://www.debian.org/CD/verify). Record the image filename and hash. Rehearse in VMware before writing removable media. The [Debian 13.7 LXQt live package list](https://cdimage.debian.org/debian-cd/current-live/amd64/iso-hybrid/debian-live-13.7.0-amd64-lxqt.iso.packages) includes Xorg, Xfwm4 and PulseAudio. Use the stock image for the initial hardware check. Prepare and verify Openbox and PipeWire/WirePlumber in the VM before using an image for the Stage 2 session and audio-stack checks.

Before booting external media, confirm the work device permits it and check that any BitLocker recovery process is available through the device owner or IT. Keep Secure Boot, BitLocker, permanent boot order and the Windows boot configuration as configured. Use the model's one-time boot menu (typically **Esc → F9** on HP). Choose the live session, leave the internal drives unmounted and skip the installer. If the media is blocked or a firmware change is required, stop and record the blocker. After the test, shut down, remove the media and confirm normal Windows startup.

## 3. Run the first native checks

1. Record boot success, the image hash, `uname -r`, and the session. Run `printf '%s / %s\n' "$XDG_CURRENT_DESKTOP" "$XDG_SESSION_TYPE"` and `pgrep -x openbox`. Record **LXQt / x11** and a running Openbox process before counting the run as the Stage 2 baseline. If the stock live image starts with Xfwm4, record the result as a stock live hardware check. For the baseline image, use the [Stage 1 Xorg package list](https://github.com/aakashj16/veloce_os/blob/eb76c0fe778faebd525d7efefd1ca6c1d9d7cd37/system/packages/stage1-xorg.txt) to prepare and verify the Openbox session in the VM before the next physical run.
2. Use `lspci -nnk` and `lsusb` to identify devices and loaded PCI drivers. Look for [`iwlwifi` on the AX201](https://www.intel.com/content/www/us/en/support/articles/000005511/wireless.html) and test Wi-Fi connect/reconnect plus wired Ethernet. Check Bluetooth pairing, speakers, the internal microphone, webcam and touchpad. If the internal microphone or speakers are missing and the kernel log reports an SOF firmware error, check Debian's [`firmware-sof-signed` package](https://packages.debian.org/trixie/firmware-sof-signed). Check the 1920 × 1080 panel, brightness and external monitor output with both GPUs enumerated. If initial checks pass, test lid-close and repeated suspend/resume, then recheck Wi-Fi, display and audio.
3. Record per-device results in the [hardware matrix](hardware-test-matrix.md#physical-test-checklist). Use an approved disposable external device for filesystem write/eject tests. Defer encrypted installation and recovery to a disposable target. Track live-image and installed-system memory measurements separately using the [performance protocol](../README.md#what-the-performance-budgets-mean).

References for the test paths include [Debian 13 release notes for live and HTTP boot](https://www.debian.org/News/2025/20250809), [HP's 15s-dr2000 family support page](https://support.hp.com/in-en/product/product-specs/hp-15s-dr2000-laptop-pc-series/32612075), [HP's general boot options](https://support.hp.com/in-en/document/ish_3891464-2337914-16), [HP's one-time USB boot instructions](https://www.hp.com/my-en/tech-takes/software/how-to/how-to-boot-from-usb-drive-on-windows-10-pcs.html), [Microsoft's BitLocker guidance](https://learn.microsoft.com/en-us/windows/security/information-protection/bitlocker/bitlocker-key-management-faq) and [Microsoft's WSL 2 FAQ](https://learn.microsoft.com/en-us/windows/wsl/faq).
