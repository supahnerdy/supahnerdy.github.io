# CYB-3353: Arch Linux Installation (MacOS)
Welcome to the Arch Linux Installation Documentation! This guide will go through the steps to install Arch based on the wiki instructions, as well as a few modifications that can be tweaked as desired. 

Arch Linux is a distribution that is structured by its five prinicples of simplicity, modernity, pragmatism, user-centrality, and versatility. Any configurations are completely to the user's discretion.

The hardware used for this installation is **VMware Fusion 12** running on a **MacOS 12.5.**


# Installation Guide
It is recommended that you read through and follow the installation guide on the official wiki [here](https://wiki.archlinux.org/title/installation_guide). Regardless, this section will narrow it down to what's really important for brevity and clarity.

Also, it is recommended that you complete this section all in one sitting. If you must leave during the installation process, I suggest that you finish the step you are currently on, pause your VM, and leave your computer on standby.

## 1. First Steps
NOTE: These initial steps will look different if you are running different hardware (such as VMware Workstation on Windows), but the configuration is all the same.
1. Download the Arch **.iso** file (x86_64) from any [mirror](https://archlinux.org/download/).
2. Click the **+** sign in the top left of VMWare Fusion, and select **New…**
3. **Drag** the .iso file into the box

### Installation Wizard
1. Select the OS: **Linux --> Other Linux 5.x kernel 64-bit**
2. Specify the boot firmware: **UEFI** (leave secure boot unchecked)

### Customize Settings
1. Processors: **2 processor cores** (do not overprovision)
2. Memory: **2048MB** (if low RAM: at least **1024MB** to reduce issues)
3. Internet sharing: **Share with my Mac** (we are using NAT)
4. Disk size: **20GB** (we'll allocate more than enough)
5. Bus type: **SCSI**
6. **UNCHECK** pre-allocate disk space (avoid storage issues!)

Boot up your newly created virtual machine. Select the **Arch Linux install medium** and press Enter. After many messages, you will then be logged into the root user and given a shell prompt for the rest of your installation.

## 2. Inside the Terminal
First things first, it is best to make sure that certain configurations are set and conditions are verified. This can be achieved by running the following commands:
* `# ls /sys/firmware/efi/efivars` --> verifies we are in UEFI mode (directory exists)
* `# ip link` --> verifies that a newtwork connection is established
* `# loadkeys us` --> sets the keyboard layout to American English
* `# timedatectl set-timezone “Region/City”` --> sets the timezone to the correct location (e.g. `America/Chicago`)

### Formatting Partitions
Enter fdisk with the following: `# fdisk /dev/sda`

Two disks will be partitioned: `sda1` for booting in UEFI mode (an EFI system partition), and `sda2` for the root directory. First, type the command `n` to create a partition.

* `sda1` --> the **first two args** will be defaults; since a good UEFI has at least 300MiB of allocated size, the **last arg** will be inputted as `+500M`

* `sda2` --> **all three args** will be defaults, because a root partition is the remainder of the device.

Verify your partitions by typing `p`. *(TODO: screenshot here?)*

WARNING: Make sure that you use the correct file systems! For instance, do **NOT** format an EFI system partition with an ext4 file system (I remedied this mistake by deleting then recreating the partition.)

### Mounting File Systems
To mount each file system, run the following commands:

* `# mount --mkdir /dev/sda1 /mnt/boot` --> mount the EFI system partition, sda1, to /mnt/boot

* `# mount /dev/sda2 /mnt` --> mount the root volume, sda2, to /mnt

Verify your mount points by typing `lsblk -f`. *(TODO: screenshot here?)*

NOTE: If your machine shuts down at any time before the final reboot step has been completed, you will have to re-mount the disks each time. 

### Install Needed Packages
Besides the base packages, our Arch installation may require other components to fully function. The following command will install the base packages plus other handy ones:

`# pacstrap -K /mnt base linux linux-firmware sudo nano grub dhcpcd efibootmgr`

A quick rundown of each package is as follows:
* `base` --> minimal package to define Arch installation
* `linux` --> the linux kernel and its modules
* `linux-firmware` --> linux firmware files
* `sudo` --> necessary to run commands with root privileges
* `nano` --> text file editor
* `grub` --> our chosen bootloader for a later step
* `dhcpcd` --> DHCP client, necessary for networking
* `efibootmgr` --> a compliment for our bootloader

### Fstab
It is necessary to generate an fstab file to define a set of rules for file systems/disk partitions with mounting.

 Generate this with the following: `# genfstab -U /mnt >> /mnt/etc/fstab`

 Now that you have established the disk architecture and installed the correct packages, you are ready to change root into the new system. Before you do so, make sure that you have correctly done the steps required to do in the zsh shell.

## 3. Change Root
`# arch-chroot /mnt`


# Modifications
TBA

# Final Words
Hopefully, you have now installed Arch Linux on your machine and implemented some of the many customizations that it has to offer! Thank you so much for using this guide! Continue to spark your creativity and best of luck in the world of CS :)


*(TODO: add final installation desktop image here)*
