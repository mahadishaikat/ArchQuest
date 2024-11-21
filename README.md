# Arch Linux Manual Installation Guide

This guide explains the Arch Linux installation process step-by-step for beginners. Every step is detailed, and repeated actions are fully explained to avoid confusion.

## Index
1. [Connecting to the Internet](#connecting-to-the-internet)
2. [Partitioning the Disk](#partitioning-the-disk)
3. [Setting up LVM](#setting-up-lvm)
4. [Formatting Partitions](#formatting-partitions)
5. [Mounting Partitions](#mounting-partitions)

---

### Connecting to the Internet

| Step | Command                                                   | Explanation                               | Why Doing This                                    | Optional/Mandatory |
|------|-----------------------------------------------------------|-------------------------------------------|--------------------------------------------------|---------------------|
| 1    | `ip addr show`                                            | Shows current network interfaces          | To verify the system recognizes network devices  | Mandatory           |
| 2    | `iwctl --passphrase "**" station wlan0 connect Your_Wifi_Name` | Connect to Wi-Fi using iwctl              | To set up an internet connection in the live session | Mandatory           |
| 3    | `ip addr show`                                            | Shows current IP addresses                | To check if an IP address is assigned            | Mandatory           |
| 4    | `ping google.com`                                         | Sends packets to check connectivity       | To confirm internet connection is working        | Optional            |

---

### Partitioning the Disk

| Step | Command                        | Explanation                                   | Why Doing This                                       | Optional/Mandatory |
|------|--------------------------------|-----------------------------------------------|-----------------------------------------------------|---------------------|
| 1    | `lsblk`                        | Lists all available block devices             | To identify storage devices and their partitions    | Mandatory           |
| 2    | `fdisk /dev/nvme0n1`           | Opens fdisk to manage the selected device     | To start partitioning the target disk              | Mandatory           |
| 3    | `p`                            | Prints the current partition table            | To display existing partitions                     | Mandatory           |
| 4    | `g`                            | Creates a new GPT partition table             | To clear the old partition table and start fresh   | Mandatory           |
| 5    | `p`                            | Prints the updated partition table            | To confirm a new GPT table has been created        | Mandatory           |
| 6    | `n`                            | Starts creating a new partition               | To allocate space for system components            | Mandatory           |
| 7    | Defaults, `+1G`, `Y`           | Allocates 1GB for the first partition         | To reserve space for the EFI system partition      | Mandatory           |
| 8    | `n`                            | Starts creating the next partition            | To allocate additional space for the system        | Mandatory           |
| 9    | Defaults, `+1G`, `Y`           | Allocates another 1GB for the boot partition  | To dedicate space for the bootloader               | Mandatory           |
| 10   | `p`                            | Prints the updated partition table            | To verify all created partitions                   | Mandatory           |

---

### Setting up LVM

| Step | Command                                      | Explanation                                   | Why Doing This                                       | Optional/Mandatory |
|------|----------------------------------------------|-----------------------------------------------|-----------------------------------------------------|---------------------|
| 1    | `n`                                          | Starts creating another partition             | To allocate space for the LVM partition             | Mandatory           |
| 2    | Defaults, `Y`                                | Uses all remaining space for the LVM          | To maximize storage usage for logical volumes       | Mandatory           |
| 3    | `t`                                          | Changes the type of the new partition         | To specify that this partition is for LVM           | Mandatory           |
| 4    | Partition Number, `44`                      | Selects the partition and sets type to `Linux LVM` | To prepare the partition for LVM setup            | Mandatory           |
| 5    | `w`                                          | Writes changes to the disk                    | To finalize the partition table                    | Mandatory           |
| 6    | `cryptsetup luksFormat /dev/nvme0n1p3`       | Encrypts the partition using LUKS             | To secure the data on this partition                | Optional            |
| 7    | `cryptsetup open --type luks /dev/nvme0n1p3 lvm*` | Opens the encrypted partition                 | To unlock the encrypted partition for LVM setup     | Optional            |
| 8    | `pvcreate /dev/mapper/lvm*`                  | Creates a physical volume for LVM             | To initialize the encrypted partition for LVM       | Mandatory           |
| 9    | `vgcreate volgroup0 /dev/mapper/lvm*`        | Creates a volume group named `volgroup0`      | To organize physical volumes into a group          | Mandatory           |
| 10   | `lvcreate -L 30GB volgroup0 -n lv_root`      | Creates a 30GB logical volume for root        | To allocate space for the root filesystem           | Mandatory           |
| 11   | `lvcreate -L 250GB volgroup0 -n lv_home`     | Creates a 250GB logical volume for home       | To allocate space for user data                     | Mandatory           |

---

### Formatting Partitions

| Step | Command                                  | Explanation                                   | Why Doing This                                       | Optional/Mandatory |
|------|------------------------------------------|-----------------------------------------------|-----------------------------------------------------|---------------------|
| 1    | `mkfs.fat -F32 /dev/nvme0n1p1`           | Formats the first partition as FAT32          | For the EFI system partition (bootloader)           | Mandatory           |
| 2    | `mkfs.ext4 /dev/volgroup0/lv_root`       | Formats the root logical volume as ext4       | To prepare the root partition for the filesystem    | Mandatory           |
| 3    | `mkfs.ext4 /dev/volgroup0/lv_home`       | Formats the home logical volume as ext4       | To prepare the home partition for the filesystem    | Mandatory           |

---

Let me know how else you'd like to refine this guide!
