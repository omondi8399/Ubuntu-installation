# Ubuntu Desktop Installation & Setup Guide

A complete reference for migrating from Windows 11 to Ubuntu Desktop and setting up a developer environment.

---

## Table of Contents

1. [Phase 1 — Back Up Your Files](#phase-1--back-up-your-files)
2. [Phase 2 — Create Ubuntu Bootable USB](#phase-2--create-ubuntu-bootable-usb)
3. [Phase 3 — Boot & Install Ubuntu](#phase-3--boot--install-ubuntu)
4. [Phase 4 — Restore Files & Initial Setup](#phase-4--restore-files--initial-setup)
5. [Git & GitHub SSH Setup](#git--github-ssh-setup)
6. [Developer Tools Installation](#developer-tools-installation)
7. [Troubleshooting](#troubleshooting)

---

## Phase 1 — Back Up Your Files

Before doing anything, back up everything to an external drive or cloud storage.

**Files to back up:**
- `Documents`, `Downloads`, `Desktop`, `Pictures`, `Videos`
- Project folders and repositories
- `.ssh` keys and `.gitconfig`
- Browser bookmarks and passwords (export from browser)

> **Important:** Deactivate any licensed software (Adobe, Office, etc.) before wiping — some licenses are device-locked.

---

## Phase 2 — Create Ubuntu Bootable USB

### Download Ubuntu Desktop ISO

Go to [ubuntu.com/download/desktop](https://ubuntu.com/download/desktop) and download:

- **Intel or AMD 64-bit architecture** (for all standard laptops and desktops)
- Choose the latest **LTS version** (recommended for stability)

> Do NOT download Ubuntu Server — it has no graphical interface.

### Flash to USB with Rufus

1. Download **Rufus** from [rufus.ie](https://rufus.ie) — use `rufus-4.x.exe` (Standard Windows x64)
2. Plug in a USB drive (8GB minimum)
3. Open Rufus:
   - **Device** → select your USB drive
   - **Boot selection** → select the Ubuntu ISO
   - **Partition scheme** → `GPT` (for modern UEFI machines)
   - Leave all other settings at default
4. Click **Start**

---

## Phase 3 — Boot & Install Ubuntu

### Enter the Boot Menu

1. Keep the USB plugged in
2. Restart your machine
3. Immediately spam **F12** (Lenovo) as the screen lights up
4. Select your USB drive from the boot menu (shown as "USB HDD" or USB brand name)

> **BIOS note:** If the USB doesn't appear, enter BIOS (`F2` or `Del`), disable **Secure Boot**, and set USB as first boot device.

### Install Ubuntu

1. Select **Try Ubuntu** first to verify hardware works (WiFi, display, touchpad)
2. Click the **Install Ubuntu** icon on the desktop
3. Choose **Interactive installation**
4. On the installation type screen, select **"Erase disk and install Ubuntu"**

   > Unplug your backup drive before confirming to avoid accidentally wiping it.

5. Choose **No encryption** (unless you specifically need it)
6. Set your name, username, and password
7. Let the installer complete → remove USB when prompted → reboot

---

## Phase 4 — Restore Files & Initial Setup

Once booted into Ubuntu, open Terminal (`Ctrl + Alt + T`):

```bash
# Update the system
sudo apt update && sudo apt upgrade -y

# Copy your files back from the external drive
# Mount via the Files app, then drag folders to your home directory
```

---

## Git & GitHub SSH Setup

### Step 1 — Install Git

```bash
sudo apt update && sudo apt install git -y
```

### Step 2 — Configure Git Identity

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
git config --global init.defaultBranch main
```

### Step 3 — Generate SSH Key

```bash
ssh-keygen -t ed25519 -C "you@example.com"
# Press Enter through all prompts
```

### Step 4 — Add Key to SSH Agent

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```

### Step 5 — Copy Public Key

```bash
cat ~/.ssh/id_ed25519.pub
```

Copy the entire output (starts with `ssh-ed25519`, ends with your email).

### Step 6 — Add to GitHub

Go to **GitHub → Settings → SSH and GPG keys → New SSH key** and paste the key.

### Step 7 — Test Connection

```bash
ssh -T git@github.com
# Expected: Hi YourUsername! You've successfully authenticated.
```

> Always use SSH URLs (`git@github.com:...`) not HTTPS for cloning.

---

## Developer Tools Installation

### VS Code

```bash
sudo apt install wget gpg -y
wget -qO- https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > packages.microsoft.gpg
sudo install -o root -g root -m 644 packages.microsoft.gpg /etc/apt/trusted.gpg.d/
sudo sh -c 'echo "deb [arch=amd64 signed-by=/etc/apt/trusted.gpg.d/packages.microsoft.gpg] https://packages.microsoft.com/repos/code stable main" > /etc/apt/sources.list.d/vscode.list'
sudo apt update
sudo apt install code -y
```

### Node.js (via nvm)

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
source ~/.bashrc
nvm install --lts
node -v
npm -v
```

### MongoDB Compass

```bash
wget https://downloads.mongodb.com/compass/mongodb-compass_1.42.2_amd64.deb
sudo dpkg -i mongodb-compass_1.42.2_amd64.deb
sudo apt install -f -y
rm mongodb-compass_1.42.2_amd64.deb
```

### PostgreSQL

```bash
sudo apt install postgresql postgresql-contrib -y
```

### pgAdmin 4 (via pipx)

```bash
sudo apt install pipx -y
pipx install pgadmin4
pipx ensurepath
source ~/.bashrc

# Fix permissions
sudo mkdir -p /var/lib/pgadmin
sudo mkdir -p /var/log/pgadmin
sudo chown -R $USER:$USER /var/lib/pgadmin
sudo chown -R $USER:$USER /var/log/pgadmin

# Launch (opens at http://127.0.0.1:5050)
pgadmin4
```

> **Tip:** Install pgAdmin as a desktop app via Chrome → open `http://127.0.0.1:5050` → click the install icon in the address bar.

### Postman

```bash
sudo snap install postman
```

### Google Chrome

```bash
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
sudo dpkg -i google-chrome-stable_current_amd64.deb
sudo apt install -f -y
rm google-chrome-stable_current_amd64.deb
```

---

## Troubleshooting

### SSH agent not running
```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```

### pgAdmin permission denied on `/var/lib/pgadmin`
```bash
sudo mkdir -p /var/lib/pgadmin /var/log/pgadmin
sudo chown -R $USER:$USER /var/lib/pgadmin /var/log/pgadmin
```

### pgAdmin install fails with Python dependency conflict
Use pipx instead of apt — it handles the virtual environment automatically:
```bash
sudo apt install pipx -y
pipx install pgadmin4
```

### WiFi not working after install (Broadcom chips)
Connect via ethernet first, then:
```bash
sudo apt install bcmwl-kernel-source -y
```

### No bootable device after wiping Windows
You likely removed the USB before booting. Plug it back in and restart, then complete the Ubuntu install.

---

*Last updated: March 2026 | Ubuntu 24.04 LTS (Noble Numbat)*
