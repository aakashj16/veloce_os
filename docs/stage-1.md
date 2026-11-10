# Stage 1 · Minimal Debian desktop prototype

**Status** — Complete. Xorg, Labwc, and Wayfire passed the Stage 1 VM exit checks. The initial measurements and known issues are recorded below.

This is the first slice of [BASE-001](requirements.md). Establish an LXQt login on Xorg with Openbox, then use a comparable VM configuration for the Labwc and Wayfire trials.

## Development shell and test VM

Debian on WSL can be used for editing, Git and shell tools. Use a separate disposable VM to test the Debian installer, boot, graphical sign-in and complete desktop sessions. WSLg integrates individual Linux apps with Windows. See [Microsoft's WSL GUI documentation](https://learn.microsoft.com/en-us/windows/wsl/tutorials/gui-apps).

## 1. Create the minimal Debian VM

1. Download the [Debian 13 amd64 netinst ISO](https://www.debian.org/download). Verify it against Debian's [signed checksums](https://www.debian.org/CD/verify) and record the exact filename and digest.
2. Start with 2 virtual CPUs, 4 GiB of fixed RAM, a new 25 GiB virtual disk and a network adapter with internet access. Use a local VM console for desktop checks. Record the hypervisor version, firmware mode and virtual graphics settings.
   - **Windows Pro or Enterprise.** [Hyper-V Manager](https://learn.microsoft.com/en-us/windows-server/virtualization/hyper-v/get-started/create-a-virtual-machine-in-hyper-v) is one option. Use a Generation 2 VM, the Default Switch if available, and the [Microsoft UEFI Certificate Authority template](https://learn.microsoft.com/en-us/windows-server/virtualization/hyper-v/learn-more/Generation-2-virtual-machine-security-settings-for-Hyper-V) if Secure Boot is enabled.
   - **Windows Home, including Home Single Language.** Install [VMware Workstation Pro for Windows](https://knowledge.broadcom.com/external/article/368667/download-and-license-vmware-desktop-hypervisor.html). Keep WSL 2 and Windows security settings enabled. If VMware asks to enable Windows Hypervisor Platform, accept and restart when prompted. Create a new VM and choose **I will install the operating system later** to use Debian's installer. Select Linux and the closest available 64-bit Debian guest profile. Use a new 25 GiB virtual disk, NAT networking, and 2 virtual CPUs total. Set the VM to UEFI and attach the verified ISO to its virtual CD/DVD drive before powering on. Leave nested virtualization off. VMware documents its [Windows Hypervisor Platform mode](https://knowledge.broadcom.com/external/article/315420/) for Windows hosts with the hypervisor active.
3. Boot the ISO. Set up an ordinary VM account and leave the installer root password blank so the first account receives sudo access. Select guided partitioning on the new virtual disk only. In **Software selection**, leave **standard system utilities** checked and clear **Debian desktop environment** and any desktop or server tasks.
4. Finish the installation, eject the ISO and boot the installed system. Sign in at the text console and check the output below. Expect `VERSION_CODENAME=trixie`, `amd64` and a VM type such as `microsoft`, `vmware` or `oracle`. If the type is `wsl` or `none`, return to the VM console before installing desktop packages.

```bash
cat /etc/os-release
dpkg --print-architecture
systemd-detect-virt
```

## 2. Install the Xorg baseline

Run the following in the VM from a checkout containing this guide. On a fresh Debian VM, install Git with `sudo apt update && sudo apt install git` if needed. Record the checkout commit with `git rev-parse HEAD`.

```bash
sudo apt update
sudo apt upgrade
xargs -r -a system/packages/stage1-xorg.txt sudo apt-get -s install
xargs -r -a system/packages/stage1-xorg.txt sudo apt-get install
sudo systemctl set-default graphical.target
sudo reboot
```

Review the APT simulation before running the install. Keep the installer-configured Debian 13 sources and normal APT recommendations. If prompted for a display manager, select **SDDM**. The [package manifest](../system/packages/stage1-xorg.txt) tracks the requested packages. Save the resolved versions separately for each run so Debian security updates can continue.

## 3. Test login, lock and logout

1. In SDDM, select **LXQt**. If LXQt prompts for a window manager, select **Openbox**. Confirm that the LXQt panel appears and that QTerminal and PCManFM-Qt open from the application menu.
2. In QTerminal, run the following. Expect `x11`, an Openbox process, and a successful DNS lookup.

   ```bash
   printf '%s\n' "$XDG_SESSION_TYPE"
   pgrep -a -x openbox
   getent hosts deb.debian.org
   ```

3. Select **Lock Screen** from the LXQt menu and verify that unlocking requires the VM account password. If locking fails and `pgrep -x xscreensaver` finds no process, add `xscreensaver -no-splash` in **LXQt Session Settings → Autostart**, sign out and back in, then retry. Record any remaining failure. See the [LXQt locker guide](https://lxqt-project.org/wiki/Screensaver-and-Screen-Locker.html).
4. Log out to SDDM, then reboot and repeat the LXQt login. Record each result. If the greeter fails, try a VM text console and inspect `systemctl status display-manager` and `sudo journalctl -b -u sddm`.

## 4. Record initial VM evidence

Record the date, Git commit, ISO filename and checksum, configured APT suites and mirror, hypervisor settings, kernel and session. Record Pass, Fail, Blocked or Not run for boot, login, terminal, file manager, network, lock/unlock and logout. Keep raw output local and remove usernames and other device identifiers before publishing any excerpt.

```bash
uname -r
dpkg-query -W -f='${db:Status-Abbrev}\t${binary:Package}\t${Version}\n' |
  awk '$1 == "ii" { print $2 "\t" $3 }' > "$HOME/stage1-installed-packages.tsv"
systemd-analyze time
systemctl --failed
free -m
awk '/^MemTotal:/ { total = $2 } /^MemAvailable:/ { printf "Memory used: %.1f MiB\n", (total - $2) / 1024 }' /proc/meminfo
vmstat -w 1 6
```

Record boot-to-greeter time separately with a stopwatch. Use the last five rows of `vmstat` for interval data. These are early VM measurements with a test terminal open. Use the [five-sample protocol](../README.md#what-the-1-gib-requirement-means) on reference hardware for the 1 GiB acceptance test.

### First Xorg VM run

Run date **2026-11-10 (IST)**. Setup revision [`9635539`](https://github.com/aakashj16/veloce_os/commit/96355390c206b58eaf8adcc41fab2db7b1e3f25a). The guest ran Debian 13.7 amd64 with kernel `6.12.116+deb13-amd64` and LXQt on Xorg with Openbox. The VMware Workstation Pro 26H1 VM used UEFI, 2 virtual CPUs, 4 GiB of assigned RAM, a 25 GiB virtual disk, and NAT on a Windows 11 Home Single Language host. APT used `deb.debian.org` for `trixie` and `trixie-updates` and `security.debian.org` for `trixie-security`.

ISO `debian-13.7.0-amd64-netinst.iso`. SHA-512 matched Debian's published checksum.

```text
ef04d0276850d70e4aaec0b55003628660e9631b26e35215a70ab3c8c707294a4b12731bc73c75a932902f703a7c278310ca99df28d6b36fb06d5618d8d0f678
```

| Check | Result |
|---|---|
| Debian boot and SDDM login | Pass |
| LXQt session type and window manager | Pass. `LXQt`, `x11`, `/usr/bin/openbox` |
| QTerminal and file manager | Pass |
| DNS lookup for `deb.debian.org` | Pass |
| Lock and password unlock | Pass |
| Logout and repeat login after reboot | Pass |

| Measurement | Result |
|---|---|
| System startup | 28.389 s. Kernel 17.904 s, userspace 10.484 s |
| Graphical target | Reached after 10.480 s in userspace |
| Failed system units | 0 |
| Guest memory reported by `free -m` | 753 MiB used, 3,129 MiB available, 3,883 MiB total |
| Swap used | 0 MiB of 1,327 MiB |
| CPU idle, five one-second intervals | 99%, 99%, 98%, 92%, 94% |

The VM hit a virtual CPU fault during legacy BIOS startup. Switching its firmware to UEFI reached the boot manager and allowed installation from the verified ISO. The first graphical login opened KDE Plasma on Wayland. Selecting **LXQt** in the SDDM session menu provided the Xorg session for these tests.

The run used an open QTerminal with VMware 3D acceleration disabled. The resolved package inventory was saved in the guest as `~/stage1-installed-packages.tsv` before taking the Xorg baseline snapshot. Stopwatch boot-to-greeter timing and the physical-hardware memory acceptance test remain open.

## 5. Trial LXQt with Labwc

Start from the powered-off **Stage 1 – Xorg baseline** VMware snapshot. If the VM is already at that baseline, power it on normally. Sign in to **LXQt** on Xorg and open QTerminal.

Trixie ships [Labwc](https://packages.debian.org/trixie/labwc) and LXQt 2.1, while Debian's [Wayland session package](https://packages.debian.org/search?keywords=lxqt-wayland-session) is available for testing and unstable. For this disposable VM, use upstream [lxqt-wayland-session 0.1.0](https://github.com/lxqt/lxqt-wayland-session/releases/tag/0.1.0), the release paired with LXQt 2.1. Keep the Trixie APT sources and existing desktop packages.

### Install the trial packages

```bash
sudo apt update
sudo apt install labwc swaybg swaylock qt6-wayland
sudo apt install build-essential cmake pkgconf liblxqt-dev qt6-tools-dev libkf6windowsystem-dev xdg-user-dirs
```

Labwc runs the Wayland session. Swaylock is for the lock and unlock test. Swaybg supplies the wallpaper used in the upstream Labwc configuration. The second install command adds the tools needed to build the session helper.

### Build the pinned session helper

Run these from the VM user's home directory. The checkout and build use the regular account. Only the final install requires sudo. The upstream project uses the `/usr` install prefix for these session files.

```bash
cd ~
git clone --depth 1 --branch 0.1.0 https://github.com/lxqt/lxqt-wayland-session.git
cd lxqt-wayland-session
git rev-parse HEAD
```

The commit must be `c8cbc996b58bce24c2564b62b99c4f6970106bf3`. If it matches, run:

```bash
cmake -S . -B build -DCMAKE_INSTALL_PREFIX=/usr
cmake --build build
```

Proceed only if both commands succeed.

```bash
sudo cmake --install build
ls /usr/share/wayland-sessions/lxqt-wayland.desktop
```

### Start and check the Labwc session

1. While still in LXQt on Xorg, run `lxqt-config-session`. Under **Wayland Settings**, choose **Labwc** as the compositor and enter `swaylock` as the Wayland screen locker command. Save the changes.
2. Log out. In the SDDM session selector, choose **LXQt (Wayland)** and sign in. Select **LXQt** in the same menu to return to the Xorg session if the trial fails to start.
3. In QTerminal, verify that the desktop name contains `LXQt`, the session type is `wayland`, and the Labwc process is running.

   ```bash
   echo "$XDG_CURRENT_DESKTOP"
   echo "$XDG_SESSION_TYPE"
   pgrep -a -x labwc
   ```

4. Repeat the file manager, network, lock and unlock, logout, and reboot checks. Record the same initial performance measurements as the Xorg run and capture the package versions for this trial. If the Wayland login returns to the greeter, keep the relevant lines from `~/.local/share/sddm/wayland-session.log` before retrying. Remove personal data before publishing excerpts.

### First Labwc VM login

On 2026-11-10 (IST), the session helper built successfully. Labwc 0.8.3 started the LXQt desktop after two VMware graphics issues were addressed:

1. The initial log reported `VMware: No 3D enabled` and `EGL_NOT_INITIALIZED`. With the VM powered off, enable **VM → Settings → Hardware → Display → Accelerate 3D graphics**.
2. The next login reached Xwayland but failed to import a DMA-BUF graphics buffer. For this VM, use the following Labwc-only override from the existing LXQt/Xorg session. This keeps VMware 3D enabled and disables Glamor and DRI3 in Xwayland for this trial.

   ```bash
   mkdir -p ~/.config/labwc/environment.d
   echo 'XWAYLAND_NO_GLAMOR=1' > ~/.config/labwc/environment.d/90-stage1-xwayland.env
   ```

The retry displayed the LXQt desktop and QTerminal. `XDG_SESSION_TYPE` returned `wayland`, and the running Labwc process launched `lxqt-session`. See the [Debian Xwayland manual](https://manpages.debian.org/trixie/xwayland/Xwayland.1.en.html#XWAYLAND_NO_GLAMOR).

### First Labwc VM run

Run date **2026-11-10 (IST)**. VMware 3D acceleration was enabled, the Labwc Xwayland override was active, and QTerminal was open during measurement.

| Check | Result |
|---|---|
| Debian boot and LXQt Wayland sign-in | Pass, including sign-in after a VM power cycle |
| QTerminal and file manager | Pass. Home and Downloads opened |
| DNS lookup | Pass |
| Lock and password unlock | Pass on retest. The first unlock showed swaylock’s `Wrong` indicator and the VM was powered off. The cause of the first rejection is unconfirmed |
| Logout and repeat login | Pass |

| Measurement | Result |
|---|---|
| System startup | 28.916 s. Kernel 17.938 s, userspace 10.977 s |
| Graphical target | Reached after 10.974 s in userspace |
| Failed system units | 0 |
| Guest memory reported by `free -m` | 821 MiB used, 3,061 MiB available, 3,883 MiB total |
| Swap used | 0 MiB of 1,327 MiB |
| CPU idle, five one-second intervals | 99%, 94%, 99%, 99%, 100% |

Stopwatch boot-to-greeter time and the full list of trial package versions are pending. The reference-hardware memory acceptance test remains open.

### Xorg retest with VMware 3D enabled

On 2026-11-10 (IST), the same VM returned to LXQt on Xorg with VMware 3D acceleration enabled and QTerminal open. The Labwc trial packages were still installed.

| Measurement | Xorg | Labwc |
|---|---:|---:|
| Guest memory used, `free -m` | 725 MiB | 821 MiB |
| Guest memory available, `free -m` | 3,157 MiB | 3,061 MiB |
| Swap used | 0 MiB | 0 MiB |
| Failed system units | 0 | 0 |
| CPU idle, five one-second intervals | 99%, 100%, 99%, 100%, 99% | 99%, 94%, 99%, 99%, 100% |

Xorg used 96 MiB less in this pair of VM readings. Both sessions displayed the same `systemd-analyze time` figures, including 28.916 s startup and 10.974 s to the graphical target in userspace. Those figures describe system boot. Timing a desktop sign-in requires a separate measurement.

## Wayfire follow-up

Trixie also ships [Wayfire](https://packages.debian.org/trixie/wayfire). For this comparison, take a separate powered-off checkpoint of the current VM before installing Wayfire. Use the same VM settings and the pinned LXQt session helper from the Labwc trial. Keep the Labwc Xwayland override scoped to Labwc for the first Wayfire login.

From LXQt on Xorg, install the package and check its version.

```bash
sudo apt install wayfire
wayfire --version
```

Open `lxqt-config-session`. Under **Wayland Settings**, choose **Wayfire** and use `swaylock` as the screen locker. Log out and select **LXQt (Wayland)** in SDDM. Verify the session using `echo "$XDG_SESSION_TYPE"` and `pgrep -a -x wayfire`. If login fails, return to **LXQt** in SDDM and inspect `~/.local/share/sddm/wayland-session.log`.

The [Stage 1 exit gate](../README.md#stage-1--minimal-debian-desktop-proof-of-concept) requires both complete LXQt Wayland sessions and their initial measurements.

### First Wayfire VM run

On 2026-11-10 (IST), `XDG_SESSION_TYPE` returned `wayland` and the Wayfire process was using `lxqt-wayfire.ini`. The binary reported `0.9.0-unknown`.

| Check | Result |
|---|---|
| Debian boot and LXQt Wayland sign-in | Pass, including sign-in after a normal restart |
| QTerminal and file manager | Pass |
| DNS lookup for `deb.debian.org` | Pass |
| Lock and password unlock | Pass |
| Logout and repeat login | Pass |

| Measurement | Result |
|---|---|
| System startup | 31.755 s. Kernel 16.707 s, userspace 15.048 s |
| Graphical target | Reached after 15.045 s in userspace |
| Failed system units | 0 |
| Guest memory reported by `free -m` | 717 MiB used, 3,165 MiB available, 3,883 MiB total |
| Swap used | 0 MiB of 1,327 MiB |
| CPU idle, five one-second intervals | 99%, 99%, 100%, 99%, 100% |

## Stage 1 VM comparison

These readings come from the same VM with VMware 3D acceleration enabled and QTerminal open.

| Session | RAM used | CPU idle across five samples | Swap used | Failed system units |
|---|---:|---:|---:|---:|
| LXQt on Xorg with Openbox | 725 MiB | 99–100% | 0 MiB | 0 |
| LXQt on Wayland with Labwc | 821 MiB | 94–100% | 0 MiB | 0 |
| LXQt on Wayland with Wayfire | 717 MiB | 99–100% | 0 MiB | 0 |

Wayfire used 104 MiB less than Labwc and 8 MiB less than Xorg in these readings. Each session has one idle memory reading in this comparison. Boot timings describe system startup, and sign-in and stopwatch boot-to-greeter timings were not measured. The physical-hardware memory acceptance test and compositor selection remain open.

## Stage 2 handoff

Use LXQt on Xorg with Openbox as the provisional baseline for the [physical laptop checks](hardware-test-matrix.md). It was simpler to set up in the VM, passed the desktop checks, and used 725 MiB at idle. The 8 MiB gap to Wayfire is based on one reading per session. The shipped default session remains a [Stage 4 decision](adr/0002-desktop-sessions.md).

Preserve the original Xorg baseline snapshot. A separate powered-off `Stage 1 - Complete` snapshot preserves the VM with both Wayland trials available for later comparison. The physical laptop checks can proceed independently.
