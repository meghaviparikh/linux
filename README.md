| kernel | normal | rt |
|:---------:|:--------:|:-------:|
|4.4  | [![Build Status](http://gfnd.rcn-ee.org:8080/buildStatus/icon?job=beagleboard_kernel_builder/4.4)](http://gfnd.rcn-ee.org:8080/job/beagleboard_kernel_builder/job/4.4/)   | [![Build Status](http://gfnd.rcn-ee.org:8080/buildStatus/icon?job=beagleboard_kernel_builder/4.4-rt)](http://gfnd.rcn-ee.org:8080/job/beagleboard_kernel_builder/job/4.4-rt/)   |
|4.9  | [![Build Status](http://gfnd.rcn-ee.org:8080/buildStatus/icon?job=beagleboard_kernel_builder/4.9)](http://gfnd.rcn-ee.org:8080/job/beagleboard_kernel_builder/job/4.9/)   | [![Build Status](http://gfnd.rcn-ee.org:8080/buildStatus/icon?job=beagleboard_kernel_builder/4.9-rt)](http://gfnd.rcn-ee.org:8080/job/beagleboard_kernel_builder/job/4.9-rt/)   |
|4.14 | [![Build Status](http://gfnd.rcn-ee.org:8080/buildStatus/icon?job=beagleboard_kernel_builder/4.14)](http://gfnd.rcn-ee.org:8080/job/beagleboard_kernel_builder/job/4.14/) | [![Build Status](http://gfnd.rcn-ee.org:8080/buildStatus/icon?job=beagleboard_kernel_builder/4.14-rt)](http://gfnd.rcn-ee.org:8080/job/beagleboard_kernel_builder/job/4.14-rt/) |
|4.19 | [![Build Status](http://gfnd.rcn-ee.org:8080/buildStatus/icon?job=beagleboard_kernel_builder/4.19)](http://gfnd.rcn-ee.org:8080/job/beagleboard_kernel_builder/job/4.19/) | [![Build Status](http://gfnd.rcn-ee.org:8080/buildStatus/icon?job=beagleboard_kernel_builder/4.19-rt)](http://gfnd.rcn-ee.org:8080/job/beagleboard_kernel_builder/job/4.19-rt/) |
|5.4  | [![Build Status](http://gfnd.rcn-ee.org:8080/buildStatus/icon?job=beagleboard_kernel_builder/5.4)](http://gfnd.rcn-ee.org:8080/job/beagleboard_kernel_builder/job/5.4/)   | [![Build Status](http://gfnd.rcn-ee.org:8080/buildStatus/icon?job=beagleboard_kernel_builder/5.4-rt)](http://gfnd.rcn-ee.org:8080/job/beagleboard_kernel_builder/job/5.4-rt/)   |



# Configuring U-BOOT with BBB:
Here I am going to be using the latest version of linux Kernel along with U-BOOT. Kindly follow through each step in order to successfully compile a verified U-BOOT and BBB Kernel.

Make sure to use a Virtual Machine that has atleast 30 GB of space available. While following the tutorial, kindly make a note of where you are executing the mentioned commands.

### Step 1: Configuring the Directory Structure and Environment Variables:
Let us assume that this is the directory structure you are following for your home directory.

```
root@kali:~# ls ~
new_kernel   work  u-boot
```

Issue  `nano ~/.bashrc` and copy the following Environmental variables in them. This will make things easier from here on. 

```
export CROSS_COMPILE=arm-linux-gnueabi-
export ARCH=arm
export UBOOT=/root/u-boot
export UOUT=$UBOOT/b/am335x_evm_defconfig
export KERNEL=/root/new_kernel
export OKERNEL=$KERNEL/arch/arm/boot
export WORK=/root/work
export PATH=$PATH:$UBOOT/tools
```

### Sub-step 1: Resolving the dependencies

Let's also resolve any dependency issue that might occur while compiling u-boot or kernel to make things smoother. Issue the following command to resolve all the dependencies

```
# apt-get install gcc-arm-linux-gnueabi binutils-arm-linux-gnueabi gcc-arm-linux-gnueabihf lzop libssl-dev bc bison flex device-tree-compiler
```

These are the basic dependencies that are needed, I may be forgetting something here but if you come accross an error, just google how to install them.


### Step 2: Let's compile the U-BOOT:

1. Download the latest U-BOOT Source code.

```
# cd $UBOOT
# git clone https://github.com/u-boot/u-boot.git .
```

2. Compile the U-BOOT for BBB

```  
# make am335x_evm_defconfig O=b/am335x_evm_defconfig
# make all O=b/am335x_evm_defconfig
```

3. Compile the tools,

We will need the mkimage utility that comes with u-boot later to create a Kernel image and also a signed image.

```
# make sandbox_defconfig tools-only
```

Now we have everything we need we need from U-BOOT, you can navigate to `b/am335x_boneblack_vboot/` to check if there is a `u-boot-dtb.img` file. Also navigate to `tools/` and confirm that there is an executable `mkimage`. 

```
# cd b/am335x_evm_defconfig/
# ls -l | grep "u-boot-dtb.img"
-rw-r--r--  1 root root  499141 Apr 27 07:03 u-boot-dtb.img
```


```
# cd tools
# ls -l | grep "mkimage"
-rwxr-xr-x 1 root root 308224 Apr 27 07:06 mkimage
-rw-r--r-- 1 root root  19186 Apr 27 06:43 mkimage.c
-rw-r--r-- 1 root root    965 Apr 27 06:43 mkimage.h
-rw-r--r-- 1 root root  23080 Apr 27 07:06 mkimage.o
```


If you don't have these, there is something wrong, and you should correctly follow the steps mentioned above before proceeding.

### Step 3: Compiling the linux Kernel

1. Download the Latest Linux Kernel

```
# cd $KERNEL
# git clone https://github.com/meghaviparikh/linux.git .
```

2. Compile the Linux Kernel for BBB

Here, I have used `-j 4` to speed up things. This simply means that allocate 4 computational cores to compiling the kernel. Depending upon your machine, edit the value of `-j` to speed up the process. 

```
# make bb.org_defconfig
# make uImage dtbs LOADADDR=80008000 -j 4
```

Navigate to `$OKERNEL` and make sure you have `Image` and `dts/am335x-boneblack.dtb` files

```
# cd $OKERNEL
# ls -l | grep "Image"
-rwxr-xr-x   1 root root 21022800 Apr 27 07:20 Image
-rw-r--r--   1 root root  9692832 Apr 27 07:25 uImage
-rwxr-xr-x   1 root root  9692768 Apr 27 07:20 zImage
# ls -l dts/ | grep "am335x-boneblack.dtb"
-rw-r--r-- 1 root root  60180 Apr 27 07:18 am335x-boneblack.dtb
```

If you have these, kindly proceed ahead. If you don't have these files, there is a chance that you might have done something wrong.

### Step 4: Preparing the Work Directory

Navigate to the Working directory.

```
# cd $WORK
```
Now create a file named `sign.its` and put the following contents in the file.

```
# nano sign.its
```

```
/dts-v1/;

/ {
	description = "Beaglebone black";
	#address-cells = <1>;

	images {
		kernel@1 {
			data = /incbin/("Image.lzo");
			type = "kernel";
			arch = "arm";
			os = "linux";
			compression = "lzo";
			load = <0x80008000>;
			entry = <0x80008000>;
			hash@1 {
				algo = "sha1";
			};
		};
		fdt@1 {
			description = "beaglebone-black";
			data = /incbin/("am335x-boneblack.dtb");
			type = "flat_dt";
			arch = "arm";
			compression = "none";
			hash@1 {
				algo = "sha1";
			};
		};
	};
	configurations {
		default = "conf@1";
		conf@1 {
			kernel = "kernel@1";
			fdt = "fdt@1";
			signature@1 {
				algo = "sha1,rsa2048";
				key-name-hint = "dev";
				sign-images = "fdt", "kernel";
			};
		};
	};
};
```

### Step 5: Creating the key-pair

```
# cd $WORK
# mkdir keys
# openssl genrsa -F4 -out keys/dev.key 2048
# openssl req -batch -new -x509 -key keys/dev.key -out keys/dev.crt
```

The dev.key is the private key here, so don't share it with anyone.


### Step 6: Signing the Kernel

This step depends on the success of previous steps. If the previous steps are not completed, kindly complete them and then proceed.

```
# ln -s $OKERNEL/dts/am335x-boneblack.dtb
# ln -s $OKERNEL/Image
# ln -s $UOUT/u-boot-dtb.img
# cp $UOUT/arch/arm/dts/am335x-boneblack.dtb am335x-boneblack-pubkey.dtb
# lzop Image
# $UOUT/tools/mkimage -f sign.its -K am335x-boneblack-pubkey.dtb -k keys -r image.fit
```

If you see the output something like this, you are on the right track!

```
FIT description: Beaglebone black
Created:         Mon Apr 27 07:32:06 2020
 Image 0 (kernel@1)
  Description:  unavailable
  Created:      Mon Apr 27 07:32:06 2020
  Type:         Kernel Image
  Compression:  lzo compressed
  Data Size:    12468420 Bytes = 12176.19 KiB = 11.89 MiB
  Architecture: ARM
  OS:           Linux
  Load Address: 0x80008000
  Entry Point:  0x80008000
  Hash algo:    sha1
  Hash value:   10e931a06395a5ff60b5b92bce7e2b901c009ade
 Image 1 (fdt@1)
  Description:  beaglebone-black
  Created:      Mon Apr 27 07:32:06 2020
  Type:         Flat Device Tree
  Compression:  uncompressed
  Data Size:    60180 Bytes = 58.77 KiB = 0.06 MiB
  Architecture: ARM
  Hash algo:    sha1
  Hash value:   c5ad10fe4f60bf7acb9262d58bbf5ec2fa0eb0b6
 Default Configuration: 'conf@1'
 Configuration 0 (conf@1)
  Description:  unavailable
  Kernel:       kernel@1
  FDT:          fdt@1
  Sign algo:    sha1,rsa2048:dev
```

Here you can confirm the Intigrity of the hash using:

```
# $UOUT/tools/fit_check_sign -f image.fit -k am335x-boneblack-pubkey.dtb
```

If you see something like this, everything is perfect till now! and you may proceed ahead.

```
   Loading Flat Device Tree
## Loading ramdisk from FIT Image at 7f8e78955000 ...
   Using 'conf@1' configuration
   Verifying Hash Integrity ... 
sha1,rsa2048:dev+ 
OK
```

### Step 7: Put the public key in U-Boot's image:

```
# cd $UBOOT
# make O=b/am335x_boneblack_vboot EXT_DTB=${WORK}/am335x-boneblack-pubkey.dtb

```


### Step 8: Put U-Boot and the kernel onto the board

Now, you are ready to install U-BOOT and Kernel on the board so partition your sd card into 2 volumes. The first volume should contain a minimum of 50 MB. I used the Windows Disk Mgmt to Partition them to make things easy.


When you insert your SD card, you should see something like this:


```
# lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda      8:0    0   80G  0 disk 
|-sda1   8:1    0   78G  0 part /
|-sda2   8:2    0    1K  0 part 
`-sda5   8:5    0    2G  0 part [SWAP]
sdb      8:16   0 15.2G  0 disk
├─sdb1   8:17   0   50M  0 part
└─sdb2   8:18   0 15.1G  0 part 
```
Here `sdb` is your sd card.  `sdb1` and `sdb2` are the partitiions where we are going to install `U-BOOT` and `KERNEL` respectively.


Open your `~/.bashrc` file and paste the following values below the Environment Variables we created in the step 1. 

```
export UDEV=/dev/sdb1
export KDEV=/dev/sdb2
```
Keep in mind that the smaller partition should come first as in pointed by `UDEV` and the larger one is pointed by `KDEV`.

Reload the profile:

```
source ~/.bashrc
```

Now, Issue the following commands to write the images created onto the SDCARD.

```
# sudo mount $UDEV /mnt/tmp && sudo cp $UOUT/u-boot-dtb.img /mnt/tmp/u-boot.img  && sudo cp $UOUT/MLO /mnt/tmp/MLO && sleep 1 && sudo umount $UDEV
# sudo mount $KDEV /mnt/tmp && sudo cp $WORK/image.fit /mnt/tmp/boot/image.fit && sleep 1 && sudo umount $KDEV
```

Congratulations, the SD card with verified U-BOOT is ready!
