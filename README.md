# ArchLinux-Project-Documentation
Documentation on the Installation Guide on an Arch Linux VM in Virtual Box. This documentation will follow the Arch Linux Wiki Installation Guide. 
# Pre-Installation
## Choose your iso  
- Choose the iso depending on your location and download the **Iso file** and the **PGP signature** into a folder (I placed it into the file of ArchVM). 
- Download the iso into the VM. Makes sure to allocate enough RAM to the VM as desired. (I allocated 20G)

## Boot the Live Enviorment 
- Being able to boot successfully into the ISO and have landed into **root@archiso**. 

## Verify Signature 
- This stage helps ensure that the VM is authentic and that it has not been tampered with.
- Make a **Shared Folder** from your personal device on the VM. In Virtual Box, go into the Settings and navigate to "Shared Folder". Add the shared folder that has the Iso file and PGP signature.
	- Commands to create a shared folder:
	````
		sudo mkdir ~/shared_folder 
		sudo mount –t vboxsf ArchVM ~/shared_folder
 	````

- Verify Signfature
 ````
 	cd shared_folder  # Go into the shared_folder
	gpg –-auto-key-locate clear,wkd –verbose -–locate-external-key pierre@archlinux.org 		
	gpg –-verify archlinux-2025.11.01-x86_64.iso.sig archlinux-2025.11.01-x86_64.iso
````
 - Verification worked if command comes back as **"Good Signature from "Pierre Schnitze" <pierre@archlinux.org>"**
   
## Set Console Keyboard Layout and Font 
- Change console keymap: If you wish for your keyboard to remain in English, then the following commands won’t be necessary.  
  ````
  	localectl status # Look at the current keymaps used
  	localectl list–keymaps  # List all types of keyboards
  	loadkeys keymaps    # Replace keymaps with the desired keyboard listed from the above commands
  ````

## Change Console Font: 
- Change console font: If you wish for your font to remain the same, then the following commands won’t be necessary.  
- Command:
  ````
	ls /usr/share/kbd/consolefonts/   # List all of the fonts 
	setfont desired_font  # Change desired_font into the desired font listed from the above commands
  ````

## Verify the Boot Mode 
- To verify the boot mode, we need to verify the UEFI bitness using the following command:
  ````
  	cat /sys/firmware/efi/fw_platform_size
  ````
 - When following command, it returned as **64**. This signifies that the system has booted in UEFI mode and has a 64-bit UEFI

## Connect to the Internet 
- When doing the installation in a VM, your VM should already have a network established.
- But it is important to verify connection:
  ````
	 ping ping.archlinux.org # Test connection
  ````
  - When doing the above command, if there is a network connection it should return a response from the server that it has connected. 

## Update the System Clock 
- Check if the system clock is correct using the status command. If the time is corrent, then no action will need to be taken.
  ````  
	timedatectl status # Displays the device clock 
	timedatectl list-timezones # list all the different time zones
	timedatectl set-timezone desired_timezone  # Change desire_timezone to the desire timezones listed from the above commands
  ````

## Partition the Disks 
- Use the command of **lsblk** to list the different partition.
- Using the command # fdisk /dev/disk_to_be _partitioned allows the user to partition the main storage device. (I will be using # fdisk /dev/sda) 
- To delete desired existing partitions, do the following command till all existing partitions are deleted Command (m for help):# d 
- **Note**: The size will depend on how much storage you want to allocate to each partition the one given below are suggestions 

- Creating Partition 1: EFI
````
	- Command (m for help):# n 
		- Partition number (1-128, default 1):# Enter tab 
		- First sector (..., default 2048):# Enter tab 
		- Last sector ..:# 500M # 500M
````
- Creating Partition 2: Main
	````
		- Follow the commands listed below: 
		- Command (m for help):# n 
		- Partition number (2-128, default 2):# Enter tab 
		- First sector (... default ...):# Enter tab 
		- Last sector ...:# Enter tab # Remainder space
 	````
- Changing partition types
  ````
		- Command (m for help):# t 
		- Partition number (1-2, default 2):# 1 
		- Partition type or alias (type L to list all):# uefi  
		- Partition number (1-2, default 2):# 2
		- Partition type or alias (type L to list all):# linux 
		- Command (m for help):# w   Writes the partition in disk
  ````
- Format the partitions 
	- Formatting the new partition into the appropriate file system.
	- Commands for EFI and Main
   ````
	mkfs.fat -F 32 /dev/sda1 
	mkfs –t ext4 /dev/sda2
   ````
## Mount the File Systems 
````
	mount /dev/sda2 /mnt 
	mkdir –p /mnt/boot
	mount /dev/sda1 /mnt/boot
````

# Installation
## Select the Mirrors
- Packages to be installed must be downloaded from mirror servers. 
	````
	pacman -Sy reflector
	reflector --protocol https --sort rate --latest 20 --save /etc/pacman.d/mirrorlist
	````
## Installing Essential Packages 
- Makes sure that the system is functional and secure by supporting the hardware after installation.
````
	pacstrap –K /mnt base linux linux-firmware  
````
# Configure the System 
## Fstab 
- Ensures that the partition are automatically mounted at startup.
	```` 
	genfstab –U /mnt >> /mnt/etc/fstab
	````
## Chroot 
- Allows user to install packages, configure settings, and set up the environment 
	```` 
	arch-chroot /mnt 
	````
## Set up Time  
- Setting up the corresponding timezone and hardware clock setup
	````
 	ln -sf /usr/share/zoneinfo/Region/City /etc/localtime # Changed to /US/Central 
 	hwclock --systohc 
	````
## Localization 
- To use the correct region and language specific formatting.
	````
	pacman -Syu vim
 	vim /etc/locale.gen
		- Choose the localization to the desired one(I choosed en_US.UTF-8 UTF-8). Once choosen, remove the **#**.
  		- To exit, use the command use Esc and type **:wq** to save.
	locale-gen 
	````

## Network configuration 
- To assign a consistent, identifiable name to your system
	````
 	echo archlinux-desktop > /etc/hostname
	````
- Configure networking
	- Configure the network for the VM
	````
 	pacman -Sy networkmanager
   	systemctl start NetworkManager
  	systemctl enable NetworkManager
	````

## Initramfs 
- It's a temporary root filesystem that is loaded into memory. It prepares the system enviornment to be mounted and booted.
	````
	mkinitcpio -P
	````
## Root Password 
- Sets up the password for root
	````
	passwd
	````
## Boot loader 
= Choose and install a Linux-capable boot loader (I decided to go with Grub).
- Grub installation:  
  ````
	grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRU
	grub-mkconfig –o /boot/grub/grub.cfg
  ````

# Reboot
````
	exit # Exit chroot enviornment
	umount –R /mnt
	reboot # Reboot the system
````
- After rebooting the system, shut down the system and naviagate to **Settings**. In settings, go to the **Storage** tab and remove or change the device sequence order of the arch-linux iso into the primary boot device of the newly built Arch Linux VM.  

# Post Installation 
- In this stage of developing Arch Linux, it is important to consider the packages to install core tools.

## Updating Current Packages and Installation of New Packets
- Use the following command to update the current packages in the system.
	````
 	sudo pacman –Syu update
	````
- The following are some examples of basic installation:
  	- Use the command of "**sudo pacman -Syu BLANK**" to install the following commands(Replace BLANK with the package that is wanted to be installed).
  		- Sudo, nano, vim, bash, man, grep, git, curl, wget, openssh,, htop, locate, zip, unzip, tar, gzip, xz, vi and base-devel 

## Creating Sudo User Accounts 
- Create users:
	````
	sudo useradd –m –G wheel jaz # Adds jaz into the group of wheel
	passwd jaz # Gives jaz a password
	sudo useradd –m –G wheel codi # Adds codi into the group of wheel
	passwd codi # Gives codi a password
	````
- Enable sudo:
	````
	EDITOR=nano visudo  # Edit the sudoers file using visudo
	````
 	- Uncomment "%wheel ALL=(ALL:ALL) ALL" by removing the **#**
		- Gives all users in wheel **sudo** privileges
	- To look at all of the people in wheel use the commands of **getent group wheel** to see all sudoers
 
## Install a New Shell; zsh 
- Commands: 
````
sudo pacman -Syu zsh  #Install zsh
which zsh  #Verify zsh path
zsh -s /usr/bin/zsh  #Changing shell for root
````

## Installed SSH 
- Commands:
````
pacman -Sy openssh 
systemctl start sshd 
systemctl enable sshd
systemctl status sshd
 ````

## Added Shell Color Coding 
	````
	echo -e "\033[0;31mThis message is in Red\033[0m" #Returns "This message is in Red" # in Red
	echo -e "\033[0;35mHello World\033[0m" #Returns "Hellow World" # in Magenta
	````
## Added At Least 2 Alias
````
alias install='pacman -Sy'
alias update='pacman -Syu update'
````
  
# Customization
- This will differ depending on the user needs. This will just go over the basic options. 
## Installing a Graphical Environment
### Installing a Display Server
- Your can either chose Xorg or Wayland. To compare use the below url.
	- https://thelinuxcode.com/xorg-vs-wayland/
 - Command to install Xorg.
	````
 	pacman -S xorg
 	````
### Installing a Desktop Environment and a Display Manager
- Commands to install GNOME:
	````
 	pacman -Sy gnome 
	pacman -Syu gdm
 	systemctl enable gdm
    systemctl start gdm
 	````
 ### Installing AUR Helper
 - AUR helper helps simplifies the process of installing, updating, and managing packaged for Arch User Repository.
	````
	sudo pacman -S --needed base-devel git # Instal base-devel and git
	git clone https://aur.archlinux.org/yay.git # Clone yay
	cd yay
	makepkg -si # Build the installer
	````

# Discussed Installation Problems 
- When transitioning from USB to Windows, I accidentally deleted both from my device, which meant I had to take it to where I bought my computer to see if it could be fixed. I then found out that I wasn't supposed to do that.
- When installing packages, I kept getting errors because the partition was full. I had to increase the partition size by resizing it.
- When verifying the boot mode, there were no results. I forgot to change the settings to accept the UEFI feature. After checking the UEFI box and booting the VM, I checked again and it worked.
- After rebooting the VM at the end of the installation, I forgot to change the device order and started downloading packages to the Arch-Linux ISO instead of the ArchVM.vdi. After deleting the Arch Linux ISO, I logged into the VDI and was able to save my changes and view my graphical environment. 
