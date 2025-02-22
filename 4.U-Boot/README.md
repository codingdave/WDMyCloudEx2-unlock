# UBoot: The funny part

If you reach this part of the procedure, this is the funny part :smile:

Uboot is the firmware (software code) executed by the chip in order to load the kernel.
It is a quite good utility, altough the documentation is not the best asset.
You can find useful commands here: [Uboot User Manual](https://hub.digi.com/dp/path=/support/asset/u-boot-reference-manual/)

### Press `1`

If you want to have access to the Uboot procedure, you shall keep the key `1` pressed during the boot sequence. If that does not work you can also short the Tx and Rx lines of the WDMC.

It is important that you check the reliability of the serial connection as described in the [Section 1](../1.SerialCable/README.md)
```
Enable HD1
Enable HD2
Net:   egiga1
Warning: egiga1 MAC addresses don't match:
Address in SROM is         xx:xx:xx:xx:xx:xx
Address in environment is  xx:xx:xx:xx:xx:xx

Hit any key to stop autoboot:  0
Marvell>> 
```

### `bootcmd` and `bootargs`

The boot instruction is hardcoded in the `bootargs` and `bootcmd` variables; to see all variable use `printenv` command; to get help use `help`.
```
Marvell>> printenv bootargs
bootargs=root=/dev/ram console=ttyS0,115200 max_loop=32
Marvell>>
```
All changes you do on enviroment variable and loading new kernel here are not persistent, until you use `saveenv`; feel free to test your configuration.

### Loading your kernel from local hard disk

This is the siplest solution I found.

Plug your HD and create a small partition to store the uimage files; format it as FAT32 or Ext2.
Copy on this partition the uimage file [uImage-v5.10.109gs](uImage-v5.10.109gs) 
To enable the SATA connection issue the command:
```
Marvell>> ide reset

Reset IDE:
Marvell Serial ATA Adapter
Integrated Sata device found
  Device 0 @ 0 0:      <=== means second socket
Model: XXXXXXXXXXX                              Firm: 000XXXXX Ser#:             XXXXXXXX
            Type: Hard Disk
            Supports 48-bit addressing
            Capacity: 238475.1 MB = 232.8 GB (488397168 x 512)

Marvell>>
```
Assuming the partition is the first one, you can see it with `ext2ls` (for ext2) or `fatls` (for fat32):
```
Marvell>> ext2ls ide 0:1
<DIR>       1024 .
<DIR>       1024 ..
<DIR>      12288 lost+found
         4062561 uImage-v5.10.109gs
         4830114 uinitrd
             169 readme.txt
         4062565 uimage-5.10.109
Marvell>>

```

### Load the kernel with the command `extload` or `fatload`:
```
ext2load ide 0:1 0x500000 /uImage-v5.10.109gs 
```
Here the `0x500000` is the memory address to which you want uBoot to load your kernel. You can use also `0x2000000`
Be aware that this loading is not persisently wrote into the NAND.

### Boot the kernel:
```
Marvell>> bootm 0x500000
## Booting image at 00500000 ...
## Booting kernel from Legacy Image at 00500000 ...
   Image Name:   v5.10-github.com/gisab/WDMC-Ex2
   Created:      2022-04-03  10:54:59 UTC
   Image Type:   ARM Linux Kernel Image (uncompressed)
   Data Size:    4147193 Bytes = 4 MiB
   Load Address: 00008000
   Entry Point:  00008000
   Verifying Checksum ... OK
   Loading Kernel Image ... OK
OK

Starting kernel ...
[...]
```

### Make changes persistent

Tuning:
+ bootcmd
+ bootargs

Let's imagine you want to permanently load the image file `uimage` and `uinitrd` from your `sda1` and boot your preferred linux distribution from `sda2`.
Then you need to configure the variables in this way:
```
Marvell>> printenv bootcmd bootargs
bootcmd=ide reset ;ext2load ide 0:1 0x500000 /uimage ;ext2load ide 0:1 0xa00000 /uinitrd ; bootm 0x500000 0xa00000
bootargs=root=/dev/sda2 console=ttyS0,115200 max_loop=32 usbcore.autosuspend=-1
Marvell>>
```
In order to do so, you need first to set the variables (note the `\`) and then save them:
```
Marvell>> setenv bootcmd ide reset \; ext2load ide 0:1 0x500000 /uimage \;ext2load ide 0:1 0pa00000 /uinitrd \; bootm 0x500000 0xa00000
Marvell>> printenv bootcmd
bootcmd=ide reset ; ext2load ide 0:1 0x500000 /uimage ;ext2load ide 0:1 0pa00000 /uinitrd ; bootm 0x500000 0xa00000
Marvell>> setenv bootargs root=/dev/sda2 console=ttyS0,115200 max_loop=32 usbcore.autosuspend=-1
Marvell>> printenv bootargs
bootargs=root=/dev/sda2 console=ttyS0,115200 max_loop=32 usbcore.autosuspend=-1
Marvell>> saveenv
Saving Environment to NAND...
Erasing Nand...
Writing to Nand... done
Marvell>>
```
For the kernel `uImage-v5.10.109gs` the `uinitrd` is not necessary (but you can still provide it to uboot) and you can tune as follows:
```
Marvell>> setenv bootcmd ide reset \; ext2load ide 0:1 0x500000 /uImage-v5.10.109gs \; bootm 0x500000
Marvell>> printenv bootcmd
bootcmd=ide reset ; ext2load ide 0:1 0x500000 /uImage-v5.10.109gs ; bootm 0x500000
Marvell>> setenv bootargs root=/dev/sda2 console=ttyS0,115200 max_loop=32 usbcore.autosuspend=-1
Marvell>> printenv bootargs
bootargs=root=/dev/sda2 console=ttyS0,115200 max_loop=32 usbcore.autosuspend=-1
Marvell>> saveenv
Saving Environment to NAND...
Erasing Nand...
Writing to Nand... done
Marvell>>
```
