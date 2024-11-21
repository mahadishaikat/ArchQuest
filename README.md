# Arch Linux Manual Installation Guide

This guide explains the Arch Linux installation process step-by-step from scratch. It includes an index for navigation to desired sections.

## Index
1. [Connecting to the Internet](#connecting-to-the-internet)
2. [Partitioning the Disk](#partitioning-the-disk)
3. [Setting up LVM](#setting-up-lvm)
4. [Formatting Partitions](#formatting-partitions)
5. [Mounting Partitions](#mounting-partitions)

---

### Connecting to the Internet

| Command                                                                                 | Explanation                               | Why Doing This                                    |
|-----------------------------------------------------------------------------------------|-------------------------------------------|--------------------------------------------------|
| `ip addr show`                                                                          | Shows current network interfaces          | To verify the system recognizes network devices  |
| `iwctl --passphrase "**" station wlan0 connect Your_Wifi_Name`                          | Connect to Wi-Fi using iwctl              | To set up an internet connection in the live session |
| `ip addr show`                                                                          | Shows current IP addresses                | To check if an IP address is assigned            |
| `ping google.com`                                                                       | Sends packets to check connectivity       | To confirm internet connection is working        |

---

### Partitioning the Disk

| Command                        | Explanation                                   | Why Doing This                                       |
|--------------------------------|-----------------------------------------------|-----------------------------------------------------|
| `lsblk`                        | Lists block devices                          | To identify available drives for partitioning       |
| `fdisk /dev/nvme0n1`           | Opens fdisk for partitioning                 | To create a new partition table                    |
| `p`                            | Prints the current partition table           | To view existing partitions before creating new ones |
| `g`                            | Creates a new GPT partition table            | To start with a clean partition layout             |
| `n`                            | Creates a new partition                      | To allocate space for system partitions            |
| Defaults, +1G                  | Sets default values and allocates 1GB space  | To define the size of the partition                |
| Repeat `n` for additional partitions | Repeats the process to create more partitions | For boot, root, and other necessary partitions     |
| `t`                            | Changes partition type                       | To set the Linux LVM partition type                |
| `w`                            | Writes changes to the disk                   | To finalize the partition table                    |

---

### Setting up LVM

| Command                                      | Explanation                                   | Why Doing This                                       |
|----------------------------------------------|-----------------------------------------------|-----------------------------------------------------|
| `mkfs.fat -F32 /dev/nvme0n1p1`               | Formats the first partition as FAT32          | For the EFI system partition (bootloader)           |
| `cryptsetup luksFormat /dev/nvme0n1p3`       | Encrypts the third partition with LUKS        | To secure data on the partition                     |
| `cryptsetup open --type luks /dev/nvme0n1p3 lvm*` | Opens the encrypted partition with a name     | To prepare for LVM setup                            |
| `pvcreate /dev/mapper/lvm*`                  | Creates a physical volume for LVM             | To initialize the encrypted partition for LVM       |
| `vgcreate volgroup0 /dev/mapper/lvm*`        | Creates a volume group named `volgroup0`      | To group physical volumes into a single entity      |
| `lvcreate -L 30GB volgroup0 -n lv_root`      | Creates a 30GB logical volume for root        | To allocate space for the root filesystem           |
| `lvcreate -L 250GB volgroup0 -n lv_home`     | Creates a 250GB logical volume for home       | To allocate space for user data                     |
| `vgdisplay` and `lvdisplay`                  | Displays volume group and logical volume info | To verify the LVM setup                             |

---

### Formatting Partitions

| Command                                  | Explanation                                   | Why Doing This                                       |
|------------------------------------------|-----------------------------------------------|-----------------------------------------------------|
| `mkfs.ext4 /dev/volgroup0/lv_root`       | Formats the root logical volume as ext4       | To prepare the root partition for the filesystem    |
| `mkfs.ext4 /dev/volgroup0/lv_home`       | Formats the home logical volume as ext4       | To prepare the home partition for the filesystem    |

---

### Mounting Partitions

| Command                                  | Explanation                                   | Why Doing This                                       |
|------------------------------------------|-----------------------------------------------|-----------------------------------------------------|
| `mount /dev/volgroup0/lv_root /mnt`      | Mounts the root partition to `/mnt`           | To prepare the root filesystem for installation     |
| `mkdir /mnt/boot`                        | Creates a boot directory in `/mnt`            | To organize the bootloader files                   |
| `mount /dev/nvme0n1p2 /mnt/boot`         | Mounts the boot partition                    | To prepare the boot partition                      |
| `mkdir /mnt/home`                        | Creates a home directory in `/mnt`            | To organize user data                               |
| `mount /dev/volgroup0/lv_home /mnt/home` | Mounts the home partition                     | To finalize the partition mounting structure        |

---

Continue completing this guide for the remaining steps!
