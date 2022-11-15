## Installing Arch ISO
I started out by navigating to archlinux.org, and clicking download, after scrolling all the way down to the direct https downloads and finding the mit.edu mirror. I downloaded the most recent ISO from the mirror. 

After downloading the ISO, i used the command below on windows powershell, and compared the output with the given SHA256 hash provided by ArchLinux.org

`Get-FileHash G:\Downloads\archlinux-2022.11.01-x86_64.iso`

I checked the VM for an internet connection by pining ArchLinux.org with this command.

`ping archlinux.org`

After ensuring Arch booted into BIOS by using the command below.


`ls /sys/firmware/efi/efivars` 

I updated the sytstem clock with the command below.

`timedatectl status`

## Setting up the Disk
Start by using this command to list the drives avalible.

`fdisk -l` 

Then use the partition editor fdisk by running the command below. This opens the editor on the specific virtual drive we created on startup.

`fdisk /dev/sda` 
 
In the fdisk editor these are the commands I typed in succession and what they do 
- then typed *n* to create a new partition.
	- Typing *p* to select primary.
	- Hitting enter through all the default settings of the partition.
- Finally saving using the *w* command.

Format the drive using 
`mkfs.ext4 /dev/sda1`

Then mounted the partition with the 
`mount /dev/sda1 /mnt` 

## Linux Install 

I installed the linux firmware with this command 
`pacstrap -i /mnt base`

I then generated the UUIDs with this command 
`genfstab -U /mnt >> /mnt/etc/fstab`

I then configured the installation with,
`arch-chroot /mnt` 

inside that menu I used the command,
`pacman -S linux-lts linux-lts-headers` 

Next I installed Nano,
`pacman -S nano`

Then OpenSSH,
`pacman -S OpenSSH`

and enabled it,
`systemctl enable sshd`

I then added some packages for networking,
`pacman -S networkmanager wpa_supplicant wireless_tools netctl` 

And enabled the NetworkManager on startup with the following command,
`systemctl enable NetworkManager`

Following the video I Installed LVM support, if I ever want to get more technical with this Arch VM this will be helpful.
`pacman -S lvm2`

Now I needed to edit a specific line in /mkinitcpio which has to do with boot using LVM. This command used to nano that file is below. I added *lvm2* to the arguments for *hook*.
`nano /etc/mkinitcpio.conf`

To enable these changes I used this command,
`mkinitcpio -p linux-lts`

To set the locale to my current location, you must edit the file /etc/locale.gen. This can be done by using nano, with the command below.
`nano /etc/locale.gen` 

The specifc locale I uncommented was *en_US.UTF-8 UTF-8* 

I then generated the locale with this command,
`locale-gen` 

Then using `passwd` command I changed the root password to *sysadmin* \

I then added the *grant* and *codi* users with these commands,
`useradd -m -g users -G wheel grant`
	`passwd grant` Changed to *sysadmin*. 
`useradd -m -g users -G wheel codi`
	`passwd -e codi` Changed to *GraceHopper1906* 

Installing sudo with the following command,
`pacman -S sudo`

To give the *wheel* group all permissions it needs, I use nano to edit a file with the command below,
`EDITOR=nano visudo`
	Then I uncommented the `%wheel ALL=(ALL:ALL) ALL` line, which lets the wheel group execute any commands.

## GRUB Install

GRUB is the boot manager recommened by the tutorial I followed, GRUB should be installed with specific packages, I used the command below. 
`pacman -S grub dosfstools os-prober mtools` 

Now to install GRUB I used the command below,
`grub-install --target=i386-pc --recheck /dev/sda` 

To ensure that Arch boots in english, the following command should be used. 
`cp /usr/share/locale/en\@quot/LC_MESSAGE/grub.mo /boot/grub/locale/en.mo` 

To generate the GRUB config file, the following command is used,
`grub-mkconfig -o /boot/grub/grub.cfg`

Now you can exit the chroot by typing `exit` then unmounting everything with the following command,
`umount -a` 

Then, type `reboot` to restart the kernel.

I can now freely log in as `grant` with password `sysadmin`  

## Post Install Tweaks

You can see what time zones are avalible for selection with the following command,
`timedatectl list-timezones`

Then set the timezone with this command,
`timedatectl set-timezone America/Detriot` 

Then enabled the systemctl to sync the clock with the command,
`systemctl enable systemd-timesyncd` 

To set the hostname of the device, I used the following command,
`hostnamectl set-hostname grantarch` 

I then edited /etc/hosts with nano,
`nano /etc/hosts` 
	I added local host as 127.0.0.1, and grantarch as 127.0.1.1

I then installed the ucode for my specific CPU,
`pacman -S intel-ucode`

To install a DE, Xorg is the way to go, 
`pacman -S xorg-server` 

To install a display manager for VMWare Workstation the command should be, 
`pacman -S virtualbox-guest-utils xf86-video-vmware`

To ensure that the VM extensions actually load properly, this command is below,
`systemctl enable vboxservice`

## Desktop Environment

I chose to install GNOME as my DE, the command to install the entire GNOME set of extensions was listed below,
`pacman -S gnome`
`pacman -S gnome-tweaks` was also recomended


For the login screen to boot, you need to do the following command,
`systemctl enable gdm`

Finally, `reboot` to get into the DE and enjoy your Arch-Linux.


## Project Specifcs

I installed fish by typing this command, `pacman -S fish`

Which I then opened by typing `fish`

I added aliases to bash to help shorten commands, 

```
alias c='clear'
alias ls='ls --color=auto'
alias mkdir='mkdir -pv'
alias diff='colordiff'
alias h='history'
alias j='jobs -l'
```
![Pasted image 20221103195838](https://user-images.githubusercontent.com/111905937/202046467-66622baf-a847-4510-8ba4-8f303f70b1fc.png)

![Pasted image 20221103200007](https://user-images.githubusercontent.com/111905937/202046476-61fcf22f-fc54-406b-8815-dd4481d0ba91.png)
