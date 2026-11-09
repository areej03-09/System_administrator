*This project has been created as part of the 42 curriculum by aaldobag.*

# Born2beRoot

## Description

Born2beRoot is a system administration project. The goal is to set up a minimal, hardened Debian server inside a VirtualBox virtual machine, with no graphical interface, and to apply strict security rules: an encrypted LVM disk layout, SSH on a custom port, a firewall, a strong password policy, a restricted `sudo` configuration, and a monitoring script that broadcasts system information every 10 minutes.

### Main design choices

- **Operating system:** Debian 13 (Trixie), stable branch, no desktop environment installed.
- **Partitioning:** encrypted LVM on `sda5`, with separate logical volumes for `/` (root), `swap` and `/home`. `/boot` stays outside the encrypted volume because the bootloader must read it before decryption.
- **SSH:** OpenSSH on port 4242, root login disabled.
- **Firewall:** UFW active at boot, with only port 4242 allowed.
- **User management:** user `aaldobag` belongs to the `sudo` and `user42` groups. Hostname is `aaldobag42`.
- **Password policy:** expiry every 30 days, minimum 2 days between changes, warning 7 days before expiry (`/etc/login.defs` and `chage`). Quality rules via `libpam-pwquality`: minimum 10 characters, upper case, lower case and digit required, at most 3 repeated characters, user name not allowed, at least 7 characters different from the previous password.
- **sudo policy** (`/etc/sudoers.d/sudo_config`): 3 attempts, custom bad-password message, input/output logging in `/var/log/sudo/`, TTY required, restricted `secure_path`.
- **Services installed:** only `ssh`, `ufw`, `cron`, `apparmor` and `sudo`.
- **Monitoring:** `monitoring.sh` (bash) scheduled by cron at boot and every 10 minutes, using `wall`.

### Comparisons

**Debian vs Rocky Linux**
- Debian: large community and package repository, easier for beginners, stable release cycle, uses `apt`/`dpkg`, AppArmor by default.
- Rocky Linux: RHEL-compatible, oriented towards enterprise, uses `dnf`/`rpm`, SELinux and firewalld by default, longer support cycles but more complex to configure.
- Choice: Debian, because it is recommended for newcomers to system administration.

**AppArmor vs SELinux**
- AppArmor confines programs through path-based profiles. It is simpler to read and write.
- SELinux labels every file and process and applies policies on those labels. It is more granular and powerful, but more complex.

**UFW vs firewalld**
- UFW is a simple front-end for iptables/nftables with straightforward commands, typical on Debian/Ubuntu.
- firewalld manages rules through zones and services, can change rules dynamically without restarting, typical on RHEL-based systems.

**VirtualBox vs UTM**
- VirtualBox is free and cross-platform on x86 hosts (Windows, Linux, Intel macOS). It stores disks as `.vdi`.
- UTM is built on QEMU and is the option for Apple Silicon Macs, where it can emulate or virtualize ARM and x86 systems. It stores disks as `.qcow2`.

## Instructions

1. Create a VM in VirtualBox (Linux / Debian 64-bit, 1 GB RAM, about 30 GB dynamic VDI disk), skipping unattended installation.
2. Add a NAT port-forwarding rule: host port 4242 to guest port 4242.
3. Install Debian with the guided "encrypted LVM" option and a separate `/home`. Do not select any desktop environment.
4. Configure the system as described above (user and groups, SSH, UFW, password policy, sudo).
5. Copy `monitoring.sh` to `/usr/local/bin/` and add to root's crontab:
   ```
   @reboot /usr/local/bin/monitoring.sh
   */10 * * * * /usr/local/bin/monitoring.sh
   ```
6. Connect from the host with `ssh aaldobag@127.0.0.1 -p 4242`.
7. Signature: with the VM powered off, compute the SHA-1 of the `.vdi` file (for example `certUtil -hashfile born2beroot.vdi sha1` on Windows) and paste it into `signature.txt`.

## Resources

- Debian documentation: https://www.debian.org/doc/
- `man sudoers`, `man sshd_config`, `man pwquality.conf`, `man login.defs`, `man chage`, `man crontab`
- Debian Wiki on AppArmor and UFW: https://wiki.debian.org/AppArmor and https://wiki.debian.org/Uncomplicated%20Firewall%20%28ufw%29
- VirtualBox manual: https://www.virtualbox.org/manual/

### Use of AI

AI (Claude) was used as a step-by-step guide during the setup, mainly to explain what each configuration step does and why (LVM and encryption, SSH hardening, UFW, password policy, sudo options) and to help debug issues during the installation (partition layout, SSH port forwarding, command mistakes). It was also used to help draft the structure of `monitoring.sh` and of this README. All configuration was typed, tested and verified on the virtual machine by the author, and every part was reviewed so that it can be explained during the defense.
