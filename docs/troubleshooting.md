# Troubleshooting: NVIDIA `nouveau` boot hang on hybrid graphics (ARIA / HW-01)

**Recorded:** 2026-11-24  
**Status:** Boot restored with a persistent Intel-only workaround; NVIDIA root cause **not confirmed**.  
**Evidence:** Owner-reported live recovery and command output on ARIA (stock Debian 13), **not a Veloce OS image or release test**.  
**Related:** [ARIA field report](field-reports/aria-debian13-2026-11-17.md) · [HW-01 matrix](hardware-test-matrix.md) · [Stage 2](stage-2.md)

## Affected reference system

- **Machine:** HW-01, HP Laptop 15s-dr2xxx / 15s-dr2007tx (SKU `191F4PA#ACJ`).
- **OS/session:** Debian 13 (Trixie), KDE Plasma on Wayland, GRUB/UEFI.
- **Kernel at incident:** `6.12.118+deb13-amd64` (shown in GRUB and `update-grub` output).
- **Integrated graphics:** Intel Iris Plus Graphics G1 (Ice Lake, PCI `8086:8a56`), driver `i915`.
- **Discrete graphics:** NVIDIA GeForce MX130 / GM108M (PCI `10de:174d`), `nouveau` listed as an available kernel module.

## Symptoms and evidence (2026-11-23 to 2026-11-24)

1. The laptop powered on and displayed HP diagnostics and GRUB, but normal Debian startup hung with a black screen/kernel messages. `Ctrl+Alt+F3` did not recover a text login.
2. The boot messages included `nouveau 0000:01:00.0 ... MMIO read ... FAULT` and `DMAR: [DMA Read NO_PASID] ... fault`. These implicate GPU/PCIe initialization, but do **not** alone prove a driver bug or defective GPU.
3. Adding `modprobe.blacklist=nouveau` **temporarily** at the GRUB editor allowed Debian/KDE to start.
4. `lspci -nnk` then showed `Kernel driver in use: i915` for Intel and **no** `Kernel driver in use` line for the MX130. Seeing `Kernel modules: nouveau` only lists a candidate driver; it does not mean that driver is loaded.
5. The parameter was made persistent in `/etc/default/grub`, followed by a successful `sudo update-grub`. A subsequent **normal reboot, without hand-editing GRUB, succeeded**. A post-reboot `/proc/cmdline` printout was recommended but not supplied, so record the successful boot as the observed verification.

### Separate thermal event — do not conflate causes

On 2026-11-23 HP firmware displayed **System Temperature (90D)** once, the first such reported event. The HP system-board and AC-adapter diagnostics passed, but those tests did **not** certify the fan or thermal sensors. Owner-reported CPU package readings during troubleshooting were approximately 66°C, then 62°C; a later reading was 60°C. Following an internal cleaning on 2026-11-24, fan noise decreased and readings were approximately 58°C, with 59°C at a subsequent startup. These isolated readings are not controlled before/after benchmarks. **No causal link between 90D and the `nouveau` hang has been established.** If a firmware thermal warning recurs, power down and investigate cooling independently.

## Recovery: one-boot workaround

Use only when the display shows GRUB and the failure resembles the documented `nouveau` hang. If the system is reporting overheating, address that first.

1. At the GRUB menu, highlight **Debian GNU/Linux** and press `e`.
2. Find the line beginning `linux /boot/vmlinuz-...` that ends with boot parameters such as `ro quiet`.
3. Append **one space** and `modprobe.blacklist=nouveau` to that line; do not alter the root UUID or other options.
4. Press `Ctrl+X` (or `F10`) to boot.
5. If KDE starts, inspect GPU bindings before changing any persistent configuration:

   ```bash
   lspci -nnk | grep -EA4 'VGA compatible controller|3D controller|Display controller'
   ```

This GRUB editor change affects **one boot only**.

## Persistent workaround (verified on HW-01)

First ensure an Intel-driven desktop successfully boots with the temporary parameter. Then:

```bash
# Save the original configuration before modifying it.
sudo cp -a /etc/default/grub /etc/default/grub.backup-before-nouveau
sudo nano /etc/default/grub
```

On ARIA, the original configuration was:

```bash
GRUB_CMDLINE_LINUX_DEFAULT="quiet"
GRUB_CMDLINE_LINUX=""
```

Change **only** the default line to:

```bash
GRUB_CMDLINE_LINUX_DEFAULT="quiet modprobe.blacklist=nouveau"
```

Leave `GRUB_CMDLINE_LINUX` and all unrelated options unchanged. Save and run:

```bash
grep -E '^GRUB_CMDLINE_LINUX(_DEFAULT)?=' /etc/default/grub
sudo update-grub
sudo reboot
```

On HW-01, `update-grub` found the installed Debian kernels, completed with `done`, and the next ordinary boot succeeded. The warning about `os-prober` not scanning other operating systems was unrelated.

**Additional verification after restarting** (recommended; not yet evidenced in the incident transcript):

```bash
cat /proc/cmdline
lspci -nnk | grep -EA4 'VGA compatible controller|3D controller|Display controller'
```

Expect the parameter in the kernel command line, Intel `i915` active, and the MX130 without a `nouveau` driver bound.

### Reversal / recovery

When a tested alternative NVIDIA configuration is available, restore the saved default and regenerate GRUB:

```bash
sudo cp -a /etc/default/grub.backup-before-nouveau /etc/default/grub
sudo update-grub
```

**Do not remove the blacklist on HW-01 until another driver or kernel path has been tested.** If normal startup fails after a later configuration change, use the one-boot GRUB editor workaround again to recover.

## Interpretation and Veloce OS follow-up

- **Confirmed:** Intel graphics works with `i915`; blacklisting `nouveau` allows a normal desktop boot on HW-01; persistent GRUB workaround succeeded.
- **Not confirmed:** Exact origin of the `nouveau`/DMAR faults; whether a kernel regression, driver/firmware interaction, discrete GPU power state, or hardware defect caused them.
- **Trade-off:** The NVIDIA MX130 is not available for discrete-GPU acceleration under this workaround. It is **not** a physical GPU disablement.
- **Shipping policy:** Do **not** apply a global `nouveau` blacklist to all Veloce OS systems. Treat this as an **HW-01-specific documented fallback** until cross-hardware testing supports a product policy.
- **Remaining validation:** Compare earlier known-good kernels; collect sanitized `journalctl -b -k`, PCI/driver and update-history logs when a safe test path exists; evaluate compatible maintained NVIDIA driver options and Secure Boot implications before installing anything; test hybrid GPU power management, suspend/resume, external display outputs, battery draw and performance. Track the separate thermal-warning history.
- **Release implication:** Graphics reliability/compatibility remains **partially validated**, not passed, for HW-01. A successful Intel-only fallback does not establish NVIDIA support or universal hybrid-graphics compatibility.
