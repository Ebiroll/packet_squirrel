# Running packet squirrel software in qemu

The idea was to learn more about the packet squirrel, https://www.hak5.org/gear/packet-squirrel
'''
    Atheros AR9331 SoC at 400 MHz MIPS
    16 MB Onboard Flash
    64 MB DDR2 RAM
    2x 10/100 Ethernet Port
    USB 2.0 Host Port
    4-way payload select switch
    RGB Indicator LED
    Scriptable Push-Button
    Power: USB 5V 120mA average draw
    Dimensions: 50 x 39 x 16 mm
    Weight: 24 grams
'''

First i logged onto the packet squirel and did some information gathering, see the file info.txt [info.txt]

Then I checked out the the upgrade upgrade-1.2.bin file. The easiest way to do this is by renaming to upgrade-1.2.bin.7z
Thunar was able to explore this file without a problem. Symbolic inks are however lost. The /sbin/init file is the first process to be run, in order to see what it does i need to install the strace package.




# Dumping the filesystem
'''
On your host system make a dump of the
ssh root@172.16.32.1 "dd if=/dev/mtdblock2 " | dd of=block2.7z
After this you have a file that can later be recreated.

> file block2.7z 
block2: Squashfs filesystem, little endian, version 4.0, 13685232 bytes, 2386 inodes, blocksize: 262144 bytes, created: Fri Jul 14 00:58:36 2017

Create an image that can be used in qemu,
>qemu-img create -f raw openwrt-malta-be-root.ext4 4G

>mkfs.ext4 -F openwrt-malta-be-root.ext4

>losetup /dev/loop0 openwrt-malta-be-root.ext4
>mkdir /mnt/tmp
>mount  openwrt-malta-be-root.ext4  /mnt/tmp 
 
Extract all the files from the upgrade-1.2.bin.7z to your /mnt/tmp
If you do this fro thunar links are lost, there is probably a better way to do this.

> mkdir /tmp/block2
>sudo mount -t squashfs  block2.7z /tmp/block2
> cp -av /tmp/block2/. /mnt/tmp


'''

# build your own openwrt kernel, with vagrant
cd build
cd
>vagrant up
>vagrant ssh
>git clone https://github.com/hak5/wifipineapple-openwrt
>cd wifipineapple-openwrt
>make menuconfig
>cp /vagrant_data/.config .

The config file contains BE MIPS/Mach support to run on an
emulated 

When finnished you should  have a bin/malta directory,
to transfer them to host do
> cp bin/malta/openwrt-malta-be-vmlinux-initramfs.elf /vagrant_data

You can run qemu in vagrant or on your host system
sudo apt-get qemu-system


# Start qemu
qemu-system-mips -M malta -kernel openwrt-malta-be-vmlinux-initramfs.elf -hda openwrt-malta-le-root.ext4 -append "-root=hda console=ttyS0" -nographic
'''
BusyBox v1.23.2 (2018-01-18 12:24:13 UTC) built-in shell (ash)

  _______                     ________        __
 |       |.-----.-----.-----.|  |  |  |.----.|  |_
 |   -   ||  _  |  -__|     ||  |  |  ||   _||   _|
 |_______||   __|_____|__|__||________||__|  |____|
          |__| W I R E L E S S   F R E E D O M
 -----------------------------------------------------
 CHAOS CALMER (Chaos Calmer, r49610)
 -----------------------------------------------------
  * 1 1/2 oz Gin            Shake with a glassful
  * 1/4 oz Triple Sec       of broken ice and pour
  * 3/4 oz Lime Juice       unstrained into a goblet.
  * 1 1/2 oz Orange Juice
  * 1 tsp. Grenadine Syrup
 -----------------------------------------------------
'''

After booting to openWrt

>mkdir /mnt/ps
>mount /dev/hda /mnt/ps
When finished
>halt

mount  /dev/sda /mnt/ps

# Original openwrt configuration
You also can use this one to build qemu openwrt image
git clone -b chaos_calmer git://github.com/openwrt/chaos_calmer.git



# Setting up chroot
In order to run all the original binaries we dumped earlier.
'''
TARGETDIR="/mnt/ps"
mount -t proc proc $TARGETDIR/proc
mount -t sysfs sysfs $TARGETDIR/sys
mount -t devtmpfs devtmpfs $TARGETDIR/dev
mount -t tmpfs tmpfs $TARGETDIR/dev/shm
mount -t devpts devpts $TARGETDIR/dev/pts
'''
chroot /mn/ps


# Links
https://astr0baby.wordpress.com/2017/11/13/packet-squirrel-hands-on/

https://reverseengineering.stackexchange.com/questions/4817/dumping-firmware-thr
ough-mtdblock-device


https://web.archive.org/web/20160421212952/https://fosiao.com/content/running-openwrt-under-qemu

http://aleph0.info/software/lisp-forth-openwrt/openwrt-on-qemu.html


https://integriography.wordpress.com/2015/03/16/mounting-a-jffs2-dd-image-in-linux/


https://integriography.wordpress.com/2015/03/15/drone-forensics-an-overview/



# Here is some good info about memory technology devices

https://wiki.dave.eu/index.php/Memory_Tecnology_Device_(MTD)

https://free-electrons.com/blog/managing-flash-storage-with-linux/
