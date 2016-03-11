# Zynq Notes
Notes for settings up Zynq on Digilent Zybo

------------------------------------------------------------------------------------------------------

## How to partition an SD card for Zynq:

	1. Plug in micro SD card
	2. Identify micro SD card -> $ lsblk
	3. Unmount if necessary (if there are sdX1, sdX2 etc) -> $ sudo umount /media/<user>/<usb name>
	4. Create the partitions

	$ sudo fdisk /dev/sdX
	$ p
		$ d (delete all existing partitions)
	$ w
	$ n
		$ p
		$ 1
		$ <just press enter for default start location>
		$ +1G
	$ n
		$ p
		$ <enter>
		$ <enter>
		$ <enter>
	$ w
	$ sync
	$ sudo mkfs -t vfat -n ZYBO_BOOT /dev/sdc1
	$ sudo mkfs -t ext4 -L ROOT_FS /dev/sdc2
	$ sync

------------------------------------------------------------------------------------------------------

## Unmodified zynq_zybo.dts does not work

Note: My compiled devicetree file does not work without modififcation. One solution was to decompile device tree file
from other example project (http://www.instructables.com/files/orig/FAJ/VKWE/I7CCBSBN/FAJVKWEI7CCBSBN.zip) and
modifying as required.

To compile/decompile devicetree file/blob:

dtc -I <input type (dts:dtb)> -O <output type (dts:dtb)> -o <outputfile> <inputfile>

------------------------------------------------------------------------------------------------------

Note: If using digilent repositories, make sure to use branch 'master-next'

------------------------------------------------------------------------------------------------------

## To unpack the linaro filesystem onto the Zynq SD card:

mkdir –p Documents/sd_ext4
sudo mount /dev/sd<X>2 Documents/sd_ext4
cd <path>/linaro/binary/boot/filesystem.dir
sudo rsync -a ./ /home/digilent/Documents/sd_ext4

------------------------------------------------------------------------------------------------------

Standard kernel boot parameters (in devicetree.dts):

bootargs = "console=ttyPS0,115200 root=/dev/mmcblk0p2 rw earlyprintk rootfstype=ext4 rootwait devtmpfs.mount=1";

------------------------------------------------------------------------------------------------------

## To set the boot loader (u-boot.elf) to load the filesystem instead of the ramdisk, modify as such:

Before:

	"sdboot=if mmcinfo; then " \
				"run uenvboot; " \
				"echo Copying Linux from SD to RAM... && " \
				"fatload mmc 0 0x3000000 ${kernel_image} && " \
				"fatload mmc 0 0x2A00000 ${devicetree_image} && " \
				"fatload mmc 0 0x2000000 ${ramdisk_image} && " \
				"bootm 0x3000000 0x2000000 0x2A00000; " \
			"fi\0" \

After:

	"sdboot=if mmcinfo; then " \
				"run uenvboot; " \
				"echo Copying Linux from SD to RAM... && " \
				"fatload mmc 0 0x3000000 ${kernel_image} && " \
				"fatload mmc 0 0x2A00000 ${devicetree_image} && " \
				"bootm 0x3000000 - 0x2A00000; " \
			"fi\0" \

And then recompile:

	$ make CROSS_COMPILE=arm-xilinx-linux-gnueabi- clean

------------------------------------------------------------------------------------------------------

## Required environment variables for cross compilation:

export ARCH=arm
export CROSS_COMPILE=arm-xilinx-linux-gnueabi-
source ~/Applications/Xilinx/Vivado/Vivado/2015.4/settings64.sh
export PATH=~/Documents/Projects/Wanderings/Zybo/u_boot/u-boot-Digilent-Dev/tools:$PATH

------------------------------------------------------------------------------------------------------

Use minicom to get startup sequence as it does not require an active uart connection (it may initially 
but not for power on/power off)

sudo minicom -D /dev/ttyUSB<X>

------------------------------------------------------------------------------------------------------

## Required modifications to zync_zybo.dts to successfully boot and use load root filesystem from ROOT_FS partition:

Bootargs must be equal to:
	
bootargs = "console=ttyPS0,115200 root=/dev/mmcblk0p2 rw earlyprintk rootfstype=ext4 rootwait";
	linux,stdout-path = "/amba@0/serial@e0001000";
} ;

CPU clocks must be equal to:
	clocks = <&clkc 2>;