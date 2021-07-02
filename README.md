# alpine-persistent-usb
Guide for making an Alpine Linux USB with persistence

Note: Replace all instances of `{?X}` (including the braces) with the proper value

## Preparation

1. Download the latest version of [Alpline Standard x86_64](https://www.alpinelinux.org/downloads/)
2. Write it to a USB using `dd` or `rufus` and leave a shared or unallocated partition to use later for your `/home` directory
3. Reboot to your boot menu and boot to Alpine on the USB

## Setup Root Partition

1. Login using login: `root`, password: (none)
2. Run `mkdir /media/trueroot`
3. Run `setup-alpine`
  - Diskless install
  - Store configs in `trueroot`
  - Store cache in `/media/trueroot/cache`
4. Run the following commands
  - `apk add util-linux nano` (nano is optional if you're comfortable with vi)
  - `lsblk` and note what partition (usually `/media/usb`) the usb is running from on your USB
  - `blkid | grep /dev/{?0} >> /etc/fstab` where `{?0}` is the currently booted partition,
4. Change your `fstab` to use the uuid and mount it to `/media/trueroot`
  - Run `nano /etc/fstab`
  - Replace the line that contains your root partition with the UUID at the bottom line to creating the following:
  ```UUID={?m1-own-uu1d} /media/trueroot vfat defaults,noatime 0 0```
    where `{?m1-own-uu1d}` is the UUID for your root partition on the last line (added with grep)
  - Press `ctrl`+`o`, followed by `ctrl`+`x` to save and exit
5. Partition your USB
  - Run `fdisk /dev/{?sd}` where `{?sd}` is the USB
  - Create home partition (can also create a shared partition to act like a regular USB`
  - Set home partition of type `Linux home`
  - Select `Write`
  - Select `Yes`
  - Exit
6. Run the following commands to save and reboot:
  - `lbu ci`
  - `reboot`
  
## Setup Home Partition

1. Update your repositories and apkcache
  - Run `apk add nano e2fsprogs f2fs-tools dosfstools util-linux`
  - Run `nano /etc/apk/repositories`
  - Change top line to `/media/trueroot/apks`
  - Uncomment `edge/main` and `edge/community` by removing the `#`
  - Comment out everything else by addting a `#` at the beginning of the line
  - Press `ctrl`+`o`, followed by `ctrl`+`x` to save and exit
2. Format your partitions
  - Run `mkfs.f2fs /dev/{?sd}` where `{?sd}` is the home partition
  - (OPTIONAL, shared partition only) Run `mkfs.fat -F32 /dev/{?sd}` where `{?sd}` is the shared partition
3. Update your fstab to mount your home partition to `/home`
  - Run `blkid | grep /dev/{?sd} >> /etc/fstab` where `{?sd}` is your home partition created above
  - Run `nano /etc/fstab`
  - Change the bottom line to the following:
  ```UUID={?m1-h0m3-uu1d} /home f2fs defaults,noatime 0 0``` where `{?m1-h0m3-uu1d}` is the UUID for your home partition at the last line (added from grep)
  - Press `ctrl`+`o`, followed by `ctrl`+`x` to save and exit
4. Update lbu and apkcache
  - Run `setup-lbu`
  - Type `trueroot`
  - Run `setup-apkcache`
  - Type `/media/trueroot/cache`
5. Run the following commands to save and reboot:
  - `lbu ci`
  - `reboot`
  
## User Configuration
1. Run the following commands for a simple dwm setup
  - `setup-xorg-base`
  - `apk add sudo dmenu st dwm font-fira-mono-nerd firefox`
  - `sed -i '/%wheel all=(all) all/s/^#//g' /etc/sudoers` or simply uncomment `%wheel all=(all) all` in `/etc/sudoers`
  - `adduser user -G wheel`
  - `groupadd user input,tty,video,wheel`
  - `su user`
  - `echo 'exec dwm' >> ~/.xinitrc`
  - `echo 'startx' >> ~/.profile`
  - `sudo lbu ci`
  - `startx`
  
2. Follow this guide for audio
  - https://wiki.alpinelinux.org/wiki/Sound_Setup
  
3. Follow this guide for flatpak
  - Install flatpaks with `--user` since root (system) installs will not save, but user saves in `/home` partition
  - https://gist.github.com/IronOxidizer/06bd98459e9bb0b9d2de3b76fa0ca421

4. Reduce drive writes
  - https://wiki.archlinux.org/index.php/Install_Arch_Linux_on_a_removable_medium#Tips
  - https://wiki.archlinux.org/index.php/Improving_performance#Reduce_disk_reads/writes
  - https://foxutech.com/how-to-disable-enable-journaling/
  - [libeatmydata](https://github.com/stewartsmith/libeatmydata) `eatmydata firefox` https://www.reddit.com/r/programming/comments/1qligk
  
5. Config npm if necessary
  - `npm config set prefix '~/.local'`

6. Config inittab (reduce ttys)
  - `nano /etc/inittab`

7. Config OpenRc for faster boot times
  - `nano /etc/rc.conf`
  - uncomment `rc_parallel="YES"`

8. `lbu ci`
