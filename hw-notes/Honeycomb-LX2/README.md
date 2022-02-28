# HoneyComb LX2

This document has some info about the [HoneyComb LX2 device, made by SolidRun](https://www.solid-run.com/arm-servers-networking-platforms/honeycomb-workstation/).
It's just some info I found for myself by working with this hardware.  It
describes some of the hardware features, and some of my experiences with
upgrading and working with it.

## Disclaimer

I am not a SolidRun employee.  This is just me writing about what I know, or
think I know, about a device I own.  If something breaks, that's a bummer, but
don't blame, flame or sue me.

## Intro

I have done some embedded work with ARM in the past, but that was a long time
ago, and 32 bits ago.  Nowadays, I have a NAS box with limited computing power
and was looking for something with a little more oomph, and was intrigued by
the prospect of a 16-core ARM board.  So I grabbed one.

It took a while to arrive, they had some production issues probably due to
global pandemics and such, but it's here and it's running and I'm pretty happy
with it.

I've been messing with 64-bit ARM a bit in Amazon AWS, but having a board right
in front of me is a different experience.  More tangible, literally.
This is also the first time I've messed with SFP-style network transceivers.

## Vendor Docs

SolidRun provides some useful guides on what's what and how to get things
working.  Their [Quick Start Guide](
https://solidrun.atlassian.net/wiki/spaces/developer/pages/197494288) worked
for me.

NXP has datasheets and such too, if you sign up for access.

## Hardware

### Serial console

The system doesn't boot, for me, without having the serial console plugged in.
It doesn't need to be plugged into a separate machine; plugging the console
cable into a USB port on the same Honeycomb board is sufficient.

I am just guessing here, but there is a fairly common problem where the FTDI
chip won't initialize properly when there isn't anything plugged in, and the
software running on the host machine (U-Boot in this case) will receive trash
data.  That trash may be enough to trigger the "Press a key to pause boot"
prompt.

### Lists of things

There were a few cases where the starter guides helped, but were unclear about
what the options were.  Here's what I've been able to figure out, by reading
through source code and schematics and such.

#### SW1

SW1 is a set of 5 DIP switches on the daughter board.  This controls the boot
device; the CPU will attempt to read a boot-loader from this device at the very
beginning of the boot process.  The individual switches are labeled from 1 to 5,
and the casing has an "ON" label, on the side closest to the edge of the
daughter board.

One of the switches is unused.  The other 4 switches are configured as pulldowns
to the CPU's CFG_RCW_SRC* pins.  That means, when the switch is "on", the config
pin is connected to ground, and will read as a zero.  So in the below tables,
when it says "0", the switch should look "ON" as you look at it.

The default is 1000.  Physically, that looks like OFF, ON, ON, ON, and whatever
(we ignore the 5th one).  This setting will tell it to boot from the SD card.
Once you follow their guide and install an image to eMMC, you can toggle the
4th switch, resulting in 1001 (OFF, ON, ON, OFF).  And if you mess up, you
can fix it by switching back to 1000 and booting from the microSD slot.

I found tables of possible SW1 values in a few places, including SolidRun's
daughter board schematic and a couple of NXP documents.

The most complete list is in the CPU datasheet:

* 00xx: Hard-coded RCW (reference to section "RCW Settings for Hard-Coded Options")
* 1000: SDHC1: SD card
* 1001: SDHC2: eMMC
* 1010: I2C boot EEPROM
* 1011: Reserved
* 1100: XSPI1_A: FlexSPI Serial NAND 2 KB pages
* 1101: XSPI1_A: FlexSPI Serial NAND 4 KB pages
* 1110: Reserved
* 1111: XSPI1_A: FlexSPI Serial NOR 24-bit Addressing

Of these, 1000 and 1001 are the most useful for this board.  But if you
were wondering what else was possible, there it is.


#### SFP modules

The Honeycomb LX2 board has 4 10Gbit SFP+ ports, arranged in a 2x2 grid.  There
is an upgraded motherboard which also has a QSFP port; I don't have that
version.  But I can see that there's space left for it in the hardware design.

These ports are handled by a SoC hardware feature called DPAA2, which is
configured with a command-line tool called `restool`.  It is also possible to
bake a configuration into your device tree file; I have not done this.

For info on restool, see the software section.

The hardware has several software MACs which can be mapped to OS interfaces.
As far as I can tell, here is the mapping of MAC IDs to hardware:

* dpmac.1 directly using the QSFP28 port
* dpmac.2 connecting QSFP28 to a 40G to 4x10G splitter cable (I think)
* dpmac.3: connect a 40G to 4x10G splitter to the (non-populated) QSFP28: split port 0
* dpmac.4: connect a 40G to 4x10G splitter to the (non-populated) QSFP28: split port 1
* dpmac.5: connect a 40G to 4x10G splitter to the (non-populated) QSFP28: split port 2
* dpmac.6: connect a 40G to 4x10G splitter to the (non-populated) QSFP28: split port 3
* dpmac.7: SFP+ port in the 2x2 cage, upper row, right towards PCB middle
* dpmac.8: SFP+ port in the 2x2 cage, lower row, right towards PCB middle
* dpmac.9: SFP+ port in the 2x2 cage, upper row, left towards PCB edge
* dpmac.10: SFP+ port in the 2x2 cage, lower row, left towards PCB edge

I'm not sure what exactly is the difference between #2 and #3-#6, I don't have
QSFP28.  Feel free to let me know if you have more info.

### My build

I've got it set up in a Fractal Design Core 500 mini-ITX case with a Phanteks
550W power supply and some Noctua fans.

I'm currently using 64GB of 2600MHz RAM.  I tried 3200MHz, but found it to be
unreliable; only one of the two slots would be detected properly.

The stock fan was loud and annoying, so I replaced it with a tall heat sink
(with a northbridge-sized base) with a 40mm fan stuck on one side.

## Software

### Kernel

The kernel is built from [the `linux-5.15.y-cex7` branch of SolidRun's kernel
fork](https://github.com/SolidRun/linux-stable/tree/linux-5.15.y-cex7).  It is
working reasonably well.  I did have to add `arm-smmu.disable_bypass=0` to the
kernel command line when I updated from the default 4.19 kernel that the sdcard
image uses.

Here is the script I use to build from these sources:

```sh
mv -f ../*.{gz,dsc,buildinfo,changes,deb} ../old/
if test -f .version; then
    perl -e 'print(<> + 1)' .version > .version.new && mv .version.new .version
else
    echo 1 > .version
fi

rm -f vmlinux-gdb.py

make oldconfig
make deb-pkg -j31

prefix=/home/infinoid/.local make -C tools perf_install
```

This makes .deb packages for the kernel itself, and installs the `perf` tool
into `$HOME/.local/bin`.

### Restool

I built this from [restool sources in the qoriq git repo](
https://source.codeaurora.org/external/qoriq/qoriq-components/restool).
It's just one binary and some shell scripts.

I have this simple script running at boot time, it adds the SFP+ ports and
gives them nice names:

```bash
#!/bin/sh

ls-addni dpmac.7
ls-addni dpmac.8
ls-addni dpmac.9
ls-addni dpmac.10
ip link set name ens0v0 dev eth1
ip link set name ens0v1 dev eth2
ip link set name ens0v2 dev eth3
ip link set name ens0v3 dev eth4
```

### Disks / boot

I am currently booting from eMMC, and using the default ubuntu installation as
a rescue image and as a /boot folder.

I have Debian Bullseye installed on NVMe, and I have some terrible half-written
scripts that know where to put newly installed kernel images.

## Output snippets

These were taken after booting Debian Bullseye with a 5.15.10 kernel.

### lscpu

```
# lscpu
Architecture:            aarch64
  CPU op-mode(s):        32-bit, 64-bit
  Byte Order:            Little Endian
CPU(s):                  16
  On-line CPU(s) list:   0-15
Vendor ID:               ARM
  Model name:            Cortex-A72
    Model:               3
    Thread(s) per core:  1
    Core(s) per cluster: 16
    Socket(s):           -
    Cluster(s):          1
    Stepping:            r0p3
    CPU max MHz:         2000.0000
    CPU min MHz:         900.0000
    BogoMIPS:            50.00
    Flags:               fp asimd evtstrm aes pmull sha1 sha2 crc32 cpuid
Caches (sum of all):
  L1d:                   512 KiB (16 instances)
  L1i:                   768 KiB (16 instances)
  L2:                    8 MiB (8 instances)
NUMA:
  NUMA node(s):          1
  NUMA node0 CPU(s):     0-15
Vulnerabilities:
  Itlb multihit:         Not affected
  L1tf:                  Not affected
  Mds:                   Not affected
  Meltdown:              Not affected
  Spec store bypass:     Not affected
  Spectre v1:            Mitigation; __user pointer sanitization
  Spectre v2:            Mitigation; Branch predictor hardening
  Srbds:                 Not affected
  Tsx async abort:       Not affected
```

### lspci

The NVMe drive is something I added, it didn't come with the board.

```
# lspci
0000:00:00.0 PCI bridge: Freescale Semiconductor Inc Device 8d80 (rev 20)
0000:01:00.0 Non-Volatile memory controller: Samsung Electronics Co Ltd NVMe SSD Controller SM981/PM981/PM983
0001:00:00.0 PCI bridge: Freescale Semiconductor Inc Device 8d80 (rev 20)
```

### lsblk

Again, the NVMe drive is something I added, it didn't come with the board.

```
# lsblk
NAME         MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
mmcblk1      179:0    0 59.2G  0 disk
└─mmcblk1p1  179:1    0 59.2G  0 part /rescue
nvme0n1      259:0    0  1.8T  0 disk
├─nvme0n1p1  259:1    0  500G  0 part /var/lib/containers/storage/overlay
│                                     /
├─nvme0n1p2  259:2    0   20G  0 part [SWAP]
└─nvme0n1p3  259:3    0  1.3T  0 part
mmcblk1boot0 179:256  0    4M  1 disk
mmcblk1boot1 179:512  0    4M  1 disk
```

### lsusb

```
# lsusb
Bus 004 Device 002: ID 04b4:6500 Cypress Semiconductor Corp.
Bus 004 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
Bus 003 Device 003: ID 0424:2514 Microchip Technology, Inc. (formerly SMSC) USB 2.0 Hub
Bus 003 Device 002: ID 04b4:6502 Cypress Semiconductor Corp. CY4609
Bus 003 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
Bus 002 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
```

### U-Boot

This is the output from U-Boot on the console port when booting the default Ubuntu from eMMC.
```
NOTICE:  BL2: v1.5(release):LSDK-20.12-8-g9115785
NOTICE:  BL2: Built : 19:30:19, Jun 15 2021
NOTICE:  UDIMM 18ASF4G72HZ-2G6B1
NOTICE:  DDR4 UDIMM with 2-rank 64-bit bus (x8)

NOTICE:  64 GB DDR4, 64-bit, CL=19, ECC on, 256B, CS0+CS1
NOTICE:  BL2: Booting BL31
NOTICE:  BL31: v1.5(release):LSDK-20.12-8-g9115785
NOTICE:  BL31: Built : 19:30:19, Jun 15 2021
NOTICE:  Welcome to LX2160 BL31 Phase


U-Boot 2020.04-00026-gbc620478 (Jun 15 2021 - 19:30:17 +0000)

SoC:  LX2160ACE Rev2.0 (0x87360020)
Clock Configuration:
       CPU0(A72):2000 MHz  CPU1(A72):2000 MHz  CPU2(A72):2000 MHz
       CPU3(A72):2000 MHz  CPU4(A72):2000 MHz  CPU5(A72):2000 MHz
       CPU6(A72):2000 MHz  CPU7(A72):2000 MHz  CPU8(A72):2000 MHz
       CPU9(A72):2000 MHz  CPU10(A72):2000 MHz  CPU11(A72):2000 MHz
       CPU12(A72):2000 MHz  CPU13(A72):2000 MHz  CPU14(A72):2000 MHz
       CPU15(A72):2000 MHz
       Bus:      700  MHz  DDR:      2600 MT/s
Reset Configuration Word (RCW):
       00000000: 506b6b38 24500050 00000000 00000000
       00000010: 00000000 0e010000 00000000 00000000
       00000020: 0fc001a0 00002580 00000000 08000086
       00000030: 09240000 00000001 00000000 00000000
       00000040: 00000000 00000000 00000000 00000000
       00000050: 00000000 00000000 00000000 00000000
       00000060: 00000000 00000000 00027000 00000000
       00000070: 08a80001 00151020
Model: SolidRun LX2160ACEX7 COM express type 7 based board
Board: LX2160ACE Rev2.0-CEX7, eMMC
SERDES1 Reference: Clock1 = 161.13MHz Clock2 = 100MHz
SERDES2 Reference: Clock1 = 100MHz Clock2 = 100MHz
SERDES3 Reference: Clock1 = 100MHz Clock2 = 100Hz
DRAM:  63.9 GiB
DDR    63.9 GiB (DDR4, 64-bit, CL=19, ECC on)
       DDR Controller Interleaving Mode: 256B
       DDR Chip-Select Interleaving Mode: CS0+CS1
WDT:   Started with servicing (30s timeout)
Using SERDES1 Protocol: 8 (0x8)
Using SERDES2 Protocol: 5 (0x5)
Using SERDES3 Protocol: 2 (0x2)
MMC:   FSL_SDHC: 0, FSL_SDHC: 1
Loading Environment from MMC... *** Warning - bad CRC, using default environment

In:    serial_pl01x
Out:   serial_pl01x
Err:   serial_pl01x
Net:   EEPROM: TlvInfo v1 len=88
PCIe1: pcie@3400000 disabled
PCIe2: pcie@3500000 disabled
PCIe3: pcie@3600000 Root Complex: x4 gen3
PCIe4: pcie@3700000 disabled
PCIe5: pcie@3800000 Root Complex: no link
PCIe6: pcie@3900000 disabled
DPMAC3@xgmii, DPMAC4@xgmii, DPMAC5@xgmii, DPMAC6@xgmii, DPMAC7@xgmii, DPMAC8@xgmii, DPMAC9@xgmii, DPMAC10@xgmii, DPMAC17@rgmii-id [PRIME], DPMAC18@rgmii-id
switch to partitions #0, OK
mmc1(part 0) is current device

MMC read: dev # 1, block # 20480, count 4608 ... 4608 blocks read: OK

MMC read: dev # 1, block # 28672, count 2048 ... 2048 blocks read: OK
crc32+
fsl-mc: Booting Management Complex ... SUCCESS
fsl-mc: Management Complex booted (version: 10.24.0, boot status: 0x1)
Hit any key to stop autoboot:  0
switch to partitions #0, OK
mmc1(part 0) is current device
Device: FSL_SDHC
Manufacturer ID: 45
OEM: 100
Name: DG406
Bus Speed: 50000000
Mode: MMC High Speed (52MHz)
Rd Block Len: 512
MMC version 5.1
High Capacity: Yes
Capacity: 59.2 GiB
Bus Width: 4-bit
Erase Group Size: 512 KiB
HC WP Group Size: 8 MiB
User Capacity: 59.2 GiB WRREL
Boot Capacity: 4 MiB ENH
RPMB Capacity: 4 MiB ENH

MMC read: dev # 1, block # 26624, count 2048 ... 2048 blocks read: OK
starting USB...
Bus usb3@3100000: Register 200017f NbrPorts 2
Starting the controller
USB XHCI 1.00
Bus usb3@3110000: Register 200017f NbrPorts 2
Starting the controller
USB XHCI 1.00
scanning bus usb3@3100000 for devices... 1 USB Device(s) found
scanning bus usb3@3110000 for devices... 4 USB Device(s) found
       scanning usb for storage devices... 0 Storage Device(s) found

Device 0: unknown device
MMC: no card present
switch to partitions #0, OK
mmc1(part 0) is current device
Scanning mmc 1:1...
Found /extlinux/extlinux.conf
Retrieving file: /extlinux/extlinux.conf
348 bytes read in 12 ms (28.3 KiB/s)
1:      primary kernel
Retrieving file: /boot/Image
36954624 bytes read in 1718 ms (20.5 MiB/s)
append: console=ttyAMA0,115200 earlycon=pl011,mmio32,0x21c0000 default_hugepagesz=1024m hugepagesz=1024m hugepages=2 pci=pcie_bus_perf root=PARTUUID=30303030-01 rw rootwait
Retrieving file: /boot/fsl-lx2160a-cex7.dtb
28471 bytes read in 18 ms (1.5 MiB/s)
## Flattened Device Tree blob at 81000000
   Booting using the fdt blob at 0x81000000
   Loading Device Tree to 000000009fff6000, end 000000009fffff36 ... OK
Releasing fan controller full speed gpio
fsl-mc: Deploying data path layout ... WARNING: Firmware returned an error (GSR: 0x3f)

Starting kernel ...
```
