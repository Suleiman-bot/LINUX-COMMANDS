#!/bin/bash
# ==========================================================
# DEVLOG REFERENCE FILE
# Author: Abdulsalam Suleiman
# Description: Consolidated Development Operations Log & Reference
# Date generated: 2025-10-08
# ==========================================================

# ----------------------------------------------------------
# [2025-08-23] DEVLOG Initialization
# ----------------------------------------------------------
# Setup of DEVLOG for documenting troubleshooting steps, fixes,
# and operational references across environments.
# ----------------------------------------------------------

# No commands executed — initial setup entry.

# Notes:
# This file is used to track commands, issues, and resolutions to
# prevent repetition and streamline debugging workflow.

# ==========================================================
# [2025-08-20] VirtualBox Autostart Configuration
# ==========================================================

# Attempt to enable autostart for VM "testServer"
VBoxManage modifyvm "testServer" --autostart-enabled on
# → Failed: VM name not found. Verified names below.

VBoxManage list vms
# → Used to confirm the exact VM name before retrying autostart.

# Retry using VM name and UUID (failed)
sudo VBoxManage modifyvm "testServer {5f0d8390-a982-4a85-b4a3-e457ec19e1a6}" --autostart-enabled on

# Correct approach: Use *exact* VM name as shown in VBoxManage list
VBoxManage modifyvm "<exact_vm_name>" --autostart-enabled on

# Set global autostart database path
VBoxManage setproperty autostartdbpath /etc/vbox

# Enable and start VirtualBox autostart service
sudo systemctl enable vboxautostart-service
sudo systemctl start vboxautostart-service

# Define VM autostop behavior (preserve state)
VBoxManage modifyvm "<exact_vm_name>" --autostop-type savestate

# ==========================================================
# [2025-08-20] Network Configuration (VM)
# ==========================================================

# Display all interfaces and IPs
ip a

# Display routing table and default gateway
ip r

# Apply and validate Netplan configuration
sudo netplan generate
sudo netplan apply

# Example Netplan YAML for static IP
# /etc/netplan/01-netcfg.yaml
# ---------------------------
# network:
#   version: 2
#   renderer: networkd
#   ethernets:
#     enp0s3:
#       dhcp4: no
#       addresses: [192.168.1.50/24]
#       gateway4: 192.168.1.1
#       nameservers:
#         addresses: [8.8.8.8, 8.8.4.4]

# ==========================================================
# [2025-08-20] SSH Setup on VM
# ==========================================================

sudo apt update
sudo apt install openssh-server -y
sudo systemctl enable ssh
sudo systemctl start ssh

# Test remote SSH access
ssh user@192.168.1.50

# ==========================================================
# [2025-08-19] WSL & Windows Environment Troubleshooting
# ==========================================================

# Install WSL
wsl --install

# Repair and verify WSL binaries
Get-FileHash "C:\Windows\System32\wsl.exe" -Algorithm SHA256
wsl.exe /repair

# Install via winget if repair fails
winget install --id Microsoft.WSL
winget install --id Microsoft.WSL --source winget --accept-package-agreements --accept-source-agreements

# Verify and manage winget sources
winget source list
winget --version

# Run system integrity check
sfc /scannow

# Check WSL status and installed distros
wsl --status
wsl -l -v

# Install and launch Ubuntu distribution
wsl --install -d Ubuntu-22.04
wsl -d Ubuntu

# Navigate into project folder
cd ~
cd ~/plane

# ==========================================================
# [2025-08-19] GitHub Configuration & Fixes
# ==========================================================

# Initialize and configure Git identity
git init
git branch -m main
git config --global user.name "Suleiman Bot"
git config --global user.email "abdulsalamsuleiman100@gmail.com"

# Check and set correct remote URL
git remote -v
git remote set-url origin https://github.com/Suleiman-bot/plane.git

# Push to remote (using PAT instead of password)
git push -u origin main

# Store credentials
git config --global credential.helper cache   # temporary
# or
git config --global credential.helper store   # permanent

# Routine workflow
git status
git add DEVLOG.md
git commit -m "update DEVLOG.md with new entries"
git push

# Pull updates from remote
git pull

# ==========================================================
# [2025-08-19] Docker & Plane Project Operations
# ==========================================================

# Move backend Dockerfile into correct directory
mv dockerfile.node /plane/api/

# Check backend health and endpoints
curl http://localhost:8000/api/health
curl http://localhost:8000/api/tickets
curl http://localhost:8000/api/tickets/12345

# Clean and rebuild Docker environment
docker compose down
docker compose -f docker-compose.override.yml down -v
docker compose -f docker-compose.override.yml up --build

# Create empty .env to suppress warnings
touch /plane/.env

# ==========================================================
# [2025-08-22] Docker Service Integration with Systemd
# ==========================================================

# Reload and restart service
sudo systemctl daemon-reload
sudo systemctl restart ticketing-docker
sudo systemctl status ticketing-docker

# Enable auto-start at boot
sudo systemctl enable ticketing-docker

# Verify directory path used by service
ls -ld /home/kasi/plane   # Found incorrect path → corrected to /plane

# Restart service after path fix
sudo systemctl restart ticketing-docker

# ==========================================================
# [2025-08-23] React Frontend Build & Serve
# ==========================================================

# Build production React app
npm run build

# Install and run local static server
npm install -g serve         # (may require sudo)
npx serve -s build -l 3000   # Run build locally on port 3000

# Test in browser:
# http://localhost:3000
# http://192.168.0.3:3000

# ==========================================================
# [2025-08-25] Caddy Proxy Frontend Access Tests
# ==========================================================

# Access frontend through proxy
# Test URLs after replacing Caddyfile:
#   - http://localhost/frontend/
#   - http://localhost/

# ==========================================================
# [2025-08-19 → 2025-08-23] Git Submodule Operations
# ==========================================================

# Submodule update & sync helper script
# --------------------------------------
# Pull updates for both outer repo and submodule
git pull
cd projects/ticketing-form && git pull && cd ..

# If submodule reference changed:
git status
git add projects/ticketing-form
git commit -m "Updated submodule reference to latest commit"
git push

# Commit and push changes inside submodule
cd projects/ticketing-form
git add .
git commit -m "Update ticketing-form submodule"
git push
cd ../..

# Update submodule reference in parent repo
git add projects/ticketing-form
git commit -m "Updated submodule to latest commit"
git push

# Sync latest commit from remote submodule
git submodule update --remote --merge

# Configure tracked branch for submodule
git fetch
git branch -r
git config -f .gitmodules submodule.projects/ticketing-form.branch main
git submodule update --remote --merge

# Absorb submodule into main repo (remove linkage)
cd projects/ticketing-form
rm -rf .git
cd ../../
git rm --cached -r projects/ticketing-form
git add projects/ticketing-form
git commit -m "Absorbed submodule into main repo as regular files"

# Shortcut alias (custom)
git subup

# ==========================================================
# [2025-08-23] Utility Commands
# ==========================================================

# Copy directory
cp -r /home/user/folder1 /home/user/backup/

# Verbose copy
cp -rv /home/user/folder1 /home/user/backup/

# Copy only newer or missing files
cp -ru /home/user/folder1 /home/user/backup/

# ==========================================================
# [2025-08-27] MongoDB 6.0 Installation Reference
# ==========================================================

# MongoDB Installation Script for Ubuntu 22.04 (Jammy)
# ----------------------------------------------------
curl -fsSL https://pgp.mongodb.com/server-6.0.asc | \
    sudo gpg -o /usr/share/keyrings/mongodb-server-6.0.gpg --dearmor

echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-6.0.gpg ] \
https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/6.0 multiverse" | \
    sudo tee /etc/apt/sources.list.d/mongodb-org-6.0.list

sudo apt update
sudo apt install -y mongodb-org
sudo systemctl start mongod
sudo systemctl enable mongod
systemctl status mongod

# Mongo Shell Quick Reference
mongosh mongodb://localhost:27017
mongosh mongodb://192.168.0.3:27017
show dbs
use plane
show collections
db.tickets.find().pretty()

# Show all network interfaces and their statuses
ip link

# List all Netplan configuration files
ls /etc/netplan

# Open the default network configuration file created during OS install
sudo nano /etc/netplan/00-installer-config.yaml

# Apply any changes made to Netplan configuration files
sudo netplan apply

# Open the cloud-init-generated network config (used in cloud/VM environments)
sudo nano /etc/netplan/50-cloud-init.yaml

# Open a custom or manually created Netplan configuration file
sudo nano /etc/netplan/01-netcfg.yaml

# List all PCI devices with numeric IDs — includes NICs, GPUs, RAID controllers, etc.
lspci -nn

# Show detailed information about all network interfaces — MAC address, speed, driver, etc.
sudo lshw -class network

#!/bin/bash

# Attempt to check GNS3 server config (failed: command not found)
gns3server --check-config
# Output: bash: gns3server: command not found

# Check which GNS3 packages are installed
dpkg -l | grep gns3
# Output example:
# ii  gns3-iou  0.0.3~focal1  amd64  GNS3 support for IOU

# Update package lists from repositories
sudo apt update

# Attempt to install GNS3 server, GUI, dynamips, qemu, kvm, utils
sudo apt install gns3-server gns3-gui dynamips qemu qemu-kvm qemu-utils
# Output: Error with qemu-kvm dependencies

# Check gns3server version (not found)
gns3server --version
# Output: bash: gns3server: command not found

# Navigate up one directory level
cd ..

# Change directory to root
cd /

# List root directory contents
ls
# Output: bin  dev  home  lib32  libx32  media  opt  root  sbin  swap.img  tmp  var  ...

# Change to /opt directory
cd opt

# List /opt contents
ls
# Output: containerd  docker  gns3  lost+found  VBoxGuestAdditions-7.0.26

# Change directory into /opt/gns3
cd gns3

# List contents of /opt/gns3
ls
# Output: images  projects

# Attempt to list detailed contents (wrong command 'la' used)
la
# Output: la: command not found

# Correctly list contents (shorter)
ls
# Output: images  projects

# Check OS release information
lsb_release -a
# Output example:
# Distributor ID: Ubuntu
# Description:    Ubuntu 20.04.6 LTS
# Release:        20.04
# Codename:       focal

# Check kernel version and system info
uname -a
# Output example:
# Linux gns3vm 5.15.0-136-generic #147~20.04.1-Ubuntu SMP Wed Mar 19 16:13:14 UTC 2025 x86_64 x86_64 x86_64 GNU/Linux

# Check disk usage on root partition
df -h /
# Output example:
# Filesystem      Size  Used Avail Use% Mounted on
# /dev/sda2        20G  5.0G   14G  28% /

# Navigate to Downloads directory
cd ~/Downloads

# List files in Downloads
ls
# Output example:
# anydesk_6.2.1-1_amd64.deb  GNS3.VM.VirtualBox.2.2.54  zoom_amd64.deb  ...

# Try to change directory to GNS3 VM folder (case sensitive error fixed)
cd GNS3.VM.VirtualBox.2.2.54

# List contents inside VM folder
ls
# Output:
# 'GNS3 VM.ova'

# Check if the .ova file was imported (implied by your text)

# Use find to locate gns3server binary
sudo find / -name gns3server 2>/dev/null
# Output example:
# /home/gns3/.venv/gns3server-venv/bin/gns3server
# /home/gns3/.venv/gns3server-venv/lib/python3.9/site-packages/gns3server

# Activate Python virtual environment containing gns3server
source /home/gns3/.venv/gns3server-venv/bin/activate

# Check gns3server version inside the virtualenv
gns3server --version
# Output:
# 2.2.54

# Try to list /opt/gns3/bin directory (does not exist)
ls /opt/gns3/bin/
# Output:
# ls: cannot access '/opt/gns3/bin/': No such file or directory


# Activate the GNS3 server Python virtual environment
# Use: prepares environment so gns3server command is available
source /home/gns3/.venv/gns3server-venv/bin/activate
# Expected prompt changes to: (gns3server-venv) gns3@gns3vm:~$

# Check GNS3 server version
# Use: verify gns3server is accessible and see version installed
gns3server --version
# Expected output:
# 2.2.54

# Try to check GNS3 server configuration (incorrect flag used)
# Use: attempt to verify configuration, but '--check-config' is invalid here
gns3server --check-config
# Expected output:
# usage: gns3server [-h] [-v] [--host HOST] [--port PORT] [--ssl] ...
# gns3server: error: unrecognized arguments: --check-config

# Check if the GNS3 server config file exists and view Qemu section if any
# Use: verify if Qemu section is configured in gns3_server.conf
cat ~/.config/GNS3/gns3_server.conf | grep -A2 '\[Qemu\]'
# Expected output if no Qemu section exists:
# cat: /home/gns3/.config/GNS3/gns3_server.conf: No such file or directory
# Or empty if file exists but no Qemu section

# Check where qemu-system-x86_64 binary is installed
# Use: verify QEMU is installed and available in PATH
which qemu-system-x86_64
# Expected output example:
# /usr/bin/qemu-system-x86_64

# Find exact location of gns3_server.conf (alternate location)
# Use: locate actual config file if default path missing
find /home/gns3/.config/GNS3 -name "gns3_server.conf" 2>/dev/null
# Expected output:
# /home/gns3/.config/GNS3/2.2/gns3_server.conf

# View contents of gns3_server.conf (to check current config)
# Use: inspect current config file (example shows no Qemu section)
cat /home/gns3/.config/GNS3/2.2/gns3_server.conf
# Expected output example:
# [Server]
# host = 0.0.0.0
# port = 80
# images_path = /opt/gns3/images
# projects_path = /opt/gns3/projects
# report_errors = True

# Move to root from any user
sudo -i

# ==========================================================
# END OF DEVLOG
# ==========================================================
