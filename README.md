# Android Home Server (Debian Chroot)

A lightweight, low-power server environment leveraging a native Debian 13 (Trixie) chroot on rooted Android hardware. This system is designed for 24/7 uptime, providing a near-native Linux experience without the overhead of proot.

## System Architecture

* **Runtime:** Debian 13 (Trixie) via native `chroot`
* **Access:** Tailscale (Mesh VPN) & Cloudflare (Edge Proxy)
* **Storage:** External HDD integration via bind-mounts
* **Hardware:** Rooted ARM64 Android device

## Quick Start

1. Clone the Repository:
```bash
git clone https://github.com/username/android-home-server.git
cd android-home-server
```
2. Environment Preparation:
Install dependencies and ensure you have a Debian RootFS (tar.xz) in the root directory.
```bash
pkg update && pkg install wget tar -y
```
3. Deploy & Enter:
Grant execution permissions and run the entry script as root:
```bash
chmod +x start.sh
su -c ./start.sh
```
## Repository Structure

* start.sh: The core entry point. Handles pseudo-filesystem mounts (/proc, /sys, /dev) and enters the chroot environment.
* configs/: Directory for Nginx and service configurations.
* scripts/: Helper scripts for automated backups and mount management.

## Technical Details

### The Entry Script (start.sh)
The provided start.sh automates the following critical tasks:
* Cleanup: Unmounts stale points to prevent mount-clash errors.
* Mounting: Binds host /dev, /proc, and /sys to the chroot tree.
* DNS Sync: Syncs host DNS settings to the chroot for immediate network access.
* Execution: Drops the user into a login shell within the Debian environment.

### External Storage
To persist data on an external drive, use the bind-mount method:
mount --bind /storage/XXXX-XXXX /data/debian/debian-rootfs/mnt/hdd

## Service Stack

### Navidrome & File Browser
Lightweight media and file management services. Best accessed via the Android host's Tailscale IP to ensure encrypted, firewall-traversing connections.

### Nginx + Cloudflare
The recommended stack for public-facing elements. Nginx acts as the local listener, while Cloudflare Tunnels provide a secure bridge to the public internet without port forwarding.

## Maintenance & Security
* WakeLock: Enable "Acquire WakeLock" in Termux to prevent CPU sleep.
* Networking: To allow non-root users to access the internet, they must be added to the AID_INET group (GID 3003).
* Persistence: Services inside chroot do not start automatically on boot. Use a Magisk boot script to call start.sh on device startup.

## License
MIT
