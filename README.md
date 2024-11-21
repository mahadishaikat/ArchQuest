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
| 1    | `ip addr show`                                            | Shows current network interfaces          | To verify the system recognizes network devices  | Optional            |
| 2    | `iwctl --passphrase "**" station wlan0 connect Your_Wifi_Name` | Connect to Wi-Fi using iwctl              | To set up an internet connection in the live session | Mandatory           |
| 3    | `ip addr show`                                            | Shows current IP addresses                | To verify if an IP address is assigned           | Optional            |
| 4    | `ping google.com`                                         | Sends packets to check connectivity       | To verify internet connection is working         | Optional            |

---

### Partitioning the Disk

| Step | Command                        | Explanation                                   | Why Doing This                                       | Optional/Mandatory |
|------|--------------------------------|-----------------------------------------------|-----------------------------------------------------|---------------------|
| 1    | `lsblk`                        | Lists all available block devices             | To identify storage devices and their partitions    | Mandatory           |
| 2    | `fdisk /dev/nvme0n1`           | Opens fdisk to manage the selected device     | To start partitioning the target disk              | Mandatory           |
| 3    | `p`                            | Prints the current partition table            | To display existing partitions                     | Mandatory           |
| 4    | `g`                            | Creates a new GPT partition table             | To clear the old partition table and start fresh   | Mandatory           |
| 5    | `p`                            | Prints the updated partition table            | To confirm a new GPT table has been created        | Optional            |
| 6    | `n`                            | Starts creating a new partition               | To allocate space for system components            | Mandatory           |
| 7    | Defaults, `+1G`, `Y`           | Allocates 1GB for the EFI system partition    | To reserve space for the bootloader                | Mandatory           |
| 8    | `n`                            | Starts creating the next partition            | To allocate additional space for the system        | Mandatory           |
| 9    | Defaults, `+1G`, `Y`           | Allocates another 1GB for the boot partition  | To dedicate space for the bootloader               | Mandatory           |
| 10   | `p`                            | Prints the updated partition table            | To verify all created partitions                   | Optional            |

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
| 7    | `YES`                                        | Confirms the encryption process               | Required by LUKS to proceed                         | Mandatory           |
| 8    | Enter passphrase: `***`                     | Sets a passphrase for encryption              | To unlock the encrypted partition in the future     | Mandatory           |
| 9    | `cryptsetup open --type luks /dev/nvme0n1p3 lvm*` | Opens the encrypted partition                 | To unlock the encrypted partition for LVM setup     | Mandatory           |
| 10   | Enter passphrase: `***`                     | Unlocks the encrypted partition               | Required for LVM operations                         | Mandatory           |
| 11   | `pvcreate /dev/mapper/lvm*`                  | Creates a physical volume for LVM             | To initialize the encrypted partition for LVM       | Mandatory           |
| 12   | `vgcreate volgroup0 /dev/mapper/lvm*`        | Creates a volume group named `volgroup0`      | To organize physical volumes into a group          | Mandatory           |
| 13   | `lvcreate -L 30GB volgroup0 -n lv_root`      | Creates a 30GB logical volume for root        | To allocate space for the root filesystem           | Mandatory           |
| 14   | `lvcreate -L 250GB volgroup0 -n lv_home`     | Creates a 250GB logical volume for home       | To allocate space for user data                     | Mandatory           |
| 15   | `vgdisplay`                                  | Displays information about the volume group   | To verify the volume group creation                | Optional            |
| 16   | `lvdisplay`                                  | Displays information about logical volumes    | To verify logical volume creation                  | Optional            |

---

### Mounting Partitions

| Step | Command                                  | Explanation                                   | Why Doing This                                       | Optional/Mandatory |
|------|------------------------------------------|-----------------------------------------------|-----------------------------------------------------|---------------------|
| 1    | `modprobe dm_mod`                        | Loads the kernel module for device mapper     | To enable access to logical volumes                | Mandatory           |
| 2    | `vgscan`                                 | Scans for volume groups                       | To detect the created volume group                 | Mandatory           |
| 3    | `vgchange -ay`                           | Activates all volume groups                   | To make the volume group accessible                | Mandatory           |
| 4    | `mkfs.ext4 /dev/volgroup0/lv_root`       | Formats the root logical volume as ext4       | To prepare the root partition for the filesystem    | Mandatory           |
| 5    | `mkfs.ext4 /dev/volgroup0/lv_home`       | Formats the home logical volume as ext4       | To prepare the home partition for the filesystem    | Mandatory           |
| 6    | `mount /dev/volgroup0/lv_root /mnt`      | Mounts the root logical volume                | To prepare the root filesystem for installation     | Mandatory           |
| 7    | `mkdir /mnt/boot`                        | Creates a directory for the boot partition    | To prepare the bootloader installation             | Mandatory           |
| 8    | `mount /dev/nvme0n1p2 /mnt/boot`         | Mounts the boot partition                     | To integrate the bootloader into the system         | Mandatory           |
| 9    | `mkdir /mnt/home`                        | Creates a directory for the home partition    | To store user files in a separate logical volume    | Mandatory           |
| 10   | `mount /dev/volgroup0/lv_home /mnt/home` | Mounts the home logical volume                | To complete the filesystem structure               | Mandatory           |

---

---

### Installing Required Packages

| Step | Command                                           | Explanation                                            | Why Doing This                                      | Optional/Mandatory |
|------|---------------------------------------------------|--------------------------------------------------------|----------------------------------------------------|---------------------|
| 1    | `pacstrap -i /mnt base`                           | Installs the base system packages                      | To install the essential Arch Linux packages       | Mandatory           |
| 2    | `genfstab -U -P /mnt >> /mnt/etc/fstab`            | Generates the fstab file for mounting partitions at boot | Ensures partitions are mounted automatically       | Mandatory           |
| 3    | `cat /mnt/etc/fstab`                              | Displays the contents of the fstab file                | To verify the partitions are correctly configured   | Optional            |
| 4    | `arch-chroot /mnt`                                | Changes root to the newly installed system            | To begin configuring the system inside the chroot   | Mandatory           |
| 5    | `passwd`                                          | Sets the root password for the system                  | To secure the root account                         | Mandatory           |
| 6    | `useradd -m -g users -G wheel userName`            | Creates a new user with administrative privileges      | To add a normal user account                       | Mandatory           |
| 7    | `passwd userName`                                  | Sets the password for the newly created user           | To secure the normal user account                  | Mandatory           |

---

### Installing Additional Packages

| Step | Command                                               | Explanation                                               | Why Doing This                                      | Optional/Mandatory |
|------|-------------------------------------------------------|-----------------------------------------------------------|----------------------------------------------------|---------------------|
| 1    | `pacman -S base-devel dosfstools grub efibootmgr gnome gnome-tweaks lvm2 mtools nano networkmanager openssh os-prober sudo` | Installs essential utilities and desktop environment packages | To prepare the system for basic functionality, GUI, and dual booting | Mandatory           |
| 2    | `pacman -S linux linux-headers linux-lts linux-lts-headers` | Installs the Linux kernel and headers                      | Provides the core Linux kernel for system operation | Mandatory           |
| 3    | `pacman -S linux-firmware`                             | Installs necessary firmware for hardware devices           | Needed for supporting proprietary hardware         | Optional            |
| 4    | `pacman -S mesa`                                      | Installs graphics drivers for Intel and AMD GPUs           | For proper GPU support on Intel/AMD devices         | Optional            |
| 5    | `pacman -S nvidia nvidia-utils nvidia-lts`             | Installs NVIDIA drivers for gaming and general GPU use     | Required for NVIDIA GPU support                    | Optional            |
| 6    | `pacman -S intel-media-driver`                        | Installs Intel GPU hardware acceleration drivers           | For Intel GPU hardware decoding support            | Optional            |
| 7    | `pacman -S libva-mesa-driver`                         | Installs AMD GPU hardware acceleration drivers             | For AMD hardware decoding                          | Optional            |

---

### Configuring the System

| Step | Command                                       | Explanation                                            | Why Doing This                                      | Optional/Mandatory |
|------|-----------------------------------------------|--------------------------------------------------------|----------------------------------------------------|---------------------|
| 1    | `nano /etc/mkinitcpio.conf`                   | Opens the initcpio configuration file                   | To configure kernel parameters                     | Mandatory           |
| 2    | Add `"encrypt lvm2"` between `"block"` and `"filesystem"` | Modifies the mkinitcpio hooks line                      | Enables encryption and LVM support during boot      | Mandatory           |
| 3    | `mkinitcpio -p linux`                         | Generates the initial ramdisk for the Linux kernel      | Prepares the kernel with encryption and LVM support | Mandatory           |
| 4    | `mkinitcpio -p linux-lts`                     | Generates the initial ramdisk for the LTS kernel        | Prepares the LTS kernel with encryption and LVM    | Optional            |

---

### Locale and Timezone Configuration

| Step | Command                                       | Explanation                                            | Why Doing This                                      | Optional/Mandatory |
|------|-----------------------------------------------|--------------------------------------------------------|----------------------------------------------------|---------------------|
| 1    | `nano /etc/locale.gen`                        | Opens the locale configuration file                      | To set the system locale                           | Mandatory           |
| 2    | Uncomment `en_US.UTF-8`                        | Enables US English locale                               | Sets the system language to English                 | Mandatory           |
| 3    | `locale-gen`                                   | Generates the locale data                                | Ensures the system uses the selected locale         | Mandatory           |
| 4    | `ln -sf /usr/share/zoneinfo/Your/Timezone /etc/localtime` | Sets the system timezone                               | Ensures the system uses the correct time zone       | Mandatory           |
| 5    | `hwclock --systohc`                           | Synchronizes hardware clock with system time            | Sets the hardware clock to match the system time    | Mandatory           |

---

### Setting Up GRUB and Bootloader

| Step | Command                                                   | Explanation                                            | Why Doing This                                      | Optional/Mandatory |
|------|-----------------------------------------------------------|--------------------------------------------------------|----------------------------------------------------|---------------------|
| 1    | `nano /etc/default/grub`                                  | Opens the GRUB configuration file                       | To configure GRUB bootloader settings              | Mandatory           |
| 2    | Add `cryptdevice=/dev/nvme0n1p3:volgroup0` to the `GRUB_CMDLINE_LINUX_DEFAULT` line | Sets up disk encryption in GRUB                        | Ensures that the encrypted partition is unlocked at boot | Mandatory           |
| 3    | `mkdir /boot/EFI`                                         | Creates the EFI directory                               | Prepares for EFI bootloader installation           | Mandatory           |
| 4    | `mount /dev/nvme0n1p1 /boot/EFI`                          | Mounts the EFI partition                                | Mounts the partition for the bootloader            | Mandatory           |
| 5    | `grub-install --target=x86_64-efi --bootloader-id=grub_uefi --recheck` | Installs GRUB bootloader for UEFI systems               | Installs GRUB to the EFI partition                 | Mandatory           |
| 6    | `cp /usr/share/locale/en@quot/LC_MESSAGES/grub.mo /boot/grub/locale/en.mo` | Copies GRUB locale files to boot partition              | Ensures correct language support in GRUB            | Mandatory           |
| 7    | `grub-mkconfig -o /boot/grub/grub.cfg`                    | Generates GRUB configuration file                       | Creates the GRUB boot menu                          | Mandatory           |

---

### Finalizing Setup

| Step | Command                                | Explanation                                           | Why Doing This                                      | Optional/Mandatory |
|------|----------------------------------------|-------------------------------------------------------|----------------------------------------------------|---------------------|
| 1    | `systemctl enable gdm`                 | Enables the GNOME Display Manager (GDM)                | To start the graphical interface automatically      | Mandatory           |
| 2    | `systemctl enable NetworkManager`      | Enables NetworkManager for networking support          | Ensures network connectivity after reboot           | Mandatory           |
| 3    | `exit`                                 | Exits the chroot environment                           | To leave the chroot session                        | Mandatory           |
| 4    | `umount -a`                            | Unmounts all mounted partitions                        | To safely unmount all partitions before reboot      | Mandatory           |
| 5    | `reboot`                               | Reboots the system                                     | To restart the system and boot into Arch Linux      | Mandatory           |

---

### Post-Boot Configuration

| Step | Command                                      | Explanation                                           | Why Doing This                                      | Optional/Mandatory |
|------|----------------------------------------------|-------------------------------------------------------|----------------------------------------------------|---------------------|
| 1    | Check Wi-Fi functionality and reconnect      | Ensure your Wi-Fi is working and connected            | To make sure you have an active internet connection | Mandatory           |
| 2    | Go to Settings -> Region and Languages -> Language | Change language to English if necessary                | To set the preferred system language               | Optional            |

---

Congratulations! You have successfully installed Arch Linux. Enjoy your fully customized and functional Arch system!
