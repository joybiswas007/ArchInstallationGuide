#Steps
This is a direct guide to install **Arch Linux**.I used these steps to install **Arch Linux** in my PC.I Hope this guide will help you guys.So, Let's begin right away!.
& if you are reading this guide then i assume you already are booted into **Arch** & following this steps.I've used nano as a text editor during the installation but you can use any text editor you like. Vim, Vi etc.

- [Pre-installation](Pre-installation)
- [Verify signature](Verify signature)
- [Connect to the internet](Connect to the internet)
- [Update the system clock](Update the system clock)
- [Partition the disks](Partition the disks)
- [Format the partitions](Format the partitions)
- [Mount the file systems](Mount the file systems)
- [Installation](Installation)
- [Install essential packages](Install essential packages)
- [Configure the system](Configure the system)
- [Chroot](Chroot)
- [Time zone](Time zone)
- [Localization](Localization)
- [Network configuration](Network configuration)
- [Initramfs](Initramfs)
- [Root password](Root password)
- [Post-installation](Post-installation)

#Pre-installation
Grab Arch Linux ISO file from their [Download](https://archlinux.org/download/) page & boot your usb drive with any bootable software you like.I've used [Balena Etcher](https://www.balena.io/etcher/) in this tutorial.

#Verify signature
It is recommended to verify the image signature before use, especially when downloading from an HTTP mirror, where downloads are generally prone to be intercepted to serve malicious images

```gpg --keyserver-options auto-key-retrieve --verify archlinux-version-x86_64.iso.sig```
Alternatively, from an existing Arch Linux installation run:

```pacman-key -v archlinux-version-x86_64.iso.sig```

#Connect to the internet
Ensure your [network interface](https://wiki.archlinux.org/title/Network_interface) is listed and enabled, for example with [ip-link(8)](https://man.archlinux.org/man/ip-link.8)

```ip link```

verify the connection with [ping](https://en.wikipedia.org/wiki/ping_(networking_utility))

```ping archlinux.org```

or Make sure your PC is connected to the Internet with Ethernet Cable.

#Update the system clock
Use [timedatectl(1)](https://man.archlinux.org/man/timedatectl.1) to ensure the system clock is accurate:

```timedatectl set-ntp true```
To check the service status, use `timedatectl status`

#Partition the disks
Use `cfdisk` to Patrition Disk.We'll be using `cfdisk` in this guide.It's Super easy to Partition the disks with `cfdisk`.Seariously! try it once.

#Format the partitions
as you are done with the making partitions now let's format them for making them usable.

```mkfs.ext4 /dev/root_partition```

```mkfs.fat -F32 /dev/efi_system_partition```
If you created an [EFI system partition](https://wiki.archlinux.org/title/EFI_system_partition#Format_the_partition), format it to FAT32 using [mkfs.fat(8)](https://man.archlinux.org/man/mkfs.fat.8).

```mkswap /dev/swap_partition```

#Mount the file systems

```mount /dev/root_partition /mnt```
```mount /dev/sda1 /mnt/boot/efi```

If you created a [swap](https://wiki.archlinux.org/title/Swap) volume, enable it with [swapon(8)](https://man.archlinux.org/man/swapon.8):

```swapon /dev/swap_partition```

Now let's create some directory

```mkdir -p /mnt/boot/efi```

```mkdir /mnt/home```

#Select the mirrors
Packages to be installed must be downloaded from [mirror](https://wiki.archlinux.org/title/Mirrors) servers, which are defined in `/etc/pacman.d/mirrorlist`. On the live system, after connecting to the internet, reflector updates the mirror list by choosing 20 most recently synchronized HTTPS mirrors and sorting them by download rate.

The higher a mirror is placed in the list, the more priority it is given when downloading a package. You may want to inspect the file to see if it is satisfactory. If it is not, edit the file accordingly, and move the geographically closest mirrors to the top of the list, although other criteria should be taken into account.

This file will later be copied to the new system by pacstrap, so it is worth getting right.

#Install essential packages
Use the [pacstrap(8)](https://man.archlinux.org/man/pacstrap.8) script to install the [base](https://archlinux.org/packages/?name=base) package, Linux [kernel](https://wiki.archlinux.org/title/Kernel) and firmware for common hardware:

```pacstrap /mnt base linux linux-firmware vim nano linux-headers base-devel```

Tip:
You can substitute linux for a kernel package of your choice, or you could omit it entirely when installing in a container.
You could omit the installation of the firmware package when installing in a virtual machine or container.
The base package does not include all tools from the live installation, so installing other packages may be necessary for a fully functional base system. In particular, consider installing:

userspace utilities for the management of file systems that will be used on the system,
utilities for accessing **RAID** or **LVM** partitions,
specific firmware for other devices not included in **linux-firmware** (e.g. sof-firmware for sound cards),
software necessary for networking,
a text editor,
packages for accessing documentation in man and info pages: `man-db`, `man-pages` and `texinfo`.
To install other packages or package groups, append the names to the pacstrap command above (space separated) or use pacman while chrooted into the new system. For comparison, packages available in the live system can be found in packages.x86_64.

#Configure the system
Generate an [fstab](https://wiki.archlinux.org/title/Fstab) file (use -U or -L to define by [UUID](https://wiki.archlinux.org/title/UUID) or labels, respectively):

```genfstab -U /mnt >> /mnt/etc/fstab```

Check the resulting `/mnt/etc/fstab file`, and [edit](https://wiki.archlinux.org/title/Textedit) it in case of errors.

#Chroot
[Change root](https://wiki.archlinux.org/title/Change_root) into the new system:

```arch-chroot /mnt```
- Now let's install some necessary packages which we need to complete the install process

```pacman -S grub efibootmgr efivar networkmanager intel-ucode amd-ucode```

if you device is support Intel microcode then install `intel-ucode` or if you device support AMD micorcode then install `amd-ucode`.

#Time zone
Set the [time zone](https://wiki.archlinux.org/title/System_time#Time_zone):

```ln -sf /usr/share/zoneinfo/Region/City /etc/localtime```

change the Region & City according to your location.

```ln -sf /usr/share/zoneinfo/Asia/Dhaka /etc/localtime```

or check all the available regions

```cd /usr/share/zoneinfo && ls```

& select the one you need.

Run [hwclock(8)](https://man.archlinux.org/man/hwclock.8) to generate `/etc/adjtime`:

```hwclock --systohc```

#Localization
Edit `/etc/locale.gen` and uncomment `en_US.UTF-8 UTF-8` and other needed locales. Generate the locales by running:

```locale-gen```

- Create the locale.conf(5) file, and [set the LANG variable](https://wiki.archlinux.org/title/Locale#Setting_the_system_locale) accordingly:

```nano /etc/locale.conf``` //Enter the followings in locale.conf file//

`LANG=en_US.UTF-8`

- If you [set the console keyboard layout](https://wiki.archlinux.org/title/installation_guide#Set_the_console_keyboard_layout), make the changes persistent in [vconsole.conf(5)](https://man.archlinux.org/man/vconsole.conf.5):
- NOT Necessary Nonetheless try this

```nano /etc/vconsole.conf```

#Network configuration
- Create the hostname file:

```nano /etc/hostname```
`myhostname`

Alternatively, using [hostnamectl(1)](https://man.archlinux.org/man/hostnamectl.1):

```hostnamectl set-hostname myhostname```

- Some software may however still read /etc/hosts directly, see [4] [5] for examples. To prevent them from potentially breaking, hanging or otherwise delaying operation, make sure they can resolve the local hostname and localhost by configuring the hosts(5) file:

```nano /etc/hosts```

```
127.0.0.1	localhost
::1		localhost
127.0.1.1	myhostname
```

Complete the [network configuration](https://wiki.archlinux.org/title/Network_configuration) for the newly installed environment. That may include installing suitable [network management](https://wiki.archlinux.org/title/Network_management) software.

#Initramfs
Creating a new initramfs is usually not required, because [mkinitcpio](https://wiki.archlinux.org/title/Mkinitcpio) was run on installation of the kernel package with pacstrap.

For [LVM](https://wiki.archlinux.org/title/Install_Arch_Linux_on_LVM#Adding_mkinitcpio_hooks), [system encryption](https://wiki.archlinux.org/title/Dm-crypt) or [RAID](https://wiki.archlinux.org/title/RAID#Configure_mkinitcpio), modify [mkinitcpio.conf(5)](https://man.archlinux.org/man/mkinitcpio.conf.5) and recreate the initramfs image:

```mkinitcpio -P```

#Root password
Set the root password:

```passwd```

Now let's install Grub in EFI directory.

```grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=GRUB```

- generate  grub file

```grub-mkconfig -o /boot/grub/grub.cfg```

- Enable the NetworkManager

```systemctl enable NetworkManager```

- Exit the chroot environment by typing `exit` or pressing `Ctrl+d`

```exit```

#Unmount

```/umount/dev/efi_system_partition```

```/umount /mnt```

- Optionally manually unmount all the partitions with `umount -R /mnt`: this allows noticing any "busy" partitions, and finding the cause with [fuser(1)](https://man.archlinux.org/man/fuser.1)

#Reboot

```reboot```

- and if your pc shows you login screen then you've successfull installed **Arch Linux** Cheers!

username: `root`
password: `thatUsetduringinstalltion'

#Post-installation
A new installation leaves you with only the superuser account, better known as "root". Logging in as root for prolonged periods of time, possibly even exposing it via SSH on a server, is insecure. Instead, you should create and use unprivileged user account(s) for most tasks, only using the root account for system administration.

```useradd --create-home myuser```

& set password for your user

```password myuser```

let's give your user some power

```usermod -aG wheel,users,power,storage myuser```

`-a = append the user to the supplemental GROUPS mentioned by the -G option without removing the user from other groups.`
`-G, --groups GROUPS = new list of supplementary GROUPS`

run `usermod --help` for more details.

- Now give your user permission when your user run `sudo` commands it'll ask for `password`.

```nano /etc/sudoers```
## Uncomment to allow members of group wheel to execute any command

`#wheel ALL=(ALL) ALL` & uncomment this line by just removing `#`

and it'll look like this `wheel ALL=(ALL) ALL`

- & now you're good to go.Now let's install Graphical Interface.
let's install a display driver.we will use xorg in this tutorial.
**Xorg**
Xorg (commonly referred to as simply X) is the most popular display server among Linux users. Its ubiquity has led to making it an ever-present requisite for GUI applications, resulting in massive adoption from most distributions. See the Xorg Wikipedia article or visit the Xorg website for more details.

```pacman -S xorg```

and also let's install some fonts

```pacman -S ttf-dejavu ttf-droid ttf-hack ttf-font-awesome otf-font-awesome  ttf-lato ttf-liberation ttf-libertine ttf-opensans ttf-robot ttf-ubuntu-font-family```

```nano /etc/profile.d/freetype2.sh```
=> Uncomment this line
`#export FREETYPE_PROPERTIES="truetype:interpreter-version=40"`

& make it looks like this

`export FREETYPE_PROPERTIES="truetype:interpreter-version=40"`
& the final stage let's install a Display Manager & Desktop Environment.

i'll be using **KDE** in this one cause `I Love KDE` <3.However you can use any display manager & desktop environment.

```pacman -S sddm kde-plasma-desktop```

- & let's enable our display manager

```systemctl enable sddm.service```
```systemctl start sddm.service```

& you are done.Enjoy Your **Arch Linux** and feel free to contribute if you think i missed something.

- Here is the [Official Arch Linux Install Guide](https://wiki.archlinux.org/title/installation_guide).