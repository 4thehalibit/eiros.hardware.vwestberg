# Installing Eiros on the Framework 16

This guide walks through a full NixOS install on this machine and applying the Eiros configuration.

---

## Quick Start

1. Download the NixOS minimal ISO and flash it to a USB
2. Boot the USB on the Framework — press **F12** for the boot menu
3. Partition, format, and mount the NVMe drive
4. Run `nixos-generate-config --root /mnt` and push the generated `hardware-configuration.nix` to this repo, replacing the placeholder
5. Run the install command:
   ```bash
   sudo nixos-install --flake github:lcleveland/eiros#default \
     --override-input eiros_users github:4thehalibit/eiros.users.vwestberg \
     --override-input eiros_hardware github:4thehalibit/eiros.hardware.vwestberg
   ```
6. Reboot, log in as `vwestberg`, and change your password with `passwd`

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

# Mount
sudo mount /dev/disk/by-label/nixos /mnt
sudo mkdir -p /mnt/boot
sudo mount /dev/disk/by-label/boot /mnt/boot
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
git clone https://github.com/4thehalibit/eiros.hardware.vwestberg.git /tmp/eiros-hardware
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

## Step 5 — Install NixOS

```bash
sudo nixos-install --flake github:lcleveland/eiros#default \
  --override-input eiros_users github:4thehalibit/eiros.users.vwestberg \
  --override-input eiros_hardware github:4thehalibit/eiros.hardware.vwestberg
```

This will download and build the full Eiros system. It takes a while on first run.

Set the root password when prompted at the end.

---

## Step 6 — First Boot

```bash
sudo reboot
```

Remove the USB when the machine powers off.

Log in as `vwestberg` with the initial password: see `users/vwestberg/user.nix` in the users repo.

**Change your password immediately:**
```bash
passwd
```

---

## Step 7 — Post-Install

Re-apply the config after any changes to this repo or the users repo:

```bash
sudo nixos-rebuild switch --flake github:lcleveland/eiros#default \
  --override-input eiros_users github:4thehalibit/eiros.users.vwestberg \
  --override-input eiros_hardware github:4thehalibit/eiros.hardware.vwestberg
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
| [4thehalibit/eiros.hardware.vwestberg](https://github.com/4thehalibit/eiros.hardware.vwestberg) | This repo — hardware config |
| [4thehalibit/eiros.users.vwestberg](https://github.com/4thehalibit/eiros.users.vwestberg) | User config, apps, keybinds |
