# alpine-persistent-usb
Guide for making a Alpine Linux USB with persistence

Note: Replace all instances of `{?X}` (including the braces) with the proper value

## Preparation

1. Download the latest version of [Alpline Standard x86_64](https://www.alpinelinux.org/downloads/)
2. Write it to a USB using `dd` or `rufus`
3. Reboot to your boot menu and boot to Alpine on the USB

## Setup Root Partition

1. Login using login: `root`, password: (none)
2. Run `setup-alpine`
  1. Diskless install
  2. Store configs in default
  3. Store cache in default
3. Run the following commands
  1. `mkdir /media/trueroot`
  2. `apk add util-linux nano` (nano is optional if you're comfortable with vi)
  3. `lsblk` and note what partition root (`/`) is running from on your USB
  4. `blkid | grep /dev/{?0} >> /etc/fstab` where `{?0}` is the currently booted partition,
4. Change your `fstab` to use the uuid and mount it to `/media/trueroot`
  1. Run `nano /dev/fstab`
  2. Replace the line that contains your root partition with the UUID at the bottom line to creating the following:
  ```UUID={?m1-own-uu1d} /media/trueroot vfat defaults,noatime 0 0```
    where `{?m1-own-uu1d}` is the UUID on the last line (added with grep)
  3. Press `ctrl`+`o`, followed by `ctrl`+`x` to save and exit
5. Run the following commands to save and reboot:
  1. `lbu ci`
  2. `reboot`
  
## Setup Home Partition

1. Update your repositories and apkcache
  1. Run `nano /etc/apk/repositories`
  2. Change top line to `/media/trueroot/apks`
  3. Uncomment `edge/main` and `edge/community` by removing the `#`
  4. Commount out everything else by addting a `#` at the beginning of the line
  5. Press `ctrl`+`o`, followed by `ctrl`+`x` to save and exit
2. Partition your USB
  1. Run `apk add cfdisk e2fsprogs dosfstools nano util-linux`
  2. Create home partition (can also create a shared partition to act like a regular USB`
  3. Set home partition of type `Linux home`
  4. Select `Write`
  5. Select `Yes`
  6. Exit
3. Format your partitions
  1. Run `mkfs.ext4 /dev/{sd??}` where `{sd??}` is the home partition created in the step above
  2. (OPTIONAL, shared partition only) Run `mkfs.fat -F32 /dev/{sd??}` where `{sd??}` is the shared partition
4. Update your fstab to mount your home partition to `/home`
  1. Run `blkid | grep /dev/{?sd} >> /etc/fstab` where `{?sd}` is your home partition created above
  2. Run `nano /etc/fstab`
  3. Change the bottom line to the following:
  ```UUID={?m1-h0m3-uu1d} /home ext4 defaults,noatime 0 0``` where `{?m1-h0m3-uu1d}` is the UUID at the bottom line from grep
  4. Press `ctrl`+`o`, followed by `ctrl`+`x` to save and exit
5. Run the following commands to save and reboot:
  1. `lbu ci`
  2. `reboot`
  
## User Configuration
1. Run the following commands for a simple dwm setup
  1. `setup-xorg-base`
  2. `apk add sudo dmenu st dwm font-fira-mono-nerd firefox`
  3. `sed -i '/%wheel all=(all) all/s/^#//g' file` or simply uncomment `%wheel all=(all) all`
  4. `adduser main -G wheel`
  5. `su main`
  6. `echo 'exec dwm' >> ~/.xinitrc`
  7. `echo 'startx' >> ~/.profile`
  8. `sudo lbi ci`
  9. `startx`
