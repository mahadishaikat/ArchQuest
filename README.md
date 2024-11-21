# Arch Linux Installation Guide (Both Manual and Scripted)

This guide explains the Arch Linux installation process step-by-step for beginners. Every step is detailed, and repeated actions are fully explained to avoid confusion. Errors will be solved which I have faced with my `Nvidia RTX 3060` GPU, few tips will be shared for smooth installation.

## Index
**1. Manual Installation Walkthrough**
   - [Connecting to the Internet](#connecting-to-the-internet)
   - [Partitioning the Disk](#partitioning-the-disk)
   - [Setting up LVM](#setting-up-lvm)
   - [Formatting Partitions](#formatting-partitions)
   - [Mounting Partitions](#mounting-partitions)
   - [Installing Required Packages](#installing-required-packages)
   - [Setting Up Drivers for Video Card](#setting-up-drivers-for-video-card)
   - [Special Configuration for Intel and AMD GPUs](#special-configuration-for-intel-and-amd-gpus)
   - [Configuring mkinitcpio and Kernel](#configuring-mkinitcpio-and-kernel)
   - [Setting Locale and GRUB](#setting-locale-and-grub)

**2. Scripted Installation Walkthrough (Using ArchInstall Script)**
   - [Connecting to the Internet](#connecting-to-the-internet)
   - [Partitioning the Disk](#partitioning-the-disk)
   - [Setting up LVM](#setting-up-lvm)

---

### Manual Installation Walkthrough

The *legendary* path awaits! This is the **recommended way** for those fearless individuals who crave a deep dive into the core mechanics of how Linux truly operates. By following this journey, you won't just install Arch Linux—you'll gain a profound understanding of the gears turning behind the scenes.

Take pride in earning the right to say, *"I use Linux, BTW!"* and wield it with the confidence of someone who has walked the path of mastery.

**The choice is yours: will you rise as a Linux legend, or tread the safer trails of the mundane?** 🐧🔥


---

### Connecting to the Internet

| Step | Command                                                   | Explanation                               | Why Doing This                                    | Optional/Mandatory |
|------|-----------------------------------------------------------|-------------------------------------------|--------------------------------------------------|---------------------|
| 1    | `ip addr show`                                            | Shows current network interfaces          | To verify the system recognizes network devices  | Optional            |
| 2    | `iwctl --passphrase "**" station wlan0 connect Your_Wifi_Name` | Connect to Wi-Fi using iwctl              | To set up an internet connection in the live session | Mandatory           |
| 3    | `ip addr show`                                            | Shows current IP addresses                | To verify if an IP address is assigned           | Optional            |
| 4    | `ping google.com`                                         | Sends packets to check connectivity       | To verify internet connection is working         | Optional            |

[🔙](#index)

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
`Note:` In Linux, Logical Volume Manager (LVM) is a device mapper framework that provides logical volume management for the Linux kernel.

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

### Installing Required Packages

| Step | Command                                         | Explanation                                     | Why Doing This                                   | Optional/Mandatory |
|------|-------------------------------------------------|-------------------------------------------------|-------------------------------------------------|---------------------|
| 1    | `pacstrap -i /mnt base`                         | Installs base system packages                   | To install the essential packages for the system | Mandatory           |
| 2    | `y`                                             | Confirms installation                           | To proceed with installing the base packages     | Mandatory           |
| 3    | `genfstab -U -P /mnt >> /mnt/etc/fstab`          | Generates fstab file for mounting partitions     | Required for mounting partitions at boot time    | Mandatory           |
| 4    | `cat /mnt/etc/fstab`                            | Views the generated fstab file                  | To confirm the fstab file is created correctly   | Optional            |
| 5    | `arch-chroot /mnt`                              | Chroots into the installation environment       | To access the installed system from the live environment | Mandatory           |
| 6    | `passwd`                                        | Sets the root password                          | To secure the root user account                  | Mandatory           |
| 7    | `useradd -m -g users -G wheel userName`         | Creates a new user account                      | To create a regular user with sudo privileges    | Mandatory           |
| 8    | `passwd userName`                               | Sets the password for the newly created user    | To secure the user account                       | Mandatory           |
| 9    | `pacman -S base-devel dosfstools grub efibootmgr gnome gnome-tweaks lvm2 mtools nano networkmanager openssh os-prober sudo` | Installs additional necessary packages          | To install required packages for functionality like GRUB, network management, and others | Mandatory           |
| 10   | `y`                                             | Confirms installation                           | To proceed with installing the packages          | Mandatory           |
| 11   | `pacman -S linux linux-headers linux-lts linux-lts-headers` | Installs Linux kernel and headers               | To install the main Linux kernel and a backup (optional) | Mandatory           |
| 12   | `y`                                             | Confirms installation                           | To proceed with installing the Linux kernel      | Mandatory           |
| 13   | `pacman -S linux-firmware`                      | Installs firmware packages                      | Optional for proprietary hardware support        | Optional            |
| 14   | `y`                                             | Confirms installation                           | To proceed with installing firmware packages     | Optional            |

---

### Setting Up Drivers for Video Card

| Step | Command                                          | Explanation                                     | Why Doing This                                   | Optional/Mandatory |
|------|--------------------------------------------------|-------------------------------------------------|-------------------------------------------------|---------------------|
| 1    | `lspci`                                          | Lists PCI devices                               | To identify your GPU                             | Mandatory           |
| 2    | `pacman -S mesa`                                 | Installs drivers for Intel/AMD GPUs             | To set up drivers for Intel or AMD GPUs          | Mandatory for Intel/AMD GPUs |
| 3    | `y`                                              | Confirms installation                           | To proceed with installing the GPU drivers       | Mandatory           |
| 4    | `pacman -S nvidia nvidia-utils nvidia-lts`        | Installs Nvidia drivers for gaming              | For Nvidia GPUs, needed for gaming and better performance | Mandatory for Nvidia GPUs |
| 5    | `y`                                              | Confirms installation                           | To proceed with installing Nvidia drivers        | Mandatory           |
| 6    | `pacman -S nouveau`                              | Installs open-source Nvidia driver              | For Nvidia GPUs if you prefer open-source drivers | Optional            |
| 7    | `y`                                              | Confirms installation                           | To proceed with installing Nouveau drivers       | Optional            |

---

### Special Configuration for Intel and AMD GPUs

| Step | Command                                          | Explanation                                     | Why Doing This                                   | Optional/Mandatory |
|------|--------------------------------------------------|-------------------------------------------------|-------------------------------------------------|---------------------|
| 1    | `pacman -S intel-media-driver`                   | Installs Intel hardware decoding drivers        | For Intel GPUs, enabling hardware decoding       | Optional for Intel GPUs |
| 2    | `y`                                              | Confirms installation                           | To proceed with installing the Intel driver      | Mandatory           |
| 3    | `pacman -S libva-mesa-driver`                    | Installs AMD drivers for hardware decoding      | For AMD GPUs, enabling hardware decoding         | Optional for AMD GPUs |
| 4    | `y`                                              | Confirms installation                           | To proceed with installing the AMD driver        | Mandatory           |

---

### Configuring mkinitcpio and Kernel

| Step | Command                                          | Explanation                                     | Why Doing This                                   | Optional/Mandatory |
|------|--------------------------------------------------|-------------------------------------------------|-------------------------------------------------|---------------------|
| 1    | `nano /etc/mkinitcpio.conf`                      | Opens the mkinitcpio configuration file         | To edit boot settings for the system             | Mandatory           |
| 2    | Add `encrypt lvm2` between `block` and `filesystem` | Updates mkinitcpio to enable LVM and encryption support | To set up disk encryption and LVM                | Mandatory           |
| 3    | `save and exit`                                  | Saves the file changes                         | To save and apply the changes                    | Mandatory           |
| 4    | `mkinitcpio -p linux`                           | Generates the initial ramdisk for the Linux kernel | To create the initial ramdisk                    | Mandatory           |
| 5    | `mkinitcpio -p linux-lts`                       | Generates the initial ramdisk for the LTS kernel | To create the initial ramdisk for backup kernel  | Mandatory for backup kernel |
| 6    | `done for both kernels`                         | Finalizes ramdisk creation                     | Ensures ramdisks are created for both kernels    | Mandatory           |

---

### Setting Locale and GRUB

| Step | Command                                          | Explanation                                     | Why Doing This                                   | Optional/Mandatory |
|------|--------------------------------------------------|-------------------------------------------------|-------------------------------------------------|---------------------|
| 1    | `nano /etc/locale.gen`                           | Opens locale configuration file                 | To set the system language                       | Mandatory           |
| 2    | Uncomment `en_US.UTF-8`                          | Activates English locale                        | To set English as the system language            | Mandatory           |
| 3    | `locale-gen`                                     | Generates locale files                          | To create the locale files                       | Mandatory           |
| 4    | `nano /etc/default/grub`                         | Opens the GRUB configuration file               | To configure GRUB for booting                    | Mandatory           |
| 5    | Add `cryptdevice=/dev/nvme0n1p3:volgroup0` to `GRUB_CMDLINE_LINUX_DEFAULT` before `quiet` word | Specifies encrypted partition for booting | To enable booting from encrypted LVM partition   | Mandatory           |
| 6    | `exit`                                           | Exits the file editor                           | To save and exit                                | Mandatory           |

---

### Setting Up EFI and Installing GRUB

| Step | Command                                          | Explanation                                     | Why Doing This                                   | Optional/Mandatory |
|------|--------------------------------------------------|-------------------------------------------------|-------------------------------------------------|---------------------|
| 1    | `mkdir /boot/EFI`                                | Creates the EFI directory                       | To prepare for GRUB installation                 | Mandatory           |
| 2    | `mount /dev/nvme0n1p1 /boot/EFI`                 | Mounts the EFI partition                        | To mount the EFI partition for GRUB installation | Mandatory           |
| 3    | `grub-install --target=x86_64-efi --bootloader-id=grub_uefi --recheck` | Installs GRUB bootloader                        | To install GRUB to manage booting                | Mandatory           |
| 4    | `cp /usr/share/locale/en@quot/LC_MESSAGES/grub.mo /boot/grub/locale/en.mo` | Copies GRUB locale files                        | To set GRUB's locale settings                    | Mandatory           |
| 5    | `grub-mkconfig -o /boot/grub/grub.cfg`           | Generates GRUB configuration file               | To create the GRUB configuration file            | Mandatory           |
| 6    | `systemctl enable gdm`                           | Enables GDM display manager                     | To start the graphical login manager on boot     | Mandatory           |
| 7    | `systemctl enable NetworkManager`                | Enables NetworkManager service                  | To ensure network management starts at boot      | Mandatory           |

---

### Final Steps

| Step | Command                                          | Explanation                                     | Why Doing This                                   | Optional/Mandatory |
|------|--------------------------------------------------|-------------------------------------------------|-------------------------------------------------|---------------------|
| 1    | `exit`                                           | Exits chroot environment                        | To exit the installation environment            | Mandatory           |
| 2    | `umount -a`                                      | Unmounts all mounted partitions                 | To unmount everything before reboot             | Mandatory           |
| 3    | `reboot`                                         | Reboots the system                              | To restart the system with the installed Arch Linux | Mandatory           |
| 4    | `check you have successfully booted`             | Verifies boot success                           | To ensure the system boots properly             | Mandatory           |

---

### Post-Boot Configuration

| Step | Command                                      | Explanation                                           | Why Doing This                                      | Optional/Mandatory |
|------|----------------------------------------------|-------------------------------------------------------|----------------------------------------------------|---------------------|
| 1    | Check Wi-Fi functionality and reconnect      | Ensure your Wi-Fi is working and connected            | To make sure you have an active internet connection | Mandatory           |
| 2    | Go to Settings -> Region and Languages -> Language | Change language to English if necessary                | To set the preferred system language               | Optional            |

---

Congratulations! You have successfully installed Arch Linux. Enjoy your fully customized and functional Arch system!
