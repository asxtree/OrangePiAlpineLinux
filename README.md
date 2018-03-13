# OrangePiAlpineLinux


PRIOR DOING THIS A LINUX MACHINE (PHYSICAL/VM) IS NEEDED AS A REQUIREMENT!!!!!

And also u-boot tools installed like:
```  
  apt install u-boot-tools
```
# Clone git Repo

Logon your Linux machine and execute the following command to clone this git repo into folder /build:
```
  git clone https://github.com/asxtree/OrangePiAlpineLinux /build
```
Go in the /build directory, create the sources directory and extract the alpine_image.tar.gz archive into sources:
```
  cd /build
  mkdir sources
  cat alpine_image* > alpine_image.tar.gz
  tar -zxvf alpine_image.tar.gz -C sources
```
# Modify boot.cmd to match your board (the sources were made for running on Orange Pi PC2)

The most important thng that you must do is to modify the boot.cmd file to reflect the board in which you want to run the Alpine image. To do this go in the newly created folder where you've extracted the sources in the /boot directory like this:
```
  cd sources/boot
``` 
Remove the boot.scr file like this:
```
  rm -rf boot.scr
 ``` 
Edit the boot.cmd file to match your board you be using:
```
  vi boot.cmd (then press "i" on the keyboard to edit the file)
```   
And modify second line with the bord that you want by deleting the current Device Tree Blob file "sun50i-h5-orangepi-pc2.dtb" and writing one of the availabe options like(prime is important in this case `sun50i-h5-orangepi-prime.dtb`):
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
> setenv bootargs earlyprintk /boot/vmlinuz-4.14.15-sunxi64 modules=loop,squashfs,sd-mod,usb-storage modloop=/boot/modloop-sunxi console=${console}

> fatload mmc 0:1 0x43000000 boot/dtb-4.14.15-sunxi64/allwinner/sun50i-h5-orangepi-pc2.dtb (should look like) /sun50i-h5-orangepi-prime.dtb

> fatload mmc 0:1 0x41000000 boot/vmlinuz-4.14.15-sunxi64

> fatload mmc 0:1 0x48000000 boot/initramfs-sunxi-new

> bootz 0x41000000 0x48000000 0x43000000'

Leave the rest as it is and save by pressing the "ESC" key then write ":wq" and press the "ENTER" key.

# Make the boot.scr file

In order to be able to boot you have to recreate the boot.scr file from the boot.cmd file that you've modified, issue the following commad:
```
  mkimage -C none -A arm -T script -d boot.cmd boot.scr
```
Then go back to /build:
```
  cd /build
```
# Prepare the bootable SD card and copy the sources

Now insert the SD card into the Linux machine so we would prepare it for the image.
First list the disks to see if you see the SD card, you should have an entry like "Disk /dev/mmcblk0: 14.9 GiB". Ifyou see it, issue this commands to write zero to it:
```
  dd if=/dev/zero of=/dev/mmcblk0 bs=1M count=1
```  
Then write the u-boot on it:
```
  dd if=u-boot-sunxi-with-spl.bin of=/dev/mmcblk0 bs=1024 seek=8
```
And create a new partition:
```
  fdisk /dev/mmcblk0 
 ``` 
 ```
root@ubuntu:/build# fdisk /dev/mmcblk0

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

Last sector, +sectors or +size{K,M,G,T,P} (2048-31116287, default 31116287):  <-------------> press ENTER key

Created a new partition 1 of type 'Linux' and of size 14.9 GiB.

Command (m for help): t   <-----------------------------------------------------------------> type "t"

Selected partition 1

Partition type (type L to list all types): 83

Changed type of partition 'Linux' to 'Linux'.

Command (m for help): w   <-----------------------------------------------------------------> type "w" to write changes

The partition table has been altered.

Calling ioctl() to re-read partition table.

Syncing disks.
```
Now format the partition:
```
  mkfs.ext4 /dev/mmcblk0p1 <> confirm with "y" if asked
```  
Mount the partition to /mnt:
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


# Further Alpine Linux development
TBD
