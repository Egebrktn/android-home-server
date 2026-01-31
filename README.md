# Android Home Server

This repository documents how to turn an **Android device** into a low-power,
always-on **home server** using a Debian Linux chroot environment.

No public IP or port forwarding is required.

---

## Table of Contents

- Overview
- Requirements
- Debian Chroot Setup
- External Storage
- Services Overview
  - Navidrome
  - File Browser
  - Web Hosting (Nginx + Cloudflare)
  - Remote Access (Tailscale)
- Notes
- License

---

## Overview

Modern Android devices are powerful, energy-efficient and always connected.
This project shows how to repurpose an old or spare Android phone as a personal home server.

Features:
- Debian Linux via real chroot (not proot)
- Navidrome music streaming server
- Cloudflare Tunnel for external access
- Low power consumption

---

## Requirements

- Rooted Android device
- Termux (F-Droid version recommended)
- Working `su` (Magisk or KernelSU)
- At least 4 GB free storage
- Stable internet connection

---

## Debian Chroot Setup

### Install Required Packages

Open Termux and run:

```bash
pkg update && pkg upgrade -y
pkg install wget tar -y
```

Check root access:

```bash
su
```

If you get a root shell, continue.

#### Optional: Enable SSH in Termux

For convenience, SSH can be enabled in Termux to manage the setup from another machine.

This step is optional and not required for the Debian chroot installation.

```bash
pkg install openssh
sshd
```


Connect from another machine:

```bash
ssh USERNAME@DEVICE_IP -p 8022
```

Replace `USERNAME` with your Termux username (from `whoami`) and `DEVICE_IP` with your Android device’s local IP address.

---

### Download Debian RootFS

This setup uses a minimal Debian 13 (Trixie) root filesystem provided as a compressed `rootfs.tar.xz` archive.

```bash
cd /data/local/tmp
wget <DEBIAN_ROOTFS_URL>
```

Replace `<DEBIAN_ROOTFS_URL>` with a Debian rootfs archive.

> **Note:**  
> This setup was created using a Debian 13 rootfs downloaded from:
> https://github.com/termux/proot-distro


#### Architecture

The Debian root filesystem must match the device architecture.

Most modern Android devices use **ARM64 (aarch64)**.

If you are unsure, you can check your architecture in Termux:

```bash
uname -m
```

Common values:
- `aarch64` → arm64
- `armv7l` → armhf
- `x86_64` → amd64


---

### Extract RootFS

```bash
mkdir -p /data/debian
tar -xpf rootfs.tar.xz -C /data/debian
```

---

### Create start script

The system is started using a custom `start.sh` script.

The script performs the following steps:
- Cleans up any previously mounted filesystems
- Mounts `/dev`, `/dev/pts`, `/proc`, and `/sys`
- Fixes common device permission issues
- Enters the Debian chroot environment using a login shell


Create a file named `start.sh`:

```bash
nano start.sh
```

Paste the following content:

```bash
#!/system/bin/sh

# Path to Debian root filesystem
D_PATH=/data/debian/debian-rootfs

echo "[*] Cleaning up old mounts (if any)..."

# Clean up old mounts to avoid mount conflicts
umount -l $D_PATH/dev/pts 2>/dev/null
umount -l $D_PATH/dev 2>/dev/null
umount -l $D_PATH/proc 2>/dev/null
umount -l $D_PATH/sys 2>/dev/null

echo "[*] Mounting required filesystems..."

# Mount required filesystems
mount -o bind /dev $D_PATH/dev
mount -t devpts devpts $D_PATH/dev/pts
mount -t proc proc $D_PATH/proc
mount -t sysfs sysfs $D_PATH/sys

echo "[*] Fixing device permissions..."

# Fix device node permissions
chmod 666 $D_PATH/dev/null
chmod 666 $D_PATH/dev/zero
chmod 666 $D_PATH/dev/random
chmod 666 $D_PATH/dev/urandom

echo "[*] Entering Debian chroot..."

# Enter Debian chroot
chroot $D_PATH /bin/bash --login
```

---
### Start Debian

Run the script as root:

```bash
su
data/debian/start.sh
```

You are now inside the Debian chroot environment.

> **Note:** You must run `start.sh` again after device reboot to re-enter the chroot.

### Post Installation Setup

```bash
apt update
apt install sudo nano curl locales -y
```

Configure locale and timezone:

```bash
dpkg-reconfigure locales
dpkg-reconfigure tzdata
```

### External HDD Mount

An external HDD is mounted on the Android host and then bind-mounted into the Debian chroot.

Create mount point inside the chroot:
```bash
mkdir -p /data/debian/debian-rootfs/mnt/hdd
```


#### Method 1: Manual mount (recommended)

Use this method if you know the mount path of your external drive.

Bind mount the HDD (run in Termux):
```bash
mount --bind /storage/XXXX-XXXX /data/debian/debian-rootfs/mnt/hdd
```

Replace `XXXX-XXXX` with your external drive identifier.


#### Method 2: Automatic detection

This method automatically selects the first available external storage device.

Bind mount the HDD (run in Termux):
```bash
mount -o bind $(ls -d /mnt/media_rw/* /storage/[0-9A-F]*-[0-9A-F]* 2>/dev/null | head -n 1) /data/debian/debian-rootfs/mnt/hdd
```



After mounting, the external HDD will be accessible inside the chroot at `/mnt/hdd`.

> Note: This mount needs to be re-applied after device reboot.


---

## Navidrome Music Server

Navidrome is a lightweight, self-hosted music streaming server compatible with Subsonic clients.

In my setup, Navidrome runs inside the Debian chroot as the primary music streaming service.

- Runs directly on the system (no Docker)
- Music files are stored on the local filesystem
- Accessible from anywhere using Tailscale

Detailed installation steps are not included here to keep this repository focused on the overall server setup.

For full installation and configuration instructions, refer to the official documentation:
https://www.navidrome.org/docs/installation/

## Tailscale

Remote access is provided via Tailscale installed directly on the Android device.

The Debian chroot itself does not run Tailscale. All services inside the chroot are accessed through the Android host’s Tailscale network.

## Web Hosting (Nginx + Cloudflare)

This setup also hosts my personal website inside the Debian chroot, using Nginx as the web server.

Nginx runs inside the Debian chroot and serves the website over standard HTTP. The web server listens on a local port and does not expose itself directly to the public internet.

Cloudflare is used in front of the server for DNS management and reverse proxying. Incoming traffic is handled by Cloudflare and forwarded to the Nginx service running locally.

- Nginx runs inside the Debian chroot
- The website is served as a standard HTTP service
- Cloudflare provides DNS and proxy features

This approach keeps the server simple while still allowing the website to be publicly accessible.

## File Browser

I also use File Browser as a lightweight web-based file management interface running inside the Debian chroot.

In this setup, File Browser is used as a personal cloud storage solution, allowing access to files from anywhere via Tailscale.

- Runs inside the Debian chroot
- Accessible from anywhere using Tailscale
- Used for browsing, uploading, and managing files
- Files are stored on the local filesystem

Detailed installation steps are not included to keep the repository focused on the overall server setup.

For more information, see the official File Browser documentation:
https://filebrowser.org/

---

## Notes

- Disable battery optimizations for Termux
- Keep the device plugged in if possible
- Ideal for personal use and learning
- More services can be added later

---

## License

MIT License
