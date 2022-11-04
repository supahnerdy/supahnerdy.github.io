# CYB-3353: Arch Linux Installation (MacOS)
Welcome to the Arch Linux Installation Documentation! This guide will go through the steps to install Arch based on the wiki instructions, as well as a few modifications that can be tweaked as desired. 

Arch Linux is a distribution that is structured by its five prinicples of simplicity, modernity, pragmatism, user-centrality, and versatility. Any configurations are completely to the user's discretion.


NOTE: The hardware used for this installation is **VMware Fusion 12** running on a **MacOS 12.5.** 

# Installation Guide
It is recommended that you read through and follow the installation guide on the official wiki [here](https://wiki.archlinux.org/title/installation_guide). Regardless, this section will narrow it down to what's really important for brevity and clarity.


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

Boot up your newly created virtual machine. Select the **Arch Linux install medium** and press Enter.



# Modifications
TBA

# Final Words
Hopefully, you have now installed Arch Linux on your machine and implemented some of the many customizations that it has to offer! Thank you so much for using this guide! Continue to spark your creativity and best of luck in the world of CS :)
