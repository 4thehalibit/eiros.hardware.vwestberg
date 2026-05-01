# Installing Eiros on the Framework 16

This guide walks through a full NixOS install on this machine and applying the Eiros configuration.

---

## Quick Start

1. Set your password hash in the [users repo](https://github.com/4thehalibit/eiros.users.personal) — see Step 5
2. Download the NixOS minimal ISO and flash it to a USB
3. Boot the USB on the Framework — press **F12** for the boot menu
4. Partition, format, and mount the NVMe drive
5. Run `nixos-generate-config --root /mnt` and push the generated `hardware-configuration.nix` to this repo, replacing the placeholder
6. Run the install command:
   ```bash
   sudo nixos-install --flake github:lcleveland/eiros#default \
     --override-input eiros_users github:4thehalibit/eiros.users.personal \
     --override-input eiros_hardware github:4thehalibit/eiros.hardware.framework16
   ```
7. Reboot and log in as `vwestberg`

> **Step 4 tip:** Pushing from the installer requires git and a GitHub token. If that's awkward, paste the file contents into the GitHub web editor from your phone or another device.

See the full steps below for details on each stage.

---

## What You Need

- NixOS minimal ISO on a USB drive ([download](https://nixos.org/download))
- Internet connection (ethernet recommended for install)
- This machine: Framework 16 / AMD Ryzen 7 7840HS

---

## Step 1 — Boot the Installer

1. Insert the USB and boot. Press **F12** on the Framework to open boot menu.
2. Select the NixOS USB.
3. At the shell, connect to ethernet or wifi:
   ```bash
   # Ethernet works automatically. For wifi:
   nmcli dev wifi connect "SSID" password "password"
   ```

---

## Step 2 — Partition and Format the Disk

The main NVMe drive is `/dev/nvme0n1` (1.8TB).

```bash
# Wipe and create a GPT partition table
sudo parted /dev/nvme0n1 -- mklabel gpt

# EFI partition (512MB)
sudo parted /dev/nvme0n1 -- mkpart ESP fat32 1MB 512MB
sudo parted /dev/nvme0n1 -- set 1 esp on

# Root partition (rest of disk)
sudo parted /dev/nvme0n1 -- mkpart primary 512MB 100%

# Format
sudo mkfs.fat -F 32 -n boot /dev/nvme0n1p1
sudo mkfs.ext4 -L nixos /dev/nvme0n1p2

# Verify labels exist before continuing
lsblk -f /dev/nvme0n1
# You should see:
# nvme0n1p1  vfat  FAT32  boot
# nvme0n1p2  ext4         nixos
# If either label is missing, DO NOT continue — rerun the mkfs commands above

# Mount using labels (not device names)
sudo mount /dev/disk/by-label/nixos /mnt
sudo mkdir -p /mnt/boot
sudo mount /dev/disk/by-label/boot /mnt/boot

# Verify mounts before continuing
mount | grep /mnt
# You should see both /mnt and /mnt/boot listed
# If not, DO NOT continue — troubleshoot the mounts first

```

---

## Step 3 — Generate Hardware Configuration

```bash
sudo nixos-generate-config --root /mnt
```

This creates `/mnt/etc/nixos/hardware-configuration.nix` with your disk UUIDs and kernel modules.

---

## Step 4 — Update This Repo

The `hardware-configuration.nix` in this repo is a placeholder. Replace it with the generated one.

From the installer (you need git and a GitHub token):

```bash
# Install git temporarily
nix-shell -p git

# Clone this repo
git clone https://github.com/4thehalibit/eiros.hardware.framework16.git /tmp/eiros-hardware
cd /tmp/eiros-hardware

# Replace the placeholder
cp /mnt/etc/nixos/hardware-configuration.nix ./hardware-configuration.nix

# Commit and push
git add hardware-configuration.nix
git commit -m "Add generated hardware-configuration.nix"
git push
```

> **Tip:** If pushing from the installer is difficult, copy the contents of `/mnt/etc/nixos/hardware-configuration.nix` and update the file on GitHub via the web editor before continuing.

---

## Step 5 — Set Your Password

Before installing, make sure your password hash is set in the users repo.

If you haven't done this yet, on another machine:

```bash
mkpasswd -m yescrypt
```

Paste the output into `users/vwestberg/user.nix` in the [users repo](https://github.com/4thehalibit/eiros.users.personal), replacing `REPLACE_WITH_HASH`. Commit and push. The hash is one-way and safe to store publicly.

See the [users repo README](https://github.com/4thehalibit/eiros.users.personal#first-time-setup) for full instructions.

---

## Step 6 — Install NixOS

```bash
sudo nixos-install --flake github:lcleveland/eiros#default \
  --override-input eiros_users github:4thehalibit/eiros.users.personal \
  --override-input eiros_hardware github:4thehalibit/eiros.hardware.framework16
```

This will download and build the full Eiros system. It takes a while on first run.

Set the root password when prompted at the end.

---

## Step 7 — First Boot

```bash
sudo reboot
```

Remove the USB when the machine powers off.

Log in as `vwestberg` using the password you set in Step 5.

---

## Step 8 — Post-Install

Re-apply the config after any changes to this repo or the users repo:

```bash
sudo nixos-rebuild switch --flake github:lcleveland/eiros#default \
  --override-input eiros_users github:4thehalibit/eiros.users.personal \
  --override-input eiros_hardware github:4thehalibit/eiros.hardware.framework16
```

Or use the `nh` tool (included with Eiros):
```bash
nh os switch
```

---

## Monitor Layout

Three displays are configured in `settings/monitors.nix`:

| Display | Resolution | Position | Notes |
|---------|-----------|----------|-------|
| eDP-1 | 2560×1600 @ 165Hz | Right | Framework built-in screen |
| DP-9 | 1920×1080 @ 60Hz | Bottom-left | External |
| DP-10 | 1920×1080 @ 60Hz | Top-left | External, rotated 180° |

Adjust `settings/monitors.nix` if your external monitors change.

---

## Repos

| Repo | Purpose |
|------|---------|
| [lcleveland/eiros](https://github.com/lcleveland/eiros) | Base Eiros framework |
| [4thehalibit/eiros.hardware.framework16](https://github.com/4thehalibit/eiros.hardware.framework16) | This repo — hardware config |
| [4thehalibit/eiros.users.personal](https://github.com/4thehalibit/eiros.users.personal) | User config, apps, keybinds |
