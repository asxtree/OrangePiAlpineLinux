# OrangePiAlpineLinux


PRIOR DOING THIS A LINUX MACHINE (PHYSICAL/VM) IS NEEDED AS A REQUIREMENT!!!!!

And also u-boot tools installed like:
```  
  apt install u-boot-tools
```
# Clone git Repo

Logon your Linux machine and execute the following command to clone this git repo into folder `tmp_build`:
```
  git clone https://github.com/asxtree/OrangePiAlpineLinux tmp_build
```
Go in the `tmp_build` directory, create the `sources` directory and extract the `alpine_image.tar.gz` archive into sources:
```
  cd tmp_build
  mkdir sources
  cat alpine_image* > alpine_image.tar.gz
  tar -zxvf alpine_image.tar.gz -C sources
```
# Modify boot.cmd to match your board (the sources were made for running on Orange Pi PC2)

The most important thng that you must do is to modify the `boot.cmd` file to reflect the board in which you want to run the Alpine image. To do this go in the newly created folder where you've extracted the sources in the `/boot` directory like this:
```
  cd sources/boot
``` 
Remove the `boot.scr` file like this:
```
  rm -rf boot.scr
 ``` 
Edit the `boot.cmd` file to match your board you be using:
```
  vi boot.cmd (then press "i" on the keyboard to edit the file)
```   
And modify in the `boot.cmd` file only the second line with the bord that you want by deleting the current Device Tree Blob file `sun50i-h5-orangepi-pc2.dtb` and writing one of the availabe options like(prime is important in this case `sun50i-h5-orangepi-prime.dtb`):
   ```
   sun50i-a64-bananapi-m64.dtb
   sun50i-a64-nanopi-a64.dtb
   sun50i-a64-olinuxino.dtb
   sun50i-a64-orangepi-win.dtb
   sun50i-a64-pine64.dtb
   sun50i-a64-pine64-plus.dtb
   sun50i-a64-pinebook.dtb
   sun50i-a64-sopine-baseboard.dtb
   sun50i-h5-nanopi-m1-plus2.dtb
   sun50i-h5-nanopi-neo2.dtb
   sun50i-h5-nanopi-neo-plus2.dtb
   sun50i-h5-orangepi-pc2.dtb
   sun50i-h5-orangepi-prime.dtb
   sun50i-h5-orangepi-zero-plus.dtb
   sun50i-h5-orangepi-zero-plus2.dtb
```

```
  setenv bootargs earlyprintk /boot/vmlinuz-4.14.15-sunxi64 modules=loop,squashfs,sd-mod,usb-storage modloop=/boot/modloop-sunxi console=${console}
  fdt addr 0x50000000
  fdt get value bootargs /chosen bootargs
  setenv kernel_addr_r 0x41000000
  setenv ramdisk_addr_r 0x48000000
  load mmc 0:1 ${kernel_addr_r} boot/vmlinuz-4.14.15-sunxi64
  load mmc 0:1 ${ramdisk_addr_r} boot/initramfs-sunxi
  setenv initrdsize $filesize
  load mmc 0:1 ${fdt_addr_r} boot/dtb-4.14.15-sunxi64/allwinner/sun50i-h5-orangepi-pc2.dtb
  booti ${kernel_addr_r} ${ramdisk_addr_r}:${initrdsize} ${fdt_addr_r}
```

Leave the rest as it is and save by pressing the "ESC" key then write ":wq" and press the "ENTER" key.

# Make the boot.scr file

In order to be able to boot you have to recreate the `boot.scr` file from the `boot.cmd` file that you've modified, issue the following commad:
```
  mkimage -C none -A arm -T script -d boot.cmd boot.scr
```
Then go back to tmp_build:
```
  cd tmp_build
```
# Prepare the bootable SD card and copy the sources

Now insert the SD card into the Linux machine so we would prepare it for the image.
First list the disks to see if you see the SD card, you should have an entry like `Disk /dev/mmcblk0: 14.9 GiB`. Ifyou see it, issue this commands to write zero to it:
```
  fdisk -l #to list the disks and find the sd card, in my case is mmcblk0
  dd if=/dev/zero of=/dev/mmcblk0 bs=1M count=1
```  
Then write the u-boot on it:
```
  dd if=u-boot-sunxi-with-spl.bin of=/dev/mmcblk0 bs=1024 seek=8
```
And create a the first partition:
```
  fdisk /dev/mmcblk0 
 ``` 
 ```
root@ubuntu:tmp_build# fdisk /dev/mmcblk0

Welcome to fdisk (util-linux 2.27.1).

Changes will remain in memory only, until you decide to write them.

Be careful before using the write command.

Device does not contain a recognized partition table.

Created a new DOS disklabel with disk identifier 0xe5cd38f0.

Command (m for help): n   <-----------------------------------------------------------------> type "n" for new partition
Partition type

   p   primary (0 primary, 0 extended, 4 free) 
   
   e   extended (container for logical partitions)
   
Select (default p): p   <-------------------------------------------------------------------> type "p" for primary partition

Partition number (1-4, default 1):   <------------------------------------------------------> press ENTER key

First sector (2048-31116287, default 2048):   <---------------------------------------------> press ENTER key

Last sector, +sectors or +size{K,M,G,T,P} (2048-31116287, default 31116287): +1G

Created a new partition 1 of type 'Linux' and of size 1 GiB.

Command (m for help): <---------------------------------------------> type "t" and press ENTER

Selected partition 1

Partition type (type L to list all types): <---------------------------------------------> type "c" and press ENTER

Changed type of partition 'Linux' to 'W95 FAT32 (LBA)'.

Command (m for help): <---------------------------------------------> type "a" and press ENTER 

Selected partition 1

The bootable flag on partition 1 is enabled now.

Command (m for help): n <-----------------------------------------------------------------> type "n" for new partition

Partition type

   p   primary (1 primary, 0 extended, 3 free)
   
   e   extended (container for logical partitions)
   
Select (default p): p  <-------------------------------------------------------------------> type "p" for primary partition

Partition number (2-4, default 2):  <------------------------------------------------------> press ENTER key

First sector (2099200-31116287, default 2099200):  <------------------------------------------------------> press ENTER key

Last sector, +sectors or +size{K,M,G,T,P} (2099200-31116287, default 31116287):  <-------------------------------> press ENTER key

Created a new partition 2 of type 'Linux' and of size 13.9 GiB.

Command (m for help): w   <-----------------------------------------------------------------> type "w" to write changes

The partition table has been altered.

Calling ioctl() to re-read partition table.

Syncing disks.
```
Now format the partitions:
```
  mkfs.vfat /dev/mmcblk0p1
  mkfs.ext4 /dev/mmcblk0p2 <> confirm with "y" if asked
```  
Mount the first partition to /mnt:
```
  mount /dev/mmcblk0p1 /mnt
```
Copy the sources to the mounted partition:
```
  cp -R sources/* /mnt/
```  
After the copy is finished unmount the partition and dont try to force unmount for until it unmounts:
```
umount /mnt
```
# Remove the SD card an start the board

Now you should remove the SD card from the Linux machine and insert it in your board and after 25" you should already see the kernel starting on the connected monitor and in 30" you should have the login.

`If it doesnt start in more than 35" redo the steps from "Prepare the bootable SD card and copy the sources" assuming that youve made the corect boot.cmd and boot.scr, if not redo also those steps!!!!!!!!!!!!!!!!!!!`

# Alpine Linux persistent installation

At first run Alpine doesnt have any password so you will enter only the user root.
So the board started enter the user to login then lets issue the command to setup Alpine:
```
  localhost login: root
  setup-alpine
```
NOTICE: You will need a monitor and keyboard to do this and you can always type `?` during the setup to see other alternatives.

Set your keyboard preferences by typing `us` (there are many available keyboard layouts, this is a general example), then type `us-intl`.

Enter a name for your hostname then select the ethernet port that you wan to initialize (by default is `eth0` so just press ENTER), enter youre IP address that you want to be set on eth0 (by default is `DHCP`) if youve chossed DHCP you will be asked if you want to do any manual network configuration otherwise you will be asked for `gateway`, `netmask` and `DNS`.

Now is the time to change the root password and you will be asked to enter it two times for confirmation.

Select your timezone, you can type `?` to see them, for example for Athens type `Europe\Athens`.

You will be asked for `HTTP/FTP proxy URL` just select what is default `none`.

Select one from the available mirrors to update the repositories and here you can leave whats default and just press ENTER.

Select which SSH server you want to install, I usually install `OpenSSH` but you can choose watever you want and for NTP client leave the default `chrony`.

!!!!!!!!!!! PAY ATTENTION !!!!!!!!!!!!!!

When youre asked where to store the configs type `none` and press ENTER.
```
Enter where to store configs ('floppy', 'mmcblk0p1', 'usb' or 'none') [mmcblk0p1]: none
```
Also when asked to enter the apk cache directory type `none` and press ENTER.
```
Enter apk cache directory (or '?' or 'none') [/var/cache/apk]: none
```
To make a sys installation or persistent for the Alpine to run from the SD card and not from RAM we have to do the following. 

Install Alpine linux on the second partition, remove the boot folder from the installation on the second partition and point where the boot and root will be in the following commands:
```
  mount /dev/mmcblk0p2 /mnt
  setup-disk -m sys -k vanilla /mnt
  mount -o remount,rw /dev/mmcblk0p1
  cd /mnt
  rm -rf boot
  mkdir media/mmcblk0p1
  ln -s media/mmcblk0p1 boot
  echo "/dev/mmcblk0p1 /media/mmcblk0p1 vfat defaults 0 0" >> etc/fstab
  sed -i '/cdrom/d' etc/fstab
  sed -i '/floppy/d' etc/fstab
  sed -i '/edge/!s/^#//' etc/apk/repositories
  sed -i '/community/!s/^#//' etc/apk/repositories
  poweroff
```  
After the board was powered off remove the SD card from the board and put it back into the linux machine used earlier to write the sources to the card because we need to modify the `boot.cmd` file and redo the `boot.scr` file. 

To do so issue the following commands on the linux machine:
```
  fdisk -l #to see if the two partitions are visible
  mount /dev/mmcblk0p1 /mnt
  cd /mnt/boot/
  rm -rf boot.scr
  vi boot.cmd
```
Specify where the `root` partition will be `/dev/mmcblk0p2` right after `setenv bootargs root=/dev/mmcblk0p2` like in the next example:
```
setenv bootargs root=/dev/mmcblk0p2 earlyprintk /boot/vmlinuz-4.14.15-sunxi64 modules=loop,squashfs,sd-mod,usb-storage      modloop=/boot/modloop-sunxi console=${console}
fdt addr 0x50000000
fdt get value bootargs /chosen bootargs
setenv kernel_addr_r 0x41000000
setenv ramdisk_addr_r 0x48000000
load mmc 0:1 ${kernel_addr_r} boot/vmlinuz-4.14.15-sunxi64
load mmc 0:1 ${ramdisk_addr_r} boot/initramfs-sunxi
setenv initrdsize $filesize
load mmc 0:1 ${fdt_addr_r} boot/dtb-4.14.15-sunxi64/allwinner/sun50i-h5-orangepi-pc2.dtb
booti ${kernel_addr_r} ${ramdisk_addr_r}:${initrdsize} ${fdt_addr_r}
```
Create the `boot.scr` file and unmount the partition:
```
 mkimage -C none -A arm -T script -d boot.cmd boot.scr
 umount /mnt
```
Now put the SD card back in the board and power it up.
After you login on it type `df -h` to see that `/` is mounted on the second partition `/dev/mmcblk0p2` like in the next example:
```
Filesystem                Size      Used Available Use% Mounted on
devtmpfs                 10.0M         0     10.0M   0% /dev
shm                     497.6M         0    497.6M   0% /dev/shm
/dev/mmcblk0p2           13.5G    411.4M     12.4G   3% /
tmpfs                    99.5M    120.0K     99.4M   0% /run
cgroup_root              10.0M         0     10.0M   0% /sys/fs/cgroup
/dev/mmcblk0p1         1022.0M    251.2M    770.8M  25% /media/mmcblk0p1
``````
To get rid of the `hwclock` error and `fsck.ext4` failure on the second partition, type in the following commands:
```
  apk add e2fsprogs
  rc-update del hwclock boot
  rc-update add swclock boot
```

# Further Alpine Linux development

- [x] A version update will be made asap to address the hwclock error and the partion table.
- [ ] Compiling branch will be made with information on how to build your Alpine sources
- [ ] A prebuild persistent version update will be made
