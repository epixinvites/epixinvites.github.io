# Arch

### **Do not try to use Arch if you are new to Linux, you will have a great potential at breaking something. Follow this guide at your risk! Using Arch will definitely not going to be a walk in the park.**

## Table of Contents

- Installing Arch
- Using `i3` and `emptty` on Arch (and some software recommendations)
- Arch and Packages
  - Arch and Pacman
  - Arch and the AUR (and yay)
    - PKGBUILD

### Installing Arch
I assume that you know what you are doing before continuing, don't blame me if anything goes wrong. As they said, garbage in garbage out, the computer will return you what you told it to do, so theres no need to point fingers at anyone.

**The only prerequisites is a Arch Linux bootable media**

Don't fret when you're presented with a CLI interface when you first boot up the Arch media. As I said, installing Arch will definitely not be a walk in the park.

So, you'll need to check if your computer is using BIOS or UEFI. Try running `ls /sys/firmware/efi/efivars`. If its empty then your computer is running on BIOS, if it isn't then its UEFI.

### I'll be covering the part for UEFI first since its more complicated

### Follow this guide step by step, don't skip steps or not you may have to restart everything again

### Partitioning

Make sure your `/dev/sda` is an empty disk, or not use parted and format the disk with
```
parted /dev/sda //start parted with /dev/sda as its target device
mklabel gpt //format your hard disk as a gpt type disk
```

A typical partitioning table (a simple one) is
While in the `parted` interface, you can use the `print` command to check your current partition table

EDIT: to make life 100 times easier, run `unit GB` in `parted` to view your partition table as in Gigabyte units. Bytes sometimes can be really confusing!

```
1GB boot partition flags: ('set /dev/your-boot-partition boot on' and 'set /dev/your-boot-partition esp on') partition-type: EFI-partition //this will be your boot partition
A swap partition, normally its 1 times to 2 times the amount of your RAM, adjust it according to your needs (Ex. for me I have 8GB of RAM so I have a 8GB SWAP partition) flags: (set /dev/your-swap-partition swap on) partition-type: Linux Swap
Aaaand the rest of the hard disk as your root '/' partition-type: Linux Filesystem
```
Thats for the partitioning part. Yay, hopefully nothing went wrong.

*This is purely for cross-reference*

`fdisk -l` should return something like this:
```
Disk /dev/sda: 238.47 GiB, 256060514304 bytes, 500118192 sectors
Disk model: HFS256G39TND-N21
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
Disklabel type: gpt
Disk identifier: 51B29D66-921E-48F3-A8D6-D31723075566

Device        Start       End   Sectors   Size Type
/dev/sda1      2048   2099199   2097152     1G EFI System
/dev/sda2   2099200  18876415  16777216     8G Linux swap
/dev/sda3  18876416 500118158 481241743 229.5G Linux filesystem
```


**Now we gotta format the partitions and activate the swap partition**

exit `parted` with the quit command and type the commands accordingly, don't worry I'll explain why.

`mkfs.fat -F32 /dev/your-boot-partition` to format your boot partition as FAT32

`mkswap /dev/swap_partition && swapon /dev/swap_partition` to activate your swap partition

`mkfs.ext4 /dev/root_partition` to format your root partition as `ext4` filesystem type

Congrats, you got your computer partitioned!

### Connecting to the Internet

I believe your wifi card is supported by Arch Linux, and you're using wireless internet, if you're using ethernet and `ping google.com` does return something, then skip this step. But well, knowing how to connect to the internet via a CLI interface is equally important.

Try running `ip link` and see if your wifi card is detected.
Something like this should appear:
```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: wlp1s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DORMANT group default qlen 1000
    link/ether 3c:6a:a7:8f:e5:17 brd ff:ff:ff:ff:ff:ff
```
If you see something like that, then you can proceed to running `iwctl` (iwctl is a command provided by the package iwd)

Here's what you should be running after launching `iwctl`
```
[iwd]# device list
[iwd]# station *device* scan
[iwd]# station *device* get-networks
[iwd]# station *device* connect *SSID*
```
Try pinging www.google.com with `ping google.com` and check if you get replies from the server then you have a connection.

### Installing the system

`mount /dev/root_partition /mnt` so now your root partition is mounted at `/mnt`

`mkdir -p /mnt/boot/efi` for the EFI boot folder

`mount /dev/efi_partition /mnt/boot/efi` mount it

`pacstrap /mnt base base-devel linux linux-firmware nano networkmanager` these are the essential packages that must be installed for functioning Arch system. `nano` and `networkmanager` wasn't mentioned in the official wiki, but I believe its essential for editing files and `nmtui` or `nm-applet` to manage the internet after the system has been installed.

`genfstab -p /mnt >> /mnt/etc/fstab` create the fstab

`arch-chroot /mnt` and now, you've entered the Arch system installed on your computer and not the live media anymore. **Its basically your actual OS.**

You might actually want to run
```
systemctl enable NetworkManager.service
systemctl start NetworkManager.service
```
to have a working internet on your next boot.

### Setting the clock
The timezones are listed in `/usr/share/zoneinfo` so just link your corresponding city to `/etc/localtime` via `ln -sf /usr/share/zoneinfo/Region/City /etc/localtime`

`hwclock --systohc` to make sure your hardware clock is correct.
> Note: Both -w and –systohc option does the same. I like to use –systohc as it is easy to remember. –systohc stands for “system to hardware clock”, which copies the time from system to hardware clock.

### Setting the locale

> A locale is a set of environmental variables that defines the language, country, and character encoding settings (or any other special variant preferences) for your applications and shell session on a Linux system. These environmental variables are used by system libraries and locale-aware applications on the system.

`nano /etc/locale.gen` and uncomment your corresponding locale that you're going to use. Normally, well, for most of us, we uncomment the lines `en_US.UTF-8` and `en_US ISO-8859-1`.

After uncommenting it, you should run `locale-gen` to generate your locales.

Create a new file with `nano /etc/locale.conf`, which will be your locale configuration, and append the lines `LANG=en_US.UTF-8`. This is for those who are using American English.

If you want to set a permanent keyboard layout, go to `/etc/vconsole.conf` and append `KEYMAP=your-keyboard-layout`.

### Setting the hostname
`nano /etc/hostname` and just append the hostname you want for your computer into that file, that file is originally empty btw.

Then you go to `/etc/hosts`, and you add this:
```
127.0.0.1       localhost
::1             localhost
127.0.1.1       myhostname.localdomain  myhostname
```
Change the `myhostname` to the hostname you appended into `/etc/hostname` just now.

### Installing the bootloader
Here comes the confusing and tricky part (at least for me)

`pacman -S grub efibootmgr os-prober` to install grub

`grub-install --target=x86_64-efi --efi-directory=/mnt/boot/efi --bootloader-id=GRUB` to install grub bootloader, if you would want to change the id of the bootloader, feel free to change `--bootloader-id=name-of-your-choice`.

And lastly, you'll need to generate a GRUB config. `grub-mkconfig -o /boot/grub/grub.cfg` to make the config.

### Setting up the repositories
You can skip this part actually, but if you want to edit the mirrors then go to `/etc/pacman.conf` and uncomment out the repositories you want to use.

If you want x32 packages, you might want to uncomment the `multilib` line and the line following it.

### Setting up the user account
`passwd` to first set the password for root

`useradd -m -g users -G wheel,sudo -s your-username` to add a user which can use the `sudo` command. It seriously not recommended to use a root account as your everyday use, its really dangerous so you have been warned.

`passwd your-username` to set the password for the user you're going to use for your everyday use. You should also run `xdg-user-dirs-update` after your reboot to generate the default folders for your home directory.

### Setting up the GUI
Now, you have finished installing the whole Arch OS, I hope its working still fine. But before you reboot into your actual OS, I recommend you to install a display manager and a windows manager.

For me, I use `i3-wm` together with `emptty`. `i3` is a tiling windows manager, they have extensive documentations [here](https://i3wm.org/docs/userguide.html) to get you started. I'll be attaching my `config` in the repository for reference uses. `emptty` is a dead simple CLI display manager, its very configurable and you can check it out [here](https://github.com/tvrzna/emptty). If you don't fancy these kind of stuff you can use choose any of the desktop environments [here](https://wiki.archlinux.org/index.php/Desktop_environment).

I'll be covering how to use `i3` and `emptty` in later guides.

**You have finished setting up a working Arch OS! Congratulations!**

### i3, emptty and Arch recommendations
`i3` can make your life easier by a lot and it uses little to no RAM/CPU usage, almost everything can be configured, theres a variant called `i3-gaps` which as the name indicates, has gaps in between of windows. If you're new to `i3` you may find it really confusing, since everything is done via keyboard shortcuts, and you may need a bit of time to get used to it. Rest assured, it's going to be really nice after you have get used to it. You can copy the default config file from `/etc/i3/config` to `~/.config/i3/config` and edit it.

`emptty` is also a lightweight display manager, it is ugly, but it does the job and its also fast, compared to other GUI display managers. The configuration file it located at `/etc/emptty/conf` and you might want to visit their github documentation [here](https://github.com/tvrzna/emptty) before editing it.

`st` is a lightweight, and super configurable terminal emulator, since the source code is written in C and you'll have to manually compile it and copy the executable to `/usr/local/bin` to launch it via `dmenu` or via `i3` shortcuts. Here's the documentation [https://st.suckless.org/](https://st.suckless.org/)

`dmenu` is commonly used as the application launcher for `i3`, and it isn't recommended to install `dmenu` via `pacman` since suckless tools are meant to be manually compiled. [Dmenu](https://tools.suckless.org/dmenu/)

`slock` is a simple screen locker for linux, its *also* another suckless tools (I wonder why I have such love towards suckless tools lol). [slock](https://tools.suckless.org/slock/)

`i3status-rs` is a status bar for `i3` and I love it due to the lightweightness of it and its also very configurable. It's written in Rust, and you can find the documentation [here](https://github.com/greshake/i3status-rust)

`xscreensaver` is a screensaver, very ugly, but does its job perfectly. `caffeine` is a useful applet for you to control if you want the screensaver to be activated or not. (its just like caffeine preventing you frpm falling asleep while programming late at night)

I don't really fancy Arch's built in clipboard manager, since I've had problems with it previously, `xfce4-clipman-plugin` is what I use.

`dunst` is another simple, configurable and scriptable notification manager, you can copy my config file to `~.config/dunst/dunstrc`

`redshift` is useful if you want to save your poor eyes from straining too much and becoming blind after hours of writing code.

`calcurse` is another lightweight calendar that has tons of features for you to organize your day.

`zsh` is a shell that is a few times better than the `/bin/bash` that you're defaulted to use. Go check out `oh-my-zsh` on github for more information about it.

`doublecmd` is a ugly but a very useful file manager. Its *also* very configurable, with very confusing settings.

### Pacman
Pacman is basically a nom nom friendly package manager (delicious, it eats packages rather than cherries) It's something of `apt` in Debian variants or `zypper` in SUSE variants.

SO, basically what you need to know about pacman, is pretty simple. Works kinda different from `apt` or `zypper`.
```
# Install a package
pacman -S pkg_name
pacman -U pkg_name.pkg.tar.zst
pacman -U https://webpage/pkg_name.pkg.tar.zst

# See what package belongs to which group
pacman -Sg pkg_name # skim through to filter out unnecessary bullshit

# Remove packages
pacman -Rs # remove with its dependencies
pacman -Rsu # remove even more packages
pacman -Rsc # remove all packages that depend on all its dependencies (very complicated) (NOT RECOMMENDED)

# Query packages
pacman -Ss pkg_name # Search the online database
pacman -Qs pkg_name # Search the local database
pacman -Si pkg_name # Extensive information of the package online
pacman -Qi pkg_name # Extensive information of the local package
pacman -Qii pkg_name # Shows even moar extensive information
pacman -Ql pkg_name # What kind of shit did that package install??!!
pacman -Qtd # Catch orphaned packages and yeet them outta ya system
```
There is a configuration file locate in `/etc/pacman.conf`, which is useful when you want to ignore a couple packages that you manually patched or has known issues unsolved. You'll see a line which is commented (or if you did uncomment it) ```#IgnorePkg=```

Ah, and theres also another thing, you can run `sudo paccache -rk1` which is to remove packages in your `/var/cache/pacman`, which is because every time you install a package or upgrade it, the downloaded .pkg.tar.zst will be stored in your computer, and it builds up over time. `sudo paccache -rk1` is to remove all the downloaded packages other than the 1 most recent version (which is useful for rollbacks) if you want to keep only the previous 2 versions then run `sudo paccache -rk2`.

If something goes wrong with your pacman, `/var/log/pacman.log` is your friend.

Scroll down in `/etc/pacman.conf`, you'll find a couple repositories which are commented, if you fancy wine and stuff like that you should uncomment the line where it says `multilib`, dont touch the `-testing` repositories if you have no idea what you are doing :)

### AUR
AUR is the Arch User Repository, well it's recommended to search up the package [here](https://aur.archlinux.org/). Some packages aren't maintained for a long time and may break while building the package. Since the name already indicates that it is maintained by users. You'll be asked whether to edit the PKGBUILD (which is another very complicated thing).

There are a couple AUR package managers available, like `pikaur` and `yay`, but most people prefer `yay`.

`yay` works pretty similar to [Pacman]. Most of the commands listed there should work here. It isn't recommended to run yay with sudo.

If you also want to ignore packages, just add the package in `/etc/pacman.conf`, just like how you did it with `pacman`. Keep in mind `yay` also its own cache and it needs to be cleaned periodically to prevent too much packages building up in the cache.

### PKGBUILD

#### What is `PKGBUILD`?
> A PKGBUILD is a shell script containing the build information required by Arch Linux packages.
> Packages in Arch Linux are built using the makepkg utility. When makepkg is run, it searches for a PKGBUILD file in the current directory and follows the instructions therein to either compile or otherwise acquire the files to build a package archive (pkgname.pkg.tar.zst). The resulting package contains binary files and installation instructions, readily installable with pacman.
> Mandatory variables are pkgname, pkgver, pkgrel, and arch. license is not strictly necessary to build a package, but is recommended for any PKGBUILD shared with others, as makepkg will produce a warning if not present.
> It is a common practice to define the variables in the PKGBUILD in same order as given here. However, this is not mandatory, as long as correct Bash syntax is used.
> ~According to the [Arch Wiki](https://wiki.archlinux.org/index.php/PKGBUILD)

It is recommended to read up how it works [here](https://wiki.archlinux.org/index.php/PKGBUILD) before meddling with it. It is really complicated and I don't think I will be able to cover it here.
