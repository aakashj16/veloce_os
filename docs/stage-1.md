# Stage 1 · Minimal Debian desktop prototype

**Status** — In progress. VM installation and session checks are pending.

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

## Wayland follow-up

Trixie packages [Labwc](https://packages.debian.org/trixie/labwc) and [Wayfire](https://packages.debian.org/trixie/wayfire). The Debian package index lists [LXQt's Wayland session package](https://packages.debian.org/search?keywords=lxqt-wayland-session) only for testing and unstable. Keep the Trixie package sources. Evaluate a versioned upstream LXQt 2.1-compatible session integration or a suitable Trixie backport in a VM checkpoint. Then verify complete LXQt login, lock and logout with each compositor. A standalone compositor login is useful for diagnosis. The [Stage 1 exit gate](../README.md#stage-1--minimal-debian-desktop-proof-of-concept) requires the complete LXQt sessions and recorded baselines.
