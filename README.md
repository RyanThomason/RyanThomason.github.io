# Setup of Arch Linux

This page is the documentation that goes along with my Arch Linux Install

## Installing the Arch Linux Image

1. Began with heading to the Arch Linux wiki which contained the link to the download page under **Acquire an installation image**. 
2. From there I went down to the worldwide section of the **HTTP Direct Downloads** and went to the rackspace.com link.
3. After going to the link, I downloaded the .iso file for Arch Linux.
4. After the download completed I went into my windows terminal and ran this command:
    - **certutil -hashfile (filepath)** - This command in windows shows the checksum of the file (defaults to sha1)
5. Compared the output of previous command to the **sha1sums.txt** file within the rackspace.com link
6. Once the checksum was verified I then imported the Arch Linux VM into VMware Workstation and started up the image.
    - When importing into VMware Workstation I made sure to give the VM proper specifications (20G and at least 2GB of ram)
7. Once the image was in VMware followed instructions to put the line "firmware="efi"" into the .vmx file to ensure the VM was booting into UEFI mode by default

# Setup After Image is in VMware

1. First thing after launching the VM was to make sure that the image was connected to the internet by just doing a simple ping command to ensure connection: 
    - "ping google.com"

## File Partitioning

1. After changing the file I booted up Arch. Once booted up I went ahead and setup the partitions using the command **cfdisk /dev/sda** which is a pseudo-graphic way of changing partitions.
    - Added /dev/sda1 with 500M type EFI System
    - Added /dev/sda2 with 19.5G type Linux filesystem

2. Once I created the two partitions I then needed to format them to what they should be:
    - For /dev/sda1 which was the EFI System I needed to use the command:
        - **mkfs.fat -F32 /dev/sda1**
            - Reason for -F32 is because that is the recommended for UEFI
    - For /dev/sda2 which was the Linux filesystem I needed to use the command:
        - **mkfs.ext4 /dev/sda2**
            - Reason for .ext4 is because that is one of the usual formatting for linux filesystems

3. After formatting the partitions we then have to mount the partition. To mount the partition we will be using the command:
    - **mount /dev/sda2 /mnt**
        - This command instructs the operating system that a file system is ready to use, and associates it with a particular point in the overall file system.

## Basic Package Installation and Configuration

1. Once the drives are all formatted and mounted we then need to install the essential packages using this command:
    - **pacstrap /mnt base linux linux-firmware**
        - This command will install the base package, the linux kernel, and the linux firmware for common hardware to the mount point.

2. After a few minutes these packages will be installed, and we will run the following command:
    - **genfstab -U /mnt >> /mnt/etc/fstab**
        - This will generate an fstab file

3. Once we do that we then will change root into the new system through our mount point.

4. Then after creating the fstab file we will change root into the mount point. We do this with this command:
    - **arch-chroot /mnt**

5. Once we have chrooted into the system the prompt should have changed to a different looking [root@archiso]. 

6. After this we will do a few commands to make sure things are setup right in the system. 
    - First we will make sure that the clock is setup right. For this we will run this command:
        - **ln -sf /usr/share/zoneinfo/America/Chicago /etc/localtime**
    - Second we will run this:
        - **hwclock --systohc**
    - These commands make sure that the clock is setup for the right time.

7. Once the clock is setup we then move to the locales.
    - For this we need to edit the file located at **/etc/locale.gen**. To do this we need to install some sort of text editor. The one I chose to install was nano. We do this by running:
        - **pacman -S nano**
            - This will install nano so that we are able to edit text files.
    - Then we need to go into that file and uncomment the line **en_US.UTF-8 UTF-8**. After doing this we then run the following command:
        - **locale-gen**
            - This will generate the locale that we just uncommented from the file.
    - Then we need to go and set the LANG variable to **LANG=en_US.UTF-8** in the /etc/locale.conf file with this command:
        - **nano /etc/locale.conf**

8. Once we have setup the locales we then need to configure the network. 
    - For this we will nano into the **/etc/hostname** file and add our host name which is going to be "archvm". We do this with:
        - **nano /etc/hostame**
    - We also will add the following to the **/etc/hosts** file:
        - 127.0.0.1     localhost
        - ::1           localhost
        - 127.0.1.1     archvm

## User Management

1. Then with all of this configured we will then change root passwords and add the necessary users
    - First to change the root password we will use the simple command:
        - **passwd**
    - Then we will add our users.
        - The users we will add are ryan, sal, and codi,
            - We will do this with the command **useradd -m (username)**
        - Once we do this with the three users we will need to set passwords for them and make sal and codi's passwords as "GraceHopper1906" and be set to change after login.
            - We do this with **passwd (username)**
            - Then to change after login we do **passwd -e (username)**
    - We will also have to add them to certain groups. The groups we will add them to are the basic ones especially wheel to give them sudo permissions. We use this to change their groups:
        - **usermod -aG wheel,audio,video,optical,storage (username)**
    - Wheel group allows for sudo but sudo is not installed so we have to install and configure wheel. We can do the following to install sudo.
        - **pacman -S sudo** then
        - **EDITOR=nano visudo**
        - We then uncomment the line talking about wheel.

## Bootloader and Final Steps

1. We then need to install our bootloader. The one we are going to use is GRUB.
    - To install GRUB we need to run this command:
        - **pacman -S grub**
    - Then because we are using EFI we will need to install some more packages. We can do this with this command:
        - **pacman -S efibootmagr dosfstools os-prober mtools**
    - Then we need to make the boot directory with the following:
        - **mkdir /boot/EFI**
    - Then we mount the EFI partition which was /dev/sda1 with this command:
        - **mount /dev/sda1 /boot/EFI**
    - Then we will need to run this command to actually install GRUB:
        - **grub-install --target=x86_64-efi --bootloader-id=grub_uefi --recheck**
    - Then finally we will need to get our GRUB config file created with the following:
        - **grub-mkconfig -o /boot/grub/grub.cfg**

2. Finally we can install some final packages that might be useful for us later. We are going to use the following to do that.
    - We will install our network manager and git for later.
        - **pacman -S networkmanager git**
    - Then to enable the network manager we will use the following:
        - **systemctl enable NetworkManager**

3. Then we will need to reboot the machine. We will first unmount and then shutdown the image using:
    - Exit the chroot environment with **exit** then
    - **umount /mnt** or **umount -l /mnt** then
    - **shutdown now**

4. From here we will need to remove the .iso file from VMware so that when we restart the machine it doesn't boot from the ISO. After doing this and rebooting machine we should be met with a screen that says **archvm login:**.


## Installing and Configuring the Desktop Environment

1. Now that we have installed and configured the basic Arch Linux environment we now need to install and configure our Desktop Environment. The DE that we are using is going to be the Budgie DE.

2. To start this we will need to login to root, which we set earlier. So we will just type **root** into the login and then enter the password we set.

3. After doing this we will need to run the commands to install our DE.
    - The command that we will use is 
        - **pacman -S xf86-video-vmware xorg lightdm lightdm-gtk-greeter budgie-desktop gnome gnome-control-center chromium**
            - These packages are all the ones that we will need to be able to get a working DE. Lightdm is our display manager, budgie-desktop is the actual DE and then we are also going to install our internet browser along side these which is chromium.
    - Then we will need to enable lightdm (our display manager) with this command:
        - **systemctl enable lightdm**
    - Now we can reboot the machine.
    - After rebooting we should be able to login through a DE and then see a Desktop once logged in.

4. Now that we have logged in there are a few things that we will need to do. One thing is changing our desktop wallpaper. I went ahead and changed it to one of the other default wallpapers.

5. Also for me I needed to change the resolution of the Desktop just by going to settings and searching display and then changing it there.


## Installing Necessary Packages, Adding Aliases, etc...

1. First thing that I am going to do is install a theme for the desktop. Im going to install the materia theme. I am also going to install this one program called i3lock. Basically with i3lock you can lock the pc and it go to a lock screen and then you just enter password and it will unlock. To do this I will use the following command:
    - **sudo pacman -S materia-gtk-theme i3lock**
    - Then I just have to go to settings and change to the materia theme
    - For the i3lock I am going to setup a keyboard shortcut in the settings and also an alias for terminal. I am going to have the command it run be:
        - **i3lock -i /home/ryan/Pictures/lockscreen.png**
    - This will show that lockscreen.png as the picture.

2. Next thing that we are going to do is change up our terminal. For this we are going to use a script from github that has a fancy prompt for our terminal that we can edit and make it look cool.
    - For this we are goint to run a couple commands. First is going to be getting a certain font with this command:
        - **sudo pacman -S powerline-fonts**
    - After that we are going to run a series of commands to get the script from github.
        - **git clone --recursive https://github.com/andresgongora/synth-shell.git**
        - **chmod +x synth-shell/setup.sh**
        - **cd synth-shell**
        - **./setup.sh**
    - After running this our terminal should look different. From there we can go into the config files of the script and change up the way it looks.

3. After configuring the terminal the next thing we need to do is mess with aliases
    - One alias we are going to add is going to be for clear we will add this line to the .bashrc file:
        - **alias c='clear'
    - Another one is going to be for **cd ..** which we will add these lines:
        - **alias ..='cd ..'
        - **alias ...='cd ../..'
        - **alias .3='cd ../../..'
        - **alias .4='cd ../../../..'
    - Next we will go ahead and add one to help with installing packages. We will do this for the **sudo pacman -S** command:
        - **alias inpac='sudo pacman -S'**
    - We will also do one for removing packages:
        - **alias rmpac='sudo pacman -R'**
       
4. Finally the last thing to do is to install ssh and then also zsh as our other shell.
    - For this we can do the following
         - **inpac openssh zsh** or **sudo pacman -S openssh zsh** if no alias.

## Problems

1. One problem I encountered was properly setting up the partitions and also formatting them correctly. Had never done this before and wasn't exactly sure what to do. I looked around the wiki trying to figure out what exactly to do and I sort of figured it out. I finally understood what I needed to do but wasn't sure how to do it. I then had one of my classmates help steer me in the right direction to then use this command 'cfdisk /dev/sda' and after that I was able to get the partitioning done. Then I wasn't exactly sure how to format to the proper thing. This took some more looking at the wiki and then I found what formatting you use for each type of system and was able to format them from there.
2. I also missed the mounting step at first and was wondering why my pacstrap command wasn't working but that was just me forgetting to set the mount point because I missed that step.
