# CYB-3353: Arch Linux Installation (MacOS)
Welcome to the Arch Linux Installation Documentation! This guide will go through the steps to install Arch based on the wiki instructions, as well as a few modifications that can be tweaked as desired. 

Arch Linux is a distribution that is structured by its five prinicples of simplicity, modernity, pragmatism, user-centrality, and versatility. Any configurations are completely to the user's discretion.

The hardware used for this installation is **VMware Fusion 12** running on **MacOS 12.5** with an **Intel** chip.


# Installation Guide
It is recommended that you read through and follow the installation guide on the official wiki [here](https://wiki.archlinux.org/title/installation_guide). Regardless, this section will narrow it down to what's really important for brevity and clarity. If you have any questions or are really stuck, the forums [here](https://bbs.archlinux.org/) might be of use. In the worst case scenario, deletion of the current VM and a redo of all the steps may be necessary.

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

### Formatting Partitions (ignore SWAPS)
Enter fdisk with the following: `# fdisk /dev/sda`

Two disks will be partitioned: **sda1** for booting in UEFI mode (an EFI system partition), and **sda2** for the root directory. First, type the command `n` to create a partition.

* **sda1** --> the **first two args** will be defaults; since a good UEFI has at least 300MiB of allocated size, the **last arg** will be inputted as `+500M`

* **sda2** --> **all three args** will be defaults, because a root partition is the remainder of the device.

Verify your partitions by typing `p`, then save/exit fdisk by typing `w`.

<img width="750" alt="type p" src="https://user-images.githubusercontent.com/62861572/200035793-9cb71189-71b9-4827-9bb5-be6d42fd206b.png">

*WARNING:* Make sure that you use the correct file systems! For instance, do **NOT** format an EFI system partition with an ext4 file system (I remedied this mistake by deleting then recreating the partition.)

### Mounting File Systems
To mount each file system, run the following commands:

* `# mount --mkdir /dev/sda1 /mnt/boot` --> mount the EFI system partition, sda1, to /mnt/boot

* `# mount /dev/sda2 /mnt` --> mount the root volume, sda2, to /mnt

Verify your mount points by typing `lsblk -f`:

<img width="1000" alt="lsblk" src="https://user-images.githubusercontent.com/62861572/200035455-f6f44857-4244-4e2a-a7a6-b206db78105e.png">

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

## 3. Root Change
We will now change root into our new system to perform further maintenance:
`# arch-chroot /mnt`

Similarly with the previous terminal, we will make sure that certain packages are installed and conditions verified. This can be achieved by running the following commands:
* `# ln -sf /usr/share/zoneinfo/Region/City /etc/localtime` --> set the proper timezone (e.g. `America/Chicago`)
* `# hwclock --systohc` --> generate the necessary /etc/adjtime file for proper function
* `# pacman -S man-pages texinfo sudo grub grub-install nano man-db sof-firmware dosfstools amd-ucode wpa_supplicant wireless_tools networkmanager nm-connection-editor network-manager-applet` --> installs the necessary packages beforehand for proper function

For more information on each of these packages, please visit the arch linux database [here](https://archlinux.org/packages/).

### Localization
There a few more steps required to complete the language adaptation. This will vary depending on your location/language. For the US in Central Time:
1. Edit locale.gen using `nano /etc/locale.gen` file and uncomment `en_US.UTF-8 UTF-8`
2. Create the locale.conf file using `touch /etc/locale.conf` and add `LANG=en_US.UTF-8`
3. Edit vconsole.conf using `nano /etc/vconsole.conf` and add `KEYMAP=us`

### Network Configuration
Before setting up the network configuration, make sure that NAT is enabled in the setttings, your interface is visible, and your machine is connected to the Internet. Odds are that you are using a dynamic IP address, so the instructions will follow suit (for brevity, use `nano` and `touch` to edit and create when necessary):
1. Create `/etc/hostname` and input a hostname to identify the network(e.g. `twibi`)
2. Edit `/etc/hosts` with the following: `127.0.0.1 localhost ::1 localhost 127.0.1.1 yourhostname` to resolve localhost over the network


### Firmware Structure
Just a few more things left to run before we wrap up:
* `# mkinitcpio -P` --> already addressed in package installation, but wouldn't hurt to mount
* `# passwd` --> set the password of your root
### Boot Loader (read carefully)
Finally, the last configuration is to install our boot loader. For this guide, we will install **GRUB** as it is considered to be the most widely used and popular. 

*WARNING:* Please follow these instructions **carefully**. Failure to do so will result in an error message on boot and unless you know what do to, you will have to **restart** your entire installation from the beginning, such as in my case (very unfortunate!):

<img width="500" alt="lsblk" src="https://media.discordapp.net/attachments/1019119382510702677/1036847382199009351/unknown.png?width=1354&height=1094"> 

To install GRUB, run the following: 
1. `# mount /dev/sda1 /mnt` --> (IMPORTANT) mounts the EFI system partition.
2. `# grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB` --> installs GRUB and its modules to the appropriate location.
3. `# grub-mkconfig -o /boot/grub/grub.cfg` --> required to enable microcode updates if using an Intel or AMD chip.

### Finally... run the following:
* `exit` --> exits the chroot environment
* `reboot` --> restarts the machine

If all the steps above have been done correctly, then your system should reboot into Arch Linux and prompt you to login with the root user (with the password you created earlier). Congrats on completing the first section! 🥳

<img width="500" alt="lsblk" src="https://cdn.discordapp.com/attachments/364563015917502466/1038149441728806942/image.png"> 


# Modifications
Now that you have successfully installed Arch Linux, it's time to explore one of Arch Linux's greatest features, which is seamless **customization**. This guide will show you just a few ways you can customize your new VM to your liking.

## 1. Create user accounts
Create a user for yourself and Codi w/ passwords. An example with using my name:
* `useradd -m tulsano`
* `passwd tulsano`
* `useradd -m codi`
* `passwd codi` --> set to "GraceHopper1906"

Then, for each user you want to grant sudo priviliges:
* `EDITOR=nano visudo` --> file is required to be edited with visudo cmd
* `username ALL=(ALL) ALL` --> replace username with desired user

Save and exit the file. A sudo user is required for our next step of installing a Desktop Environment, which will give the VM more life!


## 2. Install a compatible DE
For this guide, we will install [KDE](https://kde.org/) due to its accessibility and beautiful GUI customization:
* `pacman -Syu` --> update any outdated Arch packages, if any
* `pacman -S xorg plasma plasma-wayland-session kde-applications` --> display server protocol necessary for KDE 
* `reboot` --> this will reboot the system into KDE.

<img width="500" alt="lsblk" src="https://media.discordapp.net/attachments/245326317090897920/1038164513054261428/image.png?width=1354&height=1094">

NOTE: If you're having trouble installing KDE packages, you might have to make sure the correct services are enabled/disabled:
* `sudo systemctl start NetworkManager.service`
* `sudo systemctl disable dhcpcd.service`

## 3. Install a different shell
By default, the terminal comes in bash shell. But you may wish to install a different shell such as zsh or fish. In our case, we will install zsh:
1. Search up "Konsole" in the KDE. This is your terminal.
2. `sudo pacman -S zsh` --> install zsh with pacman for Arch
3. `zsh --version` --> verify the installation worked properly
4. `chsh -s $(which zsh)` --> make it your default shell
5. Restart your VM to have the changes go in effect.

If the terminal prompts you enter a configuration, go ahead and configure it with the auto-complete system and default settings.

TIP: You can customize your zsh shell using **Oh-My-Zsh**:
1. `sudo pacman -S git` --> necessary for installation (will give error if not present)
2. `sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"` --> to install Oh-My-Zsh
3. `nano ~/.zshrc` --> enable themes here, under `ZSH_THEME` (see the full [list](https://github.com/ohmyzsh/ohmyzsh/wiki/Themes))


## 4. Install and use ssh
(todo)
## 5. Add color coding
(todo)
## 6. Boot into GUI
(todo)
## 7. Add .bashrc/.zshrc alisases
(todo)

# Final Words
Hopefully, you have now installed Arch Linux on your machine and implemented some of the many customizations that it has to offer. Thank you so much for using this guide! Continue to spark your creativity and best of luck in the world of CS! 🤓


