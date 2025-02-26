# Raspberry Pi NAS System Setup Guide

## Objective
The goal of this project is to build an affordable, low-power Network-Attached Storage (NAS) system using a Raspberry Pi 4. This NAS system is designed to share files securely across your home network using Samba (SMB) and can be accessed by Windows, macOS, and iOS devices.

## Description
This guide details how to turn a Raspberry Pi 4 into a fully functional NAS. It covers:
- Installing Raspberry Pi OS (Lite)
- Connecting and mounting an external USB storage drive
- Installing and configuring Samba for network file sharing
- Adjusting underlying filesystem permissions to ensure write access
- Troubleshooting common issues (e.g., read-only mounts and iOS Files app quirks)

This project is ideal for home users who want to centralize their file storage, create backups, and easily access data from multiple devices.

## Hardware and Software Used

### Hardware:
- **Raspberry Pi 4** (2GB minimum; 4GB or 8GB recommended)
- **MicroSD Card** (16GB or larger)
- **Official Raspberry Pi Power Supply**
- **External USB Storage** (HDD, SSD, or USB flash drive)
- **Ethernet Cable** (or Wi‑Fi connection)
- **Monitor, Keyboard & Mouse** (for initial setup; SSH can be used for headless configuration)

### Software:
- **Raspberry Pi OS Lite (64-bit)** – a lightweight OS ideal for server tasks
- **Samba** – open-source SMB implementation for file sharing
- **ntfs-3g** (if using an NTFS-formatted drive)
- *(Optional)* Third-party iOS file explorer apps (e.g., Documents by Readdle or FE File Explorer) if the native Files app has issues

## Step-by-Step Setup

### 1. Prepare Raspberry Pi OS
1. **Download Raspberry Pi Imager:**  
   Visit [Raspberry Pi Imager](https://www.raspberrypi.com/software/) and download the tool for your computer.

2. **Flash the MicroSD Card:**  
   Use the Imager to write **Raspberry Pi OS Lite (64-bit)** to your MicroSD card.

3. **Boot and Update:**  
   Insert the MicroSD card into your Raspberry Pi, connect your peripherals, and boot. Once logged in, open a terminal (or SSH) and run:
   ```bash
   sudo apt update && sudo apt upgrade -y
   ```

### 2. Connect and Mount External Storage
1. **Connect the Drive:**  
   Attach your external USB storage device to the Raspberry Pi.

2. **Identify the Drive:**  
   Run:
   ```bash
   lsblk
   ```
   Note the partition name (e.g., `/dev/sda1`).

3. **Create a Mount Point:**
   ```bash
   sudo mkdir -p /mnt/nas
   ```

4. **Mount the Drive:**  
   - For an ext4 filesystem:
     ```bash
     sudo mount /dev/sda1 /mnt/nas
     ```
   - For NTFS (after installing `ntfs-3g`):
     ```bash
     sudo apt install ntfs-3g -y
     sudo mount -t ntfs-3g /dev/sda1 /mnt/nas -o rw,uid=pi,gid=pi
     ```

### 3. Install and Configure Samba
1. **Install Samba Packages:**
   ```bash
   sudo apt install samba samba-common-bin -y
   ```

2. **Edit Samba Configuration:**  
   Open the configuration file:
   ```bash
   sudo nano /etc/samba/smb.conf
   ```
   In the **[global]** section, add:
   ```ini
   [global]
       vfs objects = catia fruit streams_xattr
       workgroup = WORKGROUP
       log file = /var/log/samba/log.%m
       max log size = 1000
       map to guest = bad user
       server role = standalone server
   ```

3. **Define Your NAS Share:**  
   Append the following to the end of the file (adjust the username as needed; here we use `vdave18`):
   ```ini
   [NAS]
       comment = Raspberry Pi NAS Share
       path = /mnt/nas
       browseable = yes
       read only = no
       writable = yes
       valid users = vdave18
       create mask = 0775
       directory mask = 0775
       force user = vdave18
   ```

4. **Restart Samba:**  
   Save the file (Ctrl+X, then Y, and Enter) and restart Samba:
   ```bash
   sudo systemctl restart smbd
   ```

### 4. Adjust Filesystem Permissions
1. **Check Current Permissions:**
   ```bash
   ls -ld /mnt/nas
   ```

2. **Set Ownership and Permissions:**  
   Ensure the directory is owned by your Samba user (`vdave18` in this example):
   ```bash
   sudo chown -R vdave18:vdave18 /mnt/nas
   sudo chmod -R 775 /mnt/nas
   ```

### 5. (Optional) Configure a Static IP Address
To avoid changes in IP addresses, you can assign a static IP:

1. **Edit DHCP Client Configuration:**
   ```bash
   sudo nano /etc/dhcpcd.conf
   ```

2. **Add Static IP Settings:**
   ```ini
   interface eth0
   static ip_address=192.168.1.100/24
   static routers=192.168.1.1
   static domain_name_servers=192.168.1.1 8.8.8.8
   ```

3. **Reboot:**
   ```bash
   sudo reboot
   ```

### 6. Access Your NAS
- **From Windows/macOS:**  
  Open your file explorer and navigate to:
  ```
  \\192.168.1.100\NAS
  ```
  or, if using a hostname:
  ```
  \\raspberrypi.local\NAS
  ```

- **From iOS Devices:**  
  1. Open the **Files app**.  
  2. Tap the three-dot icon (or “...”) and choose **Connect to Server**.  
  3. Enter the address:
     ```
     smb://raspberrypi.local/NAS
     ```
     or
     ```
     smb://192.168.1.100/NAS
     ```
  4. Log in using your Samba username (`vdave18`) and password.

## Troubleshooting and Cautions
- **Mount Status:**  
  Verify that the external drive is mounted with read/write access:
  ```bash
  mount | grep /mnt/nas
  ```
  If the drive is mounted as read-only (ro), adjust your mount options accordingly.

- **Filesystem Permissions:**  
  Temporarily set permissive rights to test:
  ```bash
  sudo chmod -R 777 /mnt/nas
  ```
  If this fixes the issue, review and set secure permissions (775) afterward.

- **Samba User:**  
  Ensure your Samba user is added:
  ```bash
  sudo smbpasswd -a vdave18
  ```

- **iOS Files App Issues:**  
  Some iOS versions (e.g., iOS 18 and later) have reported issues with SMB write access. If the Files app shows the share as read-only, try removing and re-adding the SMB connection or use an alternative app like **Documents by Readdle**.

- **Backup Configurations:**  
  Always back up your Samba configuration file (`/etc/samba/smb.conf`) before making changes.

## Conclusion and Learning Outcomes
- **Conclusion:**  
  You have now transformed your Raspberry Pi 4 into a fully functional NAS system. This system enables secure, cross-platform file sharing over your home network, simplifying file storage, backups, and remote access.
  
- **Learning Outcomes:**
  - **Hardware Setup:** Learned how to connect and mount external storage on the Raspberry Pi.
  - **Software Installation:** Gained experience installing Raspberry Pi OS and required packages.
  - **Samba Configuration:** Learned to configure Samba for file sharing and troubleshoot permission issues.
  - **Filesystem Management:** Understood the importance of proper ownership and permission settings.
  - **Networking:** Learned to assign a static IP for reliable network access.
  - **Troubleshooting:** Developed strategies to diagnose and resolve common NAS and client-access issues (especially on iOS).

## License
This project is licensed under the [MIT License](LICENSE).
```

---

Simply copy and paste the above content into your `README.md` file on GitHub. This detailed guide includes all the sections you requested and is ready for direct use.
