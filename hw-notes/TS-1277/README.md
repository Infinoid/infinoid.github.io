# TS-1277

This document has some info about the [TS-1277 NAS device, made by QNAP](https://www.qnap.com/en-us/product/ts-1277).
Just some info I found for myself by working with this hardware.  It describes some
of the hardware features, and some of my experiences with upgrading and working
with it.

## Disclaimer

I am not a QNAP employee.  This is just me writing about what I know, or think
I know, about a device I own.  If something breaks, that's a bummer, but don't
blame, flame or sue me.

## Intro

I have gone through a few rounds of buying QNAP devices and installing some
variant of Debian on it.  I like QNAP hardware, as it tends to be small, quiet
and standardized.  I'm not interested in their software; I usually don't even
boot it once.  I'm taking the same approach with this one.

QTS may be great; I wouldn't know.  I'm running xubuntu on mine.

## Serial ports

The device has two old-school rs232 serial ports.

### serial console

The first serial port, ttyS0, is used as a serial console.  It is exposed as a
3.5mm connector on the back panel, just above USB port #2.  The BIOS and QTS
run this port at 115200bps, 8n1.  The OS writes console output to this serial
port, and the BIOS allows configuration through it.

The port uses normal rs232 voltage levels.  Most USB-to-3.5mm cables I saw use
TTL voltage levels, which won't work.  If you don't have a video card, you'll
need to find one that uses normal rs232 voltage levels.
[This one](https://www.amazon.com/gp/product/B07XXWVH69) works.

### lcd

The second serial port, ttyS1, is connected to the front panel LCD.  This LCD
is similar to some other QNAP products.
[Here is info on TS-439 Pro](https://forum.qnap.com/viewtopic.php?t=19180)
which seems valid for TS-1277 too.

If you replace QTS with a normal linux distribution, you will find that after
it boots, the LCD remains lit with a booting message.  You can
[turn it off or update the message by writing commands to ttyS1.](https://forum.qnap.com/viewtopic.php?t=19180#p244813)

## Booting

### BIOS

The BIOS is set to "quiet" mode by default, but it does try to "clear" the
screen by writing a bunch of spaces to the serial console.  You can hit Del or
F2 to get into the BIOS configuration menu, around the time when the LCD says
it is initializing hardware.  This is 15 to 20 seconds after the device powers
on.  If you turn off quiet mode in the BIOS, it will write a banner to both
the serial console and the video card (if you have one).  This makes it easier
to see when to press Del or F2.  There's also a setting to turn off the boot
beep, if it annoys you.

I recommend turning off "quiet" mode and getting used to the sequence of LCD
messages *before* you try any upgrades.  This will make later problems easier
to diagnose if/when they occur.

If your memory is bad, you won't be able to get into the BIOS, and the LCD
will loop through its messages once every 20 to 30 seconds.  For me it said
"Booting" and then went back to "Checking memory".  The "Booting" was a lie,
it didn't really get that far.

No BIOS password is configured by default.  Secure boot is off by default.

### Boot disk

The BIOS is able to boot debian, ubuntu and memtest86 USB sticks just fine,
in either MBR or EFI mode.  It does not do so by default, and I am not aware
of a specific key to pop up a boot menu.  But if you go into the main BIOS
configuration menu with Del or F2, the boot override section in the right-most
tab lets you select the drive to boot from.

The EFI BIOS does not seem to be able to see SATA drives at all.  I was not
able to find a BIOS setting that affects this.  I guess this is so that QTS
boots reliably, without interference from any user drives that may be attached.

There is a small, internal 4GB SSD inside of the NAS, connected via USB.
(If you remove the CPU blower fans, you can see a small square-ish board
immediately above the M.2 slots.  That's the drive.)  This is the boot drive,
and it's where QTS lives.  It uses the MBR paritioning scheme, with grub to
select between two xen dom0 images.

If you want to replace QTS and install a stock linux distribution, you can use
the 4GB SSD for your /boot partition, and install grub onto that.  I personally
left it MBR-style, but EFI should probably work too.  The EFI System partition
will need to live on the USB drive too.  Either way, the idea is that the
kernel and initramfs will start from USB, detect your SATA drives, then mount
the root filesystem and continue on as normal.

The normal caveats apply: it's a normal x86 boot process, and if you get it
wrong, it's up to you to fix it.  There's already an OS on there, and if you
want to keep access to that, you'd better be gentle with it, and keep good
backups.  The only real difference about the TS-1277 is that SATA drives aren't
visible, you are forced to boot from USB.

## video card

This NAS has no video card by default.  It has one full-height, double-width
PCIe slot for a video card, so you can install one, but I would not get too
ambitious about this.  The clearance would be pretty tight for some of the
larger high-end video cards that have spiky plastic pointing everywhere, I
don't see any secondary power connectors, and I think a heavy thermal load
would cause problems.

I personally don't plan to use the video output at all, but I do want to run
occasional CUDA tasks.  So I got
[a small, passively-cooled card](https://www.amazon.com/gp/product/B01AZ7W88O/),
which is working great for me.

## RAM

At time of writing, the high end 64GB memory configuration is about $1000 more
expensive than the 16GB configuration. It is a lot less expensive to get the
16GB version, and buy 64GB of memory and install it yourself.

However, the NAS seems pretty picky about memory.

This [Crucial DDR4 2400MHz dual rank memory](https://www.amazon.com/dp/B019FRBCQE)
worked for me.

This [Corsair DDR4 2400MHz memory](https://www.amazon.com/dp/B01K3LTX8U)
did NOT work for me.

I'm not sure why, exactly.  Maybe the "dual rank" part is important?

When in doubt, ask QNAP.  Their website has big fat warnings about memory
module compatibility, and a (very short) list of memory modules they approve.
My memory wasn't on the list, so it may explode next week.

## Warranty/Support

I dunno know exactly what kinds of modifications are smiled or frowned upon by
QNAP.  I noticed their [Warranty FAQ](https://www.qnap.com/en-us/service_ask/)
expressly disclaims any warranty for software/firmware.  I think they will try
to support their hardware no matter what you run on it, but you will probably
get a better response if your setup seems reasonable.

And you'll probably get a better response if you don't stab the piezo speaker
repeatedly with a philips screwdriver until it can beep no more.  Not that I've
done that.

# info dumps

These dumps came from booting ubuntu 20.04 (without xen/QTS).

## lscpu

    Architecture:                    x86_64
    CPU op-mode(s):                  32-bit, 64-bit
    Byte Order:                      Little Endian
    Address sizes:                   43 bits physical, 48 bits virtual
    CPU(s):                          16
    On-line CPU(s) list:             0-15
    Thread(s) per core:              2
    Core(s) per socket:              8
    Socket(s):                       1
    NUMA node(s):                    1
    Vendor ID:                       AuthenticAMD
    CPU family:                      23
    Model:                           1
    Model name:                      AMD Ryzen 7 PRO 1700 Eight-Core Processor
    Stepping:                        1
    Frequency boost:                 enabled
    CPU MHz:                         1551.490
    CPU max MHz:                     3000.0000
    CPU min MHz:                     1550.0000
    BogoMIPS:                        5988.13
    Virtualization:                  AMD-V
    L1d cache:                       256 KiB
    L1i cache:                       512 KiB
    L2 cache:                        4 MiB
    L3 cache:                        16 MiB
    NUMA node0 CPU(s):               0-15
    Vulnerability Itlb multihit:     Not affected
    Vulnerability L1tf:              Not affected
    Vulnerability Mds:               Not affected
    Vulnerability Meltdown:          Not affected
    Vulnerability Spec store bypass: Mitigation; Speculative Store Bypass disabled via prctl and seccomp
    Vulnerability Spectre v1:        Mitigation; usercopy/swapgs barriers and __user pointer sanitization
    Vulnerability Spectre v2:        Mitigation; Full AMD retpoline, IBPB conditional, STIBP disabled, RSB filling
    Vulnerability Srbds:             Not affected
    Vulnerability Tsx async abort:   Not affected
    Flags:                           fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 ht syscall nx mmxext fxsr_opt
                                      pdpe1gb rdtscp lm constant_tsc rep_good nopl nonstop_tsc cpuid extd_apicid aperfmperf pni pclmulqdq monitor ssse3 fma cx16 sse
                                     4_1 sse4_2 movbe popcnt aes xsave avx f16c rdrand lahf_lm cmp_legacy svm extapic cr8_legacy abm sse4a misalignsse 3dnowprefetch
                                      osvw skinit wdt tce topoext perfctr_core perfctr_nb bpext perfctr_llc mwaitx cpb hw_pstate sme ssbd sev ibpb vmmcall fsgsbase 
                                     bmi1 avx2 smep bmi2 rdseed adx smap clflushopt sha_ni xsaveopt xsavec xgetbv1 xsaves clzero irperf xsaveerptr arat npt lbrv svm
                                     _lock nrip_save tsc_scale vmcb_clean flushbyasid decodeassists pausefilter pfthreshold avic v_vmsave_vmload vgif overflow_recov
                                      succor smca

## lspci

    00:00.0 Host bridge: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-0fh) Root Complex
    00:00.2 IOMMU: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-0fh) I/O Memory Management Unit
    00:01.0 Host bridge: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-1fh) PCIe Dummy Host Bridge
    00:01.1 PCI bridge: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-0fh) PCIe GPP Bridge
    00:01.2 PCI bridge: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-0fh) PCIe GPP Bridge
    00:01.3 PCI bridge: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-0fh) PCIe GPP Bridge
    00:01.4 PCI bridge: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-0fh) PCIe GPP Bridge
    00:02.0 Host bridge: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-1fh) PCIe Dummy Host Bridge
    00:03.0 Host bridge: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-1fh) PCIe Dummy Host Bridge
    00:03.1 PCI bridge: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-0fh) PCIe GPP Bridge
    00:03.2 PCI bridge: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-0fh) PCIe GPP Bridge
    00:03.3 PCI bridge: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-0fh) PCIe GPP Bridge
    00:03.4 PCI bridge: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-0fh) PCIe GPP Bridge
    00:04.0 Host bridge: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-1fh) PCIe Dummy Host Bridge
    00:07.0 Host bridge: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-1fh) PCIe Dummy Host Bridge
    00:07.1 PCI bridge: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-0fh) Internal PCIe GPP Bridge 0 to Bus B
    00:08.0 Host bridge: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-1fh) PCIe Dummy Host Bridge
    00:08.1 PCI bridge: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-0fh) Internal PCIe GPP Bridge 0 to Bus B
    00:14.0 SMBus: Advanced Micro Devices, Inc. [AMD] FCH SMBus Controller (rev 59)
    00:14.3 ISA bridge: Advanced Micro Devices, Inc. [AMD] FCH LPC Bridge (rev 51)
    00:18.0 Host bridge: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-0fh) Data Fabric: Device 18h; Function 0
    00:18.1 Host bridge: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-0fh) Data Fabric: Device 18h; Function 1
    00:18.2 Host bridge: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-0fh) Data Fabric: Device 18h; Function 2
    00:18.3 Host bridge: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-0fh) Data Fabric: Device 18h; Function 3
    00:18.4 Host bridge: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-0fh) Data Fabric: Device 18h; Function 4
    00:18.5 Host bridge: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-0fh) Data Fabric: Device 18h; Function 5
    00:18.6 Host bridge: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-0fh) Data Fabric: Device 18h; Function 6
    00:18.7 Host bridge: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-0fh) Data Fabric: Device 18h; Function 7
    01:00.0 SATA controller: ASMedia Technology Inc. Device 0625 (rev 01)
    02:00.0 SATA controller: ASMedia Technology Inc. Device 0625 (rev 01)
    03:00.0 USB controller: Advanced Micro Devices, Inc. [AMD] 300 Series Chipset USB 3.1 xHCI Controller (rev 02)
    03:00.1 SATA controller: Advanced Micro Devices, Inc. [AMD] 300 Series Chipset SATA Controller (rev 02)
    03:00.2 PCI bridge: Advanced Micro Devices, Inc. [AMD] Device 43b2 (rev 02)
    04:00.0 PCI bridge: Advanced Micro Devices, Inc. [AMD] 300 Series Chipset PCIe Port (rev 02)
    04:01.0 PCI bridge: Advanced Micro Devices, Inc. [AMD] 300 Series Chipset PCIe Port (rev 02)
    04:04.0 PCI bridge: Advanced Micro Devices, Inc. [AMD] 300 Series Chipset PCIe Port (rev 02)
    05:00.0 PCI bridge: ASMedia Technology Inc. Device 1182
    06:03.0 PCI bridge: ASMedia Technology Inc. Device 1182
    06:07.0 PCI bridge: ASMedia Technology Inc. Device 1182
    07:00.0 Ethernet controller: Intel Corporation I211 Gigabit Network Connection (rev 03)
    08:00.0 Ethernet controller: Intel Corporation I211 Gigabit Network Connection (rev 03)
    09:00.0 PCI bridge: ASMedia Technology Inc. Device 1182
    0a:03.0 PCI bridge: ASMedia Technology Inc. Device 1182
    0a:07.0 PCI bridge: ASMedia Technology Inc. Device 1182
    0b:00.0 Ethernet controller: Intel Corporation I211 Gigabit Network Connection (rev 03)
    0c:00.0 Ethernet controller: Intel Corporation I211 Gigabit Network Connection (rev 03)
    0e:00.0 SATA controller: ASMedia Technology Inc. Device 0625 (rev 01)
    11:00.0 SATA controller: ASMedia Technology Inc. Device 0625 (rev 01)
    12:00.0 SATA controller: ASMedia Technology Inc. Device 0625 (rev 01)
    13:00.0 Non-Essential Instrumentation [1300]: Advanced Micro Devices, Inc. [AMD] Zeppelin/Raven/Raven2 PCIe Dummy Function
    13:00.2 Encryption controller: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-0fh) Platform Security Processor
    13:00.3 USB controller: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-0fh) USB 3.0 Host Controller
    14:00.0 Non-Essential Instrumentation [1300]: Advanced Micro Devices, Inc. [AMD] Zeppelin/Renoir PCIe Dummy Function
    14:00.2 SATA controller: Advanced Micro Devices, Inc. [AMD] FCH SATA Controller [AHCI mode] (rev 51)
    14:00.3 Audio device: Advanced Micro Devices, Inc. [AMD] Family 17h (Models 00h-0fh) HD Audio Controller

If you plug in a video card, it will show up in slot 0f:00:

    0f:00.0 VGA compatible controller: NVIDIA Corporation GK208B [GeForce GT 710] (rev a1)
    0f:00.1 Audio device: NVIDIA Corporation GK208 HDMI/DP Audio Controller (rev a1)

## lsusb

    Bus 004 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
    Bus 003 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
    Bus 002 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
    Bus 001 Device 003: ID 1005:b155 Apacer Technology, Inc. USB DISK MODULE
    Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub

## dmesg

    [    0.000000] kernel: Linux version 5.4.0-39-generic (buildd@lcy01-amd64-016) (gcc version 9.3.0 (Ubuntu 9.3.0-10ubuntu2)) #43-Ubuntu SMP Fri Jun 19 10:28:31 UTC 2020 (Ubuntu 5.4.0-39.43-generic 5.4.41)
    [    0.000000] kernel: Command line: BOOT_IMAGE=/vmlinuz-5.4.0-39-generic root=UUID=3c666d8f-8db7-4d3a-8fa5-1ffa022f85be ro quiet splash vt.handoff=7
    [    0.000000] kernel: KERNEL supported cpus:
    [    0.000000] kernel:   Intel GenuineIntel
    [    0.000000] kernel:   AMD AuthenticAMD
    [    0.000000] kernel:   Hygon HygonGenuine
    [    0.000000] kernel:   Centaur CentaurHauls
    [    0.000000] kernel:   zhaoxin   Shanghai  
    [    0.000000] kernel: x86/fpu: Supporting XSAVE feature 0x001: 'x87 floating point registers'
    [    0.000000] kernel: x86/fpu: Supporting XSAVE feature 0x002: 'SSE registers'
    [    0.000000] kernel: x86/fpu: Supporting XSAVE feature 0x004: 'AVX registers'
    [    0.000000] kernel: x86/fpu: xstate_offset[2]:  576, xstate_sizes[2]:  256
    [    0.000000] kernel: x86/fpu: Enabled xstate features 0x7, context size is 832 bytes, using 'compacted' format.
    [    0.000000] kernel: BIOS-provided physical RAM map:
    [    0.000000] kernel: BIOS-e820: [mem 0x0000000000000000-0x000000000009afff] usable
    [    0.000000] kernel: BIOS-e820: [mem 0x000000000009b000-0x000000000009ffff] reserved
    [    0.000000] kernel: BIOS-e820: [mem 0x00000000000e0000-0x00000000000fffff] reserved
    [    0.000000] kernel: BIOS-e820: [mem 0x0000000000100000-0x0000000009cfffff] usable
    [    0.000000] kernel: BIOS-e820: [mem 0x0000000009d00000-0x0000000009ffffff] reserved
    [    0.000000] kernel: BIOS-e820: [mem 0x000000000a000000-0x000000000a1fffff] usable
    [    0.000000] kernel: BIOS-e820: [mem 0x000000000a200000-0x000000000a209fff] ACPI NVS
    [    0.000000] kernel: BIOS-e820: [mem 0x000000000a20a000-0x000000000affffff] usable
    [    0.000000] kernel: BIOS-e820: [mem 0x000000000b000000-0x000000000b01ffff] reserved
    [    0.000000] kernel: BIOS-e820: [mem 0x000000000b020000-0x00000000dc22efff] usable
    [    0.000000] kernel: BIOS-e820: [mem 0x00000000dc22f000-0x00000000dc396fff] reserved
    [    0.000000] kernel: BIOS-e820: [mem 0x00000000dc397000-0x00000000dc518fff] usable
    [    0.000000] kernel: BIOS-e820: [mem 0x00000000dc519000-0x00000000dc92cfff] ACPI NVS
    [    0.000000] kernel: BIOS-e820: [mem 0x00000000dc92d000-0x00000000dd3a8fff] reserved
    [    0.000000] kernel: BIOS-e820: [mem 0x00000000dd3a9000-0x00000000deffffff] usable
    [    0.000000] kernel: BIOS-e820: [mem 0x00000000df000000-0x00000000dfffffff] reserved
    [    0.000000] kernel: BIOS-e820: [mem 0x00000000f8000000-0x00000000fbffffff] reserved
    [    0.000000] kernel: BIOS-e820: [mem 0x00000000fd100000-0x00000000fdffffff] reserved
    [    0.000000] kernel: BIOS-e820: [mem 0x00000000fea00000-0x00000000fea0ffff] reserved
    [    0.000000] kernel: BIOS-e820: [mem 0x00000000feb00000-0x00000000feb00fff] reserved
    [    0.000000] kernel: BIOS-e820: [mem 0x00000000feb80000-0x00000000fec01fff] reserved
    [    0.000000] kernel: BIOS-e820: [mem 0x00000000fec10000-0x00000000fec10fff] reserved
    [    0.000000] kernel: BIOS-e820: [mem 0x00000000fec30000-0x00000000fec30fff] reserved
    [    0.000000] kernel: BIOS-e820: [mem 0x00000000fed00000-0x00000000fed00fff] reserved
    [    0.000000] kernel: BIOS-e820: [mem 0x00000000fed40000-0x00000000fed44fff] reserved
    [    0.000000] kernel: BIOS-e820: [mem 0x00000000fed80000-0x00000000fed8ffff] reserved
    [    0.000000] kernel: BIOS-e820: [mem 0x00000000fedc2000-0x00000000fedcffff] reserved
    [    0.000000] kernel: BIOS-e820: [mem 0x00000000fedd4000-0x00000000fedd5fff] reserved
    [    0.000000] kernel: BIOS-e820: [mem 0x00000000fee00000-0x00000000feefffff] reserved
    [    0.000000] kernel: BIOS-e820: [mem 0x00000000ff000000-0x00000000ffffffff] reserved
    [    0.000000] kernel: BIOS-e820: [mem 0x0000000100000000-0x000000101f37ffff] usable
    [    0.000000] kernel: NX (Execute Disable) protection: active
    [    0.000000] kernel: SMBIOS 3.1.1 present.
    [    0.000000] kernel: DMI: Default string Default string/Default string, BIOS QZ14AR54 08/28/2018
    [    0.000000] kernel: tsc: Fast TSC calibration failed
    [    0.000000] kernel: e820: update [mem 0x00000000-0x00000fff] usable ==> reserved
    [    0.000000] kernel: e820: remove [mem 0x000a0000-0x000fffff] usable
    [    0.000000] kernel: last_pfn = 0x101f380 max_arch_pfn = 0x400000000
    [    0.000000] kernel: MTRR default type: uncachable
    [    0.000000] kernel: MTRR fixed ranges enabled:
    [    0.000000] kernel:   00000-9FFFF write-back
    [    0.000000] kernel:   A0000-BFFFF write-through
    [    0.000000] kernel:   C0000-FFFFF write-protect
    [    0.000000] kernel: MTRR variable ranges enabled:
    [    0.000000] kernel:   0 base 000000000000 mask FFFF80000000 write-back
    [    0.000000] kernel:   1 base 000080000000 mask FFFFC0000000 write-back
    [    0.000000] kernel:   2 base 0000C0000000 mask FFFFE0000000 write-back
    [    0.000000] kernel:   3 disabled
    [    0.000000] kernel:   4 disabled
    [    0.000000] kernel:   5 disabled
    [    0.000000] kernel:   6 disabled
    [    0.000000] kernel:   7 disabled
    [    0.000000] kernel: TOM2: 0000001020000000 aka 66048M
    [    0.000000] kernel: x86/PAT: Configuration [0-7]: WB  WC  UC- UC  WB  WP  UC- WT  
    [    0.000000] kernel: e820: update [mem 0xe0000000-0xffffffff] usable ==> reserved
    [    0.000000] kernel: last_pfn = 0xdf000 max_arch_pfn = 0x400000000
    [    0.000000] kernel: check: Scanning 1 areas for low memory corruption
    [    0.000000] kernel: Using GB pages for direct mapping
    [    0.000000] kernel: BRK [0x9fea01000, 0x9fea01fff] PGTABLE
    [    0.000000] kernel: BRK [0x9fea02000, 0x9fea02fff] PGTABLE
    [    0.000000] kernel: BRK [0x9fea03000, 0x9fea03fff] PGTABLE
    [    0.000000] kernel: BRK [0x9fea04000, 0x9fea04fff] PGTABLE
    [    0.000000] kernel: BRK [0x9fea05000, 0x9fea05fff] PGTABLE
    [    0.000000] kernel: BRK [0x9fea06000, 0x9fea06fff] PGTABLE
    [    0.000000] kernel: BRK [0x9fea07000, 0x9fea07fff] PGTABLE
    [    0.000000] kernel: BRK [0x9fea08000, 0x9fea08fff] PGTABLE
    [    0.000000] kernel: BRK [0x9fea09000, 0x9fea09fff] PGTABLE
    [    0.000000] kernel: BRK [0x9fea0a000, 0x9fea0afff] PGTABLE
    [    0.000000] kernel: BRK [0x9fea0b000, 0x9fea0bfff] PGTABLE
    [    0.000000] kernel: BRK [0x9fea0c000, 0x9fea0cfff] PGTABLE
    [    0.000000] kernel: RAMDISK: [mem 0x3198d000-0x34cbdfff]
    [    0.000000] kernel: ACPI: Early table checksum verification disabled
    [    0.000000] kernel: ACPI: RSDP 0x00000000000F05B0 000024 (v02 ALASKA)
    [    0.000000] kernel: ACPI: XSDT 0x00000000DC8AC0A0 0000BC (v01 ALASKA A M I    01072009 AMI  00010013)
    [    0.000000] kernel: ACPI: FACP 0x00000000DC8B30F8 000114 (v06 ALASKA A M I    01072009 AMI  00010013)
    [    0.000000] kernel: ACPI BIOS Warning (bug): Optional FADT field Pm2ControlBlock has valid Length but zero Address: 0x0000000000000000/0x1 (20190816/tbfadt-615)
    [    0.000000] kernel: ACPI: DSDT 0x00000000DC8AC1F0 006F01 (v02 ALASKA A M I    01072009 INTL 20120913)
    [    0.000000] kernel: ACPI: FACS 0x00000000DC915D80 000040
    [    0.000000] kernel: ACPI: APIC 0x00000000DC8B3210 0000DE (v03 ALASKA A M I    01072009 AMI  00010013)
    [    0.000000] kernel: ACPI: FPDT 0x00000000DC8B32F0 000044 (v01 ALASKA A M I    01072009 AMI  00010013)
    [    0.000000] kernel: ACPI: FIDT 0x00000000DC8B3338 00009C (v01 ALASKA A M I    01072009 AMI  00010013)
    [    0.000000] kernel: ACPI: SSDT 0x00000000DC8B33D8 008C98 (v02 AMD    AMD ALIB 00000002 MSFT 04000000)
    [    0.000000] kernel: ACPI: SSDT 0x00000000DC8BC070 001BF4 (v01 AMD    AMD CPU  00000001 AMD  00000001)
    [    0.000000] kernel: ACPI: CRAT 0x00000000DC8BDC68 000F50 (v01 AMD    AMD CRAT 00000001 AMD  00000001)
    [    0.000000] kernel: ACPI: CDIT 0x00000000DC8BEBB8 000029 (v01 AMD    AMD CDIT 00000001 AMD  00000001)
    [    0.000000] kernel: ACPI: SSDT 0x00000000DC8BEBE8 002C36 (v01 AMD    AMD AOD  00000001 INTL 20120913)
    [    0.000000] kernel: ACPI: MCFG 0x00000000DC8C1820 00003C (v01 ALASKA A M I    01072009 MSFT 00010013)
    [    0.000000] kernel: ACPI: HPET 0x00000000DC8C1860 000038 (v01 ALASKA A M I    01072009 AMI  00000005)
    [    0.000000] kernel: ACPI: WDRT 0x00000000DC8C1898 000047 (v01 ALASKA A M I    01072009 AMI  00000005)
    [    0.000000] kernel: ACPI: SSDT 0x00000000DC8C18E0 000024 (v01 AMDFCH FCHZP    00001000 INTL 20120913)
    [    0.000000] kernel: ACPI: UEFI 0x00000000DC8C1908 000048 (v01                 00000000      00000000)
    [    0.000000] kernel: ACPI: SPCR 0x00000000DC8C1950 000050 (v01 A M I  APTIO V  01072009 AMI. 0005000D)
    [    0.000000] kernel: ACPI: IVRS 0x00000000DC8C19A0 0000D0 (v02 AMD    AMD IVRS 00000001 AMD  00000000)
    [    0.000000] kernel: ACPI: SSDT 0x00000000DC8C1A70 001AE8 (v01 AMD    AmdTable 00000001 INTL 20120913)
    [    0.000000] kernel: ACPI: SSDT 0x00000000DC8C3558 0000BF (v01 AMD    AMD PT   00001000 INTL 20120913)
    [    0.000000] kernel: ACPI: WSMT 0x00000000DC8C3618 000028 (v01 ALASKA A M I    01072009 AMI  00010013)
    [    0.000000] kernel: ACPI: Local APIC address 0xfee00000
    [    0.000000] kernel: system APIC only can use physical flat
    [    0.000000] kernel: Setting APIC routing to physical flat.
    [    0.000000] kernel: No NUMA configuration found
    [    0.000000] kernel: Faking a node at [mem 0x0000000000000000-0x000000101f37ffff]
    [    0.000000] kernel: NODE_DATA(0) allocated [mem 0x101f355000-0x101f37ffff]
    [    0.000000] kernel: Zone ranges:
    [    0.000000] kernel:   DMA      [mem 0x0000000000001000-0x0000000000ffffff]
    [    0.000000] kernel:   DMA32    [mem 0x0000000001000000-0x00000000ffffffff]
    [    0.000000] kernel:   Normal   [mem 0x0000000100000000-0x000000101f37ffff]
    [    0.000000] kernel:   Device   empty
    [    0.000000] kernel: Movable zone start for each node
    [    0.000000] kernel: Early memory node ranges
    [    0.000000] kernel:   node   0: [mem 0x0000000000001000-0x000000000009afff]
    [    0.000000] kernel:   node   0: [mem 0x0000000000100000-0x0000000009cfffff]
    [    0.000000] kernel:   node   0: [mem 0x000000000a000000-0x000000000a1fffff]
    [    0.000000] kernel:   node   0: [mem 0x000000000a20a000-0x000000000affffff]
    [    0.000000] kernel:   node   0: [mem 0x000000000b020000-0x00000000dc22efff]
    [    0.000000] kernel:   node   0: [mem 0x00000000dc397000-0x00000000dc518fff]
    [    0.000000] kernel:   node   0: [mem 0x00000000dd3a9000-0x00000000deffffff]
    [    0.000000] kernel:   node   0: [mem 0x0000000100000000-0x000000101f37ffff]
    [    0.000000] kernel: Zeroed struct page in unavailable ranges: 12296 pages
    [    0.000000] kernel: Initmem setup node 0 [mem 0x0000000000001000-0x000000101f37ffff]
    [    0.000000] kernel: On node 0 totalpages: 16764920
    [    0.000000] kernel:   DMA zone: 64 pages used for memmap
    [    0.000000] kernel:   DMA zone: 21 pages reserved
    [    0.000000] kernel:   DMA zone: 3994 pages, LIFO batch:0
    [    0.000000] kernel:   DMA32 zone: 14132 pages used for memmap
    [    0.000000] kernel:   DMA32 zone: 904414 pages, LIFO batch:63
    [    0.000000] kernel:   Normal zone: 247758 pages used for memmap
    [    0.000000] kernel:   Normal zone: 15856512 pages, LIFO batch:63
    [    0.000000] kernel: ACPI: PM-Timer IO Port: 0x808
    [    0.000000] kernel: ACPI: Local APIC address 0xfee00000
    [    0.000000] kernel: system APIC only can use physical flat
    [    0.000000] kernel: ACPI: LAPIC_NMI (acpi_id[0xff] high edge lint[0x1])
    [    0.000000] kernel: IOAPIC[0]: apic_id 17, version 33, address 0xfec00000, GSI 0-23
    [    0.000000] kernel: IOAPIC[1]: apic_id 18, version 33, address 0xfec01000, GSI 24-55
    [    0.000000] kernel: ACPI: INT_SRC_OVR (bus 0 bus_irq 0 global_irq 2 dfl dfl)
    [    0.000000] kernel: ACPI: INT_SRC_OVR (bus 0 bus_irq 9 global_irq 9 low level)
    [    0.000000] kernel: ACPI: IRQ0 used by override.
    [    0.000000] kernel: ACPI: IRQ9 used by override.
    [    0.000000] kernel: Using ACPI (MADT) for SMP configuration information
    [    0.000000] kernel: ACPI: HPET id: 0x10228201 base: 0xfed00000
    [    0.000000] kernel: ACPI: SPCR: SPCR table version 1
    [    0.000000] kernel: ACPI: SPCR: console: uart,io,0x3f8,115200
    [    0.000000] kernel: smpboot: Allowing 16 CPUs, 0 hotplug CPUs
    [    0.000000] kernel: PM: Registered nosave memory: [mem 0x00000000-0x00000fff]
    [    0.000000] kernel: PM: Registered nosave memory: [mem 0x0009b000-0x0009ffff]
    [    0.000000] kernel: PM: Registered nosave memory: [mem 0x000a0000-0x000dffff]
    [    0.000000] kernel: PM: Registered nosave memory: [mem 0x000e0000-0x000fffff]
    [    0.000000] kernel: PM: Registered nosave memory: [mem 0x09d00000-0x09ffffff]
    [    0.000000] kernel: PM: Registered nosave memory: [mem 0x0a200000-0x0a209fff]
    [    0.000000] kernel: PM: Registered nosave memory: [mem 0x0b000000-0x0b01ffff]
    [    0.000000] kernel: PM: Registered nosave memory: [mem 0xdc22f000-0xdc396fff]
    [    0.000000] kernel: PM: Registered nosave memory: [mem 0xdc519000-0xdc92cfff]
    [    0.000000] kernel: PM: Registered nosave memory: [mem 0xdc92d000-0xdd3a8fff]
    [    0.000000] kernel: PM: Registered nosave memory: [mem 0xdf000000-0xdfffffff]
    [    0.000000] kernel: PM: Registered nosave memory: [mem 0xe0000000-0xf7ffffff]
    [    0.000000] kernel: PM: Registered nosave memory: [mem 0xf8000000-0xfbffffff]
    [    0.000000] kernel: PM: Registered nosave memory: [mem 0xfc000000-0xfd0fffff]
    [    0.000000] kernel: PM: Registered nosave memory: [mem 0xfd100000-0xfdffffff]
    [    0.000000] kernel: PM: Registered nosave memory: [mem 0xfe000000-0xfe9fffff]
    [    0.000000] kernel: PM: Registered nosave memory: [mem 0xfea00000-0xfea0ffff]
    [    0.000000] kernel: PM: Registered nosave memory: [mem 0xfea10000-0xfeafffff]
    [    0.000000] kernel: PM: Registered nosave memory: [mem 0xfeb00000-0xfeb00fff]
    [    0.000000] kernel: PM: Registered nosave memory: [mem 0xfeb01000-0xfeb7ffff]
    [    0.000000] kernel: PM: Registered nosave memory: [mem 0xfeb80000-0xfec01fff]
    [    0.000000] kernel: PM: Registered nosave memory: [mem 0xfec02000-0xfec0ffff]
    [    0.000000] kernel: PM: Registered nosave memory: [mem 0xfec10000-0xfec10fff]
    [    0.000000] kernel: PM: Registered nosave memory: [mem 0xfec11000-0xfec2ffff]
    [    0.000000] kernel: PM: Registered nosave memory: [mem 0xfec30000-0xfec30fff]
    [    0.000000] kernel: PM: Registered nosave memory: [mem 0xfec31000-0xfecfffff]
    [    0.000000] kernel: PM: Registered nosave memory: [mem 0xfed00000-0xfed00fff]
    [    0.000000] kernel: PM: Registered nosave memory: [mem 0xfed01000-0xfed3ffff]
    [    0.000000] kernel: PM: Registered nosave memory: [mem 0xfed40000-0xfed44fff]
    [    0.000000] kernel: PM: Registered nosave memory: [mem 0xfed45000-0xfed7ffff]
    [    0.000000] kernel: PM: Registered nosave memory: [mem 0xfed80000-0xfed8ffff]
    [    0.000000] kernel: PM: Registered nosave memory: [mem 0xfed90000-0xfedc1fff]
    [    0.000000] kernel: PM: Registered nosave memory: [mem 0xfedc2000-0xfedcffff]
    [    0.000000] kernel: PM: Registered nosave memory: [mem 0xfedd0000-0xfedd3fff]
    [    0.000000] kernel: PM: Registered nosave memory: [mem 0xfedd4000-0xfedd5fff]
    [    0.000000] kernel: PM: Registered nosave memory: [mem 0xfedd6000-0xfedfffff]
    [    0.000000] kernel: PM: Registered nosave memory: [mem 0xfee00000-0xfeefffff]
    [    0.000000] kernel: PM: Registered nosave memory: [mem 0xfef00000-0xfeffffff]
    [    0.000000] kernel: PM: Registered nosave memory: [mem 0xff000000-0xffffffff]
    [    0.000000] kernel: [mem 0xe0000000-0xf7ffffff] available for PCI devices
    [    0.000000] kernel: Booting paravirtualized kernel on bare hardware
    [    0.000000] kernel: clocksource: refined-jiffies: mask: 0xffffffff max_cycles: 0xffffffff, max_idle_ns: 7645519600211568 ns
    [    0.000000] kernel: setup_percpu: NR_CPUS:8192 nr_cpumask_bits:16 nr_cpu_ids:16 nr_node_ids:1
    [    0.000000] kernel: percpu: Embedded 54 pages/cpu s184320 r8192 d28672 u262144
    [    0.000000] kernel: pcpu-alloc: s184320 r8192 d28672 u262144 alloc=1*2097152
    [    0.000000] kernel: pcpu-alloc: [0] 00 01 02 03 04 05 06 07 [0] 08 09 10 11 12 13 14 15 
    [    0.000000] kernel: Built 1 zonelists, mobility grouping on.  Total pages: 16502945
    [    0.000000] kernel: Policy zone: Normal
    [    0.000000] kernel: Kernel command line: BOOT_IMAGE=/vmlinuz-5.4.0-39-generic root=UUID=3c666d8f-8db7-4d3a-8fa5-1ffa022f85be ro quiet splash vt.handoff=7
    [    0.000000] kernel: Dentry cache hash table entries: 8388608 (order: 14, 67108864 bytes, linear)
    [    0.000000] kernel: Inode-cache hash table entries: 4194304 (order: 13, 33554432 bytes, linear)
    [    0.000000] kernel: mem auto-init: stack:off, heap alloc:on, heap free:off
    [    0.000000] kernel: Calgary: detecting Calgary via BIOS EBDA area
    [    0.000000] kernel: Calgary: Unable to locate Rio Grande table in EBDA - bailing!
    [    0.000000] kernel: Memory: 65757832K/67059680K available (14339K kernel code, 2397K rwdata, 4952K rodata, 2712K init, 4992K bss, 1301848K reserved, 0K cma-reserved)
    [    0.000000] kernel: random: get_random_u64 called from kmem_cache_open+0x2d/0x410 with crng_init=0
    [    0.000000] kernel: SLUB: HWalign=64, Order=0-3, MinObjects=0, CPUs=16, Nodes=1
    [    0.000000] kernel: ftrace: allocating 44484 entries in 174 pages
    [    0.000000] kernel: rcu: Hierarchical RCU implementation.
    [    0.000000] kernel: rcu:         RCU restricting CPUs from NR_CPUS=8192 to nr_cpu_ids=16.
    [    0.000000] kernel:         Tasks RCU enabled.
    [    0.000000] kernel: rcu: RCU calculated value of scheduler-enlistment delay is 25 jiffies.
    [    0.000000] kernel: rcu: Adjusting geometry for rcu_fanout_leaf=16, nr_cpu_ids=16
    [    0.000000] kernel: NR_IRQS: 524544, nr_irqs: 1096, preallocated irqs: 16
    [    0.000000] kernel: random: crng done (trusting CPU's manufacturer)
    [    0.000000] kernel: spurious 8259A interrupt: IRQ7.
    [    0.000000] kernel: vt handoff: transparent VT on vt#7
    [    0.000000] kernel: Console: colour dummy device 80x25
    [    0.000000] kernel: printk: console [tty0] enabled
    [    0.000000] kernel: ACPI: Core revision 20190816
    [    0.000000] kernel: clocksource: hpet: mask: 0xffffffff max_cycles: 0xffffffff, max_idle_ns: 133484873504 ns
    [    0.000000] kernel: APIC: Switch to symmetric I/O mode setup
    [    0.004000] kernel: ..TIMER: vector=0x30 apic1=0 pin1=2 apic2=-1 pin2=-1
    [    0.028000] kernel: tsc: PIT calibration matches HPET. 1 loops
    [    0.028000] kernel: tsc: Detected 2994.069 MHz processor
    [    0.000003] kernel: clocksource: tsc-early: mask: 0xffffffffffffffff max_cycles: 0x2b2862a9e86, max_idle_ns: 440795222829 ns
    [    0.000005] kernel: Calibrating delay loop (skipped), value calculated using timer frequency.. 5988.13 BogoMIPS (lpj=11976276)
    [    0.000007] kernel: pid_max: default: 32768 minimum: 301
    [    0.000037] kernel: LSM: Security Framework initializing
    [    0.000045] kernel: Yama: becoming mindful.
    [    0.000094] kernel: AppArmor: AppArmor initialized
    [    0.000216] kernel: Mount-cache hash table entries: 131072 (order: 8, 1048576 bytes, linear)
    [    0.000325] kernel: Mountpoint-cache hash table entries: 131072 (order: 8, 1048576 bytes, linear)
    [    0.000340] kernel: *** VALIDATE tmpfs ***
    [    0.000476] kernel: *** VALIDATE proc ***
    [    0.000526] kernel: *** VALIDATE cgroup1 ***
    [    0.000527] kernel: *** VALIDATE cgroup2 ***
    [    0.000582] kernel: LVT offset 1 assigned for vector 0xf9
    [    0.000653] kernel: LVT offset 2 assigned for vector 0xf4
    [    0.000671] kernel: Last level iTLB entries: 4KB 1024, 2MB 1024, 4MB 512
    [    0.000672] kernel: Last level dTLB entries: 4KB 1536, 2MB 1536, 4MB 768, 1GB 0
    [    0.000674] kernel: Spectre V1 : Mitigation: usercopy/swapgs barriers and __user pointer sanitization
    [    0.000675] kernel: Spectre V2 : Mitigation: Full AMD retpoline
    [    0.000675] kernel: Spectre V2 : Spectre v2 / SpectreRSB mitigation: Filling RSB on context switch
    [    0.000676] kernel: Spectre V2 : mitigation: Enabling conditional Indirect Branch Prediction Barrier
    [    0.000677] kernel: Spectre V2 : User space: Vulnerable
    [    0.000678] kernel: Speculative Store Bypass: Mitigation: Speculative Store Bypass disabled via prctl and seccomp
    [    0.000891] kernel: Freeing SMP alternatives memory: 40K
    [    0.117682] kernel: smpboot: CPU0: AMD Ryzen 7 PRO 1700 Eight-Core Processor (family: 0x17, model: 0x1, stepping: 0x1)
    [    0.117791] kernel: Performance Events: Fam17h+ core perfctr, AMD PMU driver.
    [    0.117795] kernel: ... version:                0
    [    0.117796] kernel: ... bit width:              48
    [    0.117796] kernel: ... generic registers:      6
    [    0.117796] kernel: ... value mask:             0000ffffffffffff
    [    0.117797] kernel: ... max period:             00007fffffffffff
    [    0.117797] kernel: ... fixed-purpose events:   0
    [    0.117798] kernel: ... event mask:             000000000000003f
    [    0.117825] kernel: rcu: Hierarchical SRCU implementation.
    [    0.118320] kernel: NMI watchdog: Enabled. Permanently consumes one hw-PMU counter.
    [    0.118448] kernel: smp: Bringing up secondary CPUs ...
    [    0.118538] kernel: x86: Booting SMP configuration:
    [    0.118539] kernel: .... node  #0, CPUs:        #1  #2  #3  #4  #5  #6  #7  #8  #9 #10 #11 #12 #13 #14 #15
    [    0.150239] kernel: smp: Brought up 1 node, 16 CPUs
    [    0.150239] kernel: smpboot: Max logical packages: 1
    [    0.150239] kernel: smpboot: Total of 16 processors activated (95810.20 BogoMIPS)
    [    0.153925] kernel: devtmpfs: initialized
    [    0.153925] kernel: x86/mm: Memory block size: 128MB
    [    0.161734] kernel: PM: Registering ACPI NVS region [mem 0x0a200000-0x0a209fff] (40960 bytes)
    [    0.161734] kernel: PM: Registering ACPI NVS region [mem 0xdc519000-0xdc92cfff] (4276224 bytes)
    [    0.161734] kernel: clocksource: jiffies: mask: 0xffffffff max_cycles: 0xffffffff, max_idle_ns: 7645041785100000 ns
    [    0.161734] kernel: futex hash table entries: 4096 (order: 6, 262144 bytes, linear)
    [    0.161734] kernel: pinctrl core: initialized pinctrl subsystem
    [    0.161734] kernel: PM: RTC time: 14:27:24, date: 2020-06-26
    [    0.161734] kernel: NET: Registered protocol family 16
    [    0.161734] kernel: audit: initializing netlink subsys (disabled)
    [    0.161734] kernel: audit: type=2000 audit(1593181644.188:1): state=initialized audit_enabled=0 res=1
    [    0.161734] kernel: EISA bus registered
    [    0.161734] kernel: cpuidle: using governor ladder
    [    0.161734] kernel: cpuidle: using governor menu
    [    0.161734] kernel: ACPI: bus type PCI registered
    [    0.161734] kernel: acpiphp: ACPI Hot Plug PCI Controller Driver version: 0.5
    [    0.161734] kernel: PCI: MMCONFIG for domain 0000 [bus 00-3f] at [mem 0xf8000000-0xfbffffff] (base 0xf8000000)
    [    0.161734] kernel: PCI: MMCONFIG at [mem 0xf8000000-0xfbffffff] reserved in E820
    [    0.161734] kernel: PCI: Using configuration type 1 for base access
    [    0.163788] kernel: mtrr: your CPUs had inconsistent variable MTRR settings
    [    0.163789] kernel: mtrr: probably your BIOS does not setup all CPUs.
    [    0.163789] kernel: mtrr: corrected configuration.
    [    0.168050] kernel: HugeTLB registered 1.00 GiB page size, pre-allocated 0 pages
    [    0.168050] kernel: HugeTLB registered 2.00 MiB page size, pre-allocated 0 pages
    [    0.168110] kernel: ACPI: Added _OSI(Module Device)
    [    0.168111] kernel: ACPI: Added _OSI(Processor Device)
    [    0.168111] kernel: ACPI: Added _OSI(3.0 _SCP Extensions)
    [    0.168112] kernel: ACPI: Added _OSI(Processor Aggregator Device)
    [    0.168113] kernel: ACPI: Added _OSI(Linux-Dell-Video)
    [    0.168113] kernel: ACPI: Added _OSI(Linux-Lenovo-NV-HDMI-Audio)
    [    0.168114] kernel: ACPI: Added _OSI(Linux-HPI-Hybrid-Graphics)
    [    0.181586] kernel: ACPI: 7 ACPI AML tables successfully acquired and loaded
    [    0.183013] kernel: ACPI: [Firmware Bug]: BIOS _OSI(Linux) query ignored
    [    0.186748] kernel: ACPI: Interpreter enabled
    [    0.186760] kernel: ACPI: (supports S0 S5)
    [    0.186760] kernel: ACPI: Using IOAPIC for interrupt routing
    [    0.187103] kernel: PCI: Using host bridge windows from ACPI; if necessary, use "pci=nocrs" and report a bug
    [    0.187416] kernel: ACPI: Enabled 2 GPEs in block 00 to 1F
    [    0.196937] kernel: ACPI: PCI Root Bridge [PCI0] (domain 0000 [bus 00-ff])
    [    0.196942] kernel: acpi PNP0A08:00: _OSC: OS supports [ExtendedConfig ASPM ClockPM Segments MSI HPX-Type3]
    [    0.197141] kernel: acpi PNP0A08:00: _OSC: platform does not support [SHPCHotplug LTR]
    [    0.197331] kernel: acpi PNP0A08:00: _OSC: OS now controls [PCIeHotplug PME AER PCIeCapability]
    [    0.197342] kernel: acpi PNP0A08:00: [Firmware Info]: MMCONFIG for domain 0000 [bus 00-3f] only partially covers this bridge
    [    0.197689] kernel: PCI host bridge to bus 0000:00
    [    0.197690] kernel: pci_bus 0000:00: root bus resource [io  0x0000-0x03af window]
    [    0.197691] kernel: pci_bus 0000:00: root bus resource [io  0x03e0-0x0cf7 window]
    [    0.197692] kernel: pci_bus 0000:00: root bus resource [io  0x03b0-0x03df window]
    [    0.197693] kernel: pci_bus 0000:00: root bus resource [io  0x0d00-0xffff window]
    [    0.197694] kernel: pci_bus 0000:00: root bus resource [mem 0x000a0000-0x000bffff window]
    [    0.197695] kernel: pci_bus 0000:00: root bus resource [mem 0x000c0000-0x000dffff window]
    [    0.197696] kernel: pci_bus 0000:00: root bus resource [mem 0xe0000000-0xf7ffffff window]
    [    0.197697] kernel: pci_bus 0000:00: root bus resource [bus 00-ff]
    [    0.197705] kernel: pci 0000:00:00.0: [1022:1450] type 00 class 0x060000
    [    0.197800] kernel: pci 0000:00:00.2: [1022:1451] type 00 class 0x080600
    [    0.197910] kernel: pci 0000:00:01.0: [1022:1452] type 00 class 0x060000
    [    0.197985] kernel: pci 0000:00:01.1: [1022:1453] type 01 class 0x060400
    [    0.198312] kernel: pci 0000:00:01.1: enabling Extended Tags
    [    0.198422] kernel: pci 0000:00:01.1: PME# supported from D0 D3hot D3cold
    [    0.198538] kernel: pci 0000:00:01.2: [1022:1453] type 01 class 0x060400
    [    0.198570] kernel: pci 0000:00:01.2: enabling Extended Tags
    [    0.198675] kernel: pci 0000:00:01.2: PME# supported from D0 D3hot D3cold
    [    0.198785] kernel: pci 0000:00:01.3: [1022:1453] type 01 class 0x060400
    [    0.198817] kernel: pci 0000:00:01.3: enabling Extended Tags
    [    0.198922] kernel: pci 0000:00:01.3: PME# supported from D0 D3hot D3cold
    [    0.199407] kernel: pci 0000:00:01.4: [1022:1453] type 01 class 0x060400
    [    0.199438] kernel: pci 0000:00:01.4: enabling Extended Tags
    [    0.199546] kernel: pci 0000:00:01.4: PME# supported from D0 D3hot D3cold
    [    0.199663] kernel: pci 0000:00:02.0: [1022:1452] type 00 class 0x060000
    [    0.199749] kernel: pci 0000:00:03.0: [1022:1452] type 00 class 0x060000
    [    0.199821] kernel: pci 0000:00:03.1: [1022:1453] type 01 class 0x060400
    [    0.200045] kernel: pci 0000:00:03.1: PME# supported from D0 D3hot D3cold
    [    0.200161] kernel: pci 0000:00:03.2: [1022:1453] type 01 class 0x060400
    [    0.200195] kernel: pci 0000:00:03.2: enabling Extended Tags
    [    0.200306] kernel: pci 0000:00:03.2: PME# supported from D0 D3hot D3cold
    [    0.200419] kernel: pci 0000:00:03.3: [1022:1453] type 01 class 0x060400
    [    0.200453] kernel: pci 0000:00:03.3: enabling Extended Tags
    [    0.200562] kernel: pci 0000:00:03.3: PME# supported from D0 D3hot D3cold
    [    0.200674] kernel: pci 0000:00:03.4: [1022:1453] type 01 class 0x060400
    [    0.201010] kernel: pci 0000:00:03.4: enabling Extended Tags
    [    0.201122] kernel: pci 0000:00:03.4: PME# supported from D0 D3hot D3cold
    [    0.201237] kernel: pci 0000:00:04.0: [1022:1452] type 00 class 0x060000
    [    0.201329] kernel: pci 0000:00:07.0: [1022:1452] type 00 class 0x060000
    [    0.201402] kernel: pci 0000:00:07.1: [1022:1454] type 01 class 0x060400
    [    0.201432] kernel: pci 0000:00:07.1: enabling Extended Tags
    [    0.201505] kernel: pci 0000:00:07.1: PME# supported from D0 D3hot D3cold
    [    0.201629] kernel: pci 0000:00:08.0: [1022:1452] type 00 class 0x060000
    [    0.201703] kernel: pci 0000:00:08.1: [1022:1454] type 01 class 0x060400
    [    0.202018] kernel: pci 0000:00:08.1: enabling Extended Tags
    [    0.202095] kernel: pci 0000:00:08.1: PME# supported from D0 D3hot D3cold
    [    0.202258] kernel: pci 0000:00:14.0: [1022:790b] type 00 class 0x0c0500
    [    0.202487] kernel: pci 0000:00:14.3: [1022:790e] type 00 class 0x060100
    [    0.202718] kernel: pci 0000:00:18.0: [1022:1460] type 00 class 0x060000
    [    0.202780] kernel: pci 0000:00:18.1: [1022:1461] type 00 class 0x060000
    [    0.202840] kernel: pci 0000:00:18.2: [1022:1462] type 00 class 0x060000
    [    0.202904] kernel: pci 0000:00:18.3: [1022:1463] type 00 class 0x060000
    [    0.202964] kernel: pci 0000:00:18.4: [1022:1464] type 00 class 0x060000
    [    0.203025] kernel: pci 0000:00:18.5: [1022:1465] type 00 class 0x060000
    [    0.203087] kernel: pci 0000:00:18.6: [1022:1466] type 00 class 0x060000
    [    0.203147] kernel: pci 0000:00:18.7: [1022:1467] type 00 class 0x060000
    [    0.203288] kernel: pci 0000:01:00.0: [1b21:0625] type 00 class 0x010601
    [    0.203338] kernel: pci 0000:01:00.0: reg 0x24: [mem 0xf7f80000-0xf7f81fff]
    [    0.203345] kernel: pci 0000:01:00.0: reg 0x30: [mem 0xf7f00000-0xf7f7ffff pref]
    [    0.203459] kernel: pci 0000:00:01.1: PCI bridge to [bus 01]
    [    0.203463] kernel: pci 0000:00:01.1:   bridge window [mem 0xf7f00000-0xf7ffffff]
    [    0.203532] kernel: pci 0000:02:00.0: [1b21:0625] type 00 class 0x010601
    [    0.203587] kernel: pci 0000:02:00.0: reg 0x24: [mem 0xf7e80000-0xf7e81fff]
    [    0.203595] kernel: pci 0000:02:00.0: reg 0x30: [mem 0xf7e00000-0xf7e7ffff pref]
    [    0.204030] kernel: pci 0000:00:01.2: PCI bridge to [bus 02]
    [    0.204034] kernel: pci 0000:00:01.2:   bridge window [mem 0xf7e00000-0xf7efffff]
    [    0.204387] kernel: pci 0000:03:00.0: [1022:43bb] type 00 class 0x0c0330
    [    0.204408] kernel: pci 0000:03:00.0: reg 0x10: [mem 0xf76a0000-0xf76a7fff 64bit]
    [    0.204443] kernel: pci 0000:03:00.0: enabling Extended Tags
    [    0.204490] kernel: pci 0000:03:00.0: PME# supported from D3hot D3cold
    [    0.204573] kernel: pci 0000:03:00.1: [1022:43b7] type 00 class 0x010601
    [    0.204619] kernel: pci 0000:03:00.1: reg 0x24: [mem 0xf7680000-0xf769ffff]
    [    0.204626] kernel: pci 0000:03:00.1: reg 0x30: [mem 0xf7600000-0xf767ffff pref]
    [    0.204631] kernel: pci 0000:03:00.1: enabling Extended Tags
    [    0.204668] kernel: pci 0000:03:00.1: PME# supported from D3hot D3cold
    [    0.204731] kernel: pci 0000:03:00.2: [1022:43b2] type 01 class 0x060400
    [    0.204773] kernel: pci 0000:03:00.2: enabling Extended Tags
    [    0.204815] kernel: pci 0000:03:00.2: PME# supported from D3hot D3cold
    [    0.204908] kernel: pci 0000:00:01.3: PCI bridge to [bus 03-0d]
    [    0.204911] kernel: pci 0000:00:01.3:   bridge window [io  0xb000-0xefff]
    [    0.204913] kernel: pci 0000:00:01.3:   bridge window [mem 0xf7200000-0xf76fffff]
    [    0.205010] kernel: pci 0000:04:00.0: [1022:43b4] type 01 class 0x060400
    [    0.205057] kernel: pci 0000:04:00.0: enabling Extended Tags
    [    0.205109] kernel: pci 0000:04:00.0: PME# supported from D3hot D3cold
    [    0.205191] kernel: pci 0000:04:01.0: [1022:43b4] type 01 class 0x060400
    [    0.205238] kernel: pci 0000:04:01.0: enabling Extended Tags
    [    0.205290] kernel: pci 0000:04:01.0: PME# supported from D3hot D3cold
    [    0.205371] kernel: pci 0000:04:04.0: [1022:43b4] type 01 class 0x060400
    [    0.205418] kernel: pci 0000:04:04.0: enabling Extended Tags
    [    0.205470] kernel: pci 0000:04:04.0: PME# supported from D3hot D3cold
    [    0.205563] kernel: pci 0000:03:00.2: PCI bridge to [bus 04-0d]
    [    0.205567] kernel: pci 0000:03:00.2:   bridge window [io  0xb000-0xefff]
    [    0.205570] kernel: pci 0000:03:00.2:   bridge window [mem 0xf7200000-0xf75fffff]
    [    0.205619] kernel: pci 0000:05:00.0: [1b21:1182] type 01 class 0x060400
    [    0.205697] kernel: pci 0000:05:00.0: enabling Extended Tags
    [    0.205779] kernel: pci 0000:05:00.0: PME# supported from D0 D3hot D3cold
    [    0.205895] kernel: pci 0000:04:00.0: PCI bridge to [bus 05-08]
    [    0.205899] kernel: pci 0000:04:00.0:   bridge window [io  0xd000-0xefff]
    [    0.205902] kernel: pci 0000:04:00.0:   bridge window [mem 0xf7400000-0xf75fffff]
    [    0.205968] kernel: pci 0000:06:03.0: [1b21:1182] type 01 class 0x060400
    [    0.206046] kernel: pci 0000:06:03.0: enabling Extended Tags
    [    0.206127] kernel: pci 0000:06:03.0: PME# supported from D0 D3hot D3cold
    [    0.206212] kernel: pci 0000:06:07.0: [1b21:1182] type 01 class 0x060400
    [    0.206291] kernel: pci 0000:06:07.0: enabling Extended Tags
    [    0.206372] kernel: pci 0000:06:07.0: PME# supported from D0 D3hot D3cold
    [    0.206477] kernel: pci 0000:05:00.0: PCI bridge to [bus 06-08]
    [    0.206484] kernel: pci 0000:05:00.0:   bridge window [io  0xd000-0xefff]
    [    0.206488] kernel: pci 0000:05:00.0:   bridge window [mem 0xf7400000-0xf75fffff]
    [    0.206567] kernel: pci 0000:07:00.0: [8086:1539] type 00 class 0x020000
    [    0.206619] kernel: pci 0000:07:00.0: reg 0x10: [mem 0xf7500000-0xf751ffff]
    [    0.206657] kernel: pci 0000:07:00.0: reg 0x18: [io  0xe000-0xe01f]
    [    0.206677] kernel: pci 0000:07:00.0: reg 0x1c: [mem 0xf7520000-0xf7523fff]
    [    0.206883] kernel: pci 0000:07:00.0: PME# supported from D0 D3hot D3cold
    [    0.207044] kernel: pci 0000:06:03.0: PCI bridge to [bus 07]
    [    0.207051] kernel: pci 0000:06:03.0:   bridge window [io  0xe000-0xefff]
    [    0.207055] kernel: pci 0000:06:03.0:   bridge window [mem 0xf7500000-0xf75fffff]
    [    0.207136] kernel: pci 0000:08:00.0: [8086:1539] type 00 class 0x020000
    [    0.207187] kernel: pci 0000:08:00.0: reg 0x10: [mem 0xf7400000-0xf741ffff]
    [    0.207225] kernel: pci 0000:08:00.0: reg 0x18: [io  0xd000-0xd01f]
    [    0.207245] kernel: pci 0000:08:00.0: reg 0x1c: [mem 0xf7420000-0xf7423fff]
    [    0.207451] kernel: pci 0000:08:00.0: PME# supported from D0 D3hot D3cold
    [    0.207613] kernel: pci 0000:06:07.0: PCI bridge to [bus 08]
    [    0.207620] kernel: pci 0000:06:07.0:   bridge window [io  0xd000-0xdfff]
    [    0.207623] kernel: pci 0000:06:07.0:   bridge window [mem 0xf7400000-0xf74fffff]
    [    0.207708] kernel: pci 0000:09:00.0: [1b21:1182] type 01 class 0x060400
    [    0.207786] kernel: pci 0000:09:00.0: enabling Extended Tags
    [    0.207869] kernel: pci 0000:09:00.0: PME# supported from D0 D3hot D3cold
    [    0.207985] kernel: pci 0000:04:01.0: PCI bridge to [bus 09-0c]
    [    0.207990] kernel: pci 0000:04:01.0:   bridge window [io  0xb000-0xcfff]
    [    0.207992] kernel: pci 0000:04:01.0:   bridge window [mem 0xf7200000-0xf73fffff]
    [    0.208060] kernel: pci 0000:0a:03.0: [1b21:1182] type 01 class 0x060400
    [    0.208137] kernel: pci 0000:0a:03.0: enabling Extended Tags
    [    0.208217] kernel: pci 0000:0a:03.0: PME# supported from D0 D3hot D3cold
    [    0.208304] kernel: pci 0000:0a:07.0: [1b21:1182] type 01 class 0x060400
    [    0.208380] kernel: pci 0000:0a:07.0: enabling Extended Tags
    [    0.208461] kernel: pci 0000:0a:07.0: PME# supported from D0 D3hot D3cold
    [    0.208563] kernel: pci 0000:09:00.0: PCI bridge to [bus 0a-0c]
    [    0.208569] kernel: pci 0000:09:00.0:   bridge window [io  0xb000-0xcfff]
    [    0.208573] kernel: pci 0000:09:00.0:   bridge window [mem 0xf7200000-0xf73fffff]
    [    0.208652] kernel: pci 0000:0b:00.0: [8086:1539] type 00 class 0x020000
    [    0.208703] kernel: pci 0000:0b:00.0: reg 0x10: [mem 0xf7300000-0xf731ffff]
    [    0.208741] kernel: pci 0000:0b:00.0: reg 0x18: [io  0xc000-0xc01f]
    [    0.208761] kernel: pci 0000:0b:00.0: reg 0x1c: [mem 0xf7320000-0xf7323fff]
    [    0.208966] kernel: pci 0000:0b:00.0: PME# supported from D0 D3hot D3cold
    [    0.209126] kernel: pci 0000:0a:03.0: PCI bridge to [bus 0b]
    [    0.209133] kernel: pci 0000:0a:03.0:   bridge window [io  0xc000-0xcfff]
    [    0.209136] kernel: pci 0000:0a:03.0:   bridge window [mem 0xf7300000-0xf73fffff]
    [    0.209217] kernel: pci 0000:0c:00.0: [8086:1539] type 00 class 0x020000
    [    0.209269] kernel: pci 0000:0c:00.0: reg 0x10: [mem 0xf7200000-0xf721ffff]
    [    0.209307] kernel: pci 0000:0c:00.0: reg 0x18: [io  0xb000-0xb01f]
    [    0.209326] kernel: pci 0000:0c:00.0: reg 0x1c: [mem 0xf7220000-0xf7223fff]
    [    0.209533] kernel: pci 0000:0c:00.0: PME# supported from D0 D3hot D3cold
    [    0.209694] kernel: pci 0000:0a:07.0: PCI bridge to [bus 0c]
    [    0.209700] kernel: pci 0000:0a:07.0:   bridge window [io  0xb000-0xbfff]
    [    0.209704] kernel: pci 0000:0a:07.0:   bridge window [mem 0xf7200000-0xf72fffff]
    [    0.209773] kernel: pci 0000:04:04.0: PCI bridge to [bus 0d]
    [    0.209866] kernel: pci 0000:0e:00.0: [1b21:0625] type 00 class 0x010601
    [    0.209921] kernel: pci 0000:0e:00.0: reg 0x24: [mem 0xf7d80000-0xf7d81fff]
    [    0.209929] kernel: pci 0000:0e:00.0: reg 0x30: [mem 0xf7d00000-0xf7d7ffff pref]
    [    0.210055] kernel: pci 0000:00:01.4: PCI bridge to [bus 0e]
    [    0.210058] kernel: pci 0000:00:01.4:   bridge window [mem 0xf7d00000-0xf7dfffff]
    [    0.210384] kernel: pci 0000:0f:00.0: [10de:128b] type 00 class 0x030000
    [    0.210402] kernel: pci 0000:0f:00.0: reg 0x10: [mem 0xf6000000-0xf6ffffff]
    [    0.210411] kernel: pci 0000:0f:00.0: reg 0x14: [mem 0xe8000000-0xefffffff 64bit pref]
    [    0.210421] kernel: pci 0000:0f:00.0: reg 0x1c: [mem 0xf0000000-0xf1ffffff 64bit pref]
    [    0.210427] kernel: pci 0000:0f:00.0: reg 0x24: [io  0xf000-0xf07f]
    [    0.210433] kernel: pci 0000:0f:00.0: reg 0x30: [mem 0xf7000000-0xf707ffff pref]
    [    0.210530] kernel: pci 0000:0f:00.1: [10de:0e0f] type 00 class 0x040300
    [    0.210544] kernel: pci 0000:0f:00.1: reg 0x10: [mem 0xf7080000-0xf7083fff]
    [    0.210662] kernel: pci 0000:00:03.1: PCI bridge to [bus 0f]
    [    0.210665] kernel: pci 0000:00:03.1:   bridge window [io  0xf000-0xffff]
    [    0.210667] kernel: pci 0000:00:03.1:   bridge window [mem 0xf6000000-0xf70fffff]
    [    0.210670] kernel: pci 0000:00:03.1:   bridge window [mem 0xe8000000-0xf1ffffff 64bit pref]
    [    0.210725] kernel: pci 0000:00:03.2: PCI bridge to [bus 10]
    [    0.210794] kernel: pci 0000:11:00.0: [1b21:0625] type 00 class 0x010601
    [    0.210845] kernel: pci 0000:11:00.0: reg 0x24: [mem 0xf7c80000-0xf7c81fff]
    [    0.210853] kernel: pci 0000:11:00.0: reg 0x30: [mem 0xf7c00000-0xf7c7ffff pref]
    [    0.210970] kernel: pci 0000:00:03.3: PCI bridge to [bus 11]
    [    0.210974] kernel: pci 0000:00:03.3:   bridge window [mem 0xf7c00000-0xf7cfffff]
    [    0.211382] kernel: pci 0000:12:00.0: [1b21:0625] type 00 class 0x010601
    [    0.211434] kernel: pci 0000:12:00.0: reg 0x24: [mem 0xf7b80000-0xf7b81fff]
    [    0.211441] kernel: pci 0000:12:00.0: reg 0x30: [mem 0xf7b00000-0xf7b7ffff pref]
    [    0.211558] kernel: pci 0000:00:03.4: PCI bridge to [bus 12]
    [    0.211562] kernel: pci 0000:00:03.4:   bridge window [mem 0xf7b00000-0xf7bfffff]
    [    0.211630] kernel: pci 0000:13:00.0: [1022:145a] type 00 class 0x130000
    [    0.211662] kernel: pci 0000:13:00.0: enabling Extended Tags
    [    0.211722] kernel: pci 0000:13:00.2: [1022:1456] type 00 class 0x108000
    [    0.211739] kernel: pci 0000:13:00.2: reg 0x18: [mem 0xf7800000-0xf78fffff]
    [    0.211749] kernel: pci 0000:13:00.2: reg 0x24: [mem 0xf7900000-0xf7901fff]
    [    0.211756] kernel: pci 0000:13:00.2: enabling Extended Tags
    [    0.211828] kernel: pci 0000:13:00.3: [1022:145c] type 00 class 0x0c0330
    [    0.211840] kernel: pci 0000:13:00.3: reg 0x10: [mem 0xf7700000-0xf77fffff 64bit]
    [    0.211860] kernel: pci 0000:13:00.3: enabling Extended Tags
    [    0.211885] kernel: pci 0000:13:00.3: PME# supported from D0 D3hot D3cold
    [    0.211946] kernel: pci 0000:00:07.1: PCI bridge to [bus 13]
    [    0.211949] kernel: pci 0000:00:07.1:   bridge window [mem 0xf7700000-0xf79fffff]
    [    0.212063] kernel: pci 0000:14:00.0: [1022:1455] type 00 class 0x130000
    [    0.212098] kernel: pci 0000:14:00.0: enabling Extended Tags
    [    0.212167] kernel: pci 0000:14:00.2: [1022:7901] type 00 class 0x010601
    [    0.212195] kernel: pci 0000:14:00.2: reg 0x24: [mem 0xf7a08000-0xf7a08fff]
    [    0.212202] kernel: pci 0000:14:00.2: enabling Extended Tags
    [    0.212231] kernel: pci 0000:14:00.2: PME# supported from D3hot D3cold
    [    0.212281] kernel: pci 0000:14:00.3: [1022:1457] type 00 class 0x040300
    [    0.212291] kernel: pci 0000:14:00.3: reg 0x10: [mem 0xf7a00000-0xf7a07fff]
    [    0.212316] kernel: pci 0000:14:00.3: enabling Extended Tags
    [    0.212343] kernel: pci 0000:14:00.3: PME# supported from D0 D3hot D3cold
    [    0.212403] kernel: pci 0000:00:08.1: PCI bridge to [bus 14]
    [    0.212407] kernel: pci 0000:00:08.1:   bridge window [mem 0xf7a00000-0xf7afffff]
    [    0.212973] kernel: ACPI: PCI Interrupt Link [LNKA] (IRQs 4 5 7 10 11 14 15) *0
    [    0.213037] kernel: ACPI: PCI Interrupt Link [LNKB] (IRQs 4 5 7 10 11 14 15) *0
    [    0.213093] kernel: ACPI: PCI Interrupt Link [LNKC] (IRQs 4 5 7 10 11 14 15) *0
    [    0.213162] kernel: ACPI: PCI Interrupt Link [LNKD] (IRQs 4 5 7 10 11 14 15) *0
    [    0.213225] kernel: ACPI: PCI Interrupt Link [LNKE] (IRQs 4 5 7 10 11 14 15) *0
    [    0.213276] kernel: ACPI: PCI Interrupt Link [LNKF] (IRQs 4 5 7 10 11 14 15) *0
    [    0.213327] kernel: ACPI: PCI Interrupt Link [LNKG] (IRQs 4 5 7 10 11 14 15) *0
    [    0.213379] kernel: ACPI: PCI Interrupt Link [LNKH] (IRQs 4 5 7 10 11 14 15) *0
    [    0.216089] kernel: iommu: Default domain type: Translated 
    [    0.216165] kernel: SCSI subsystem initialized
    [    0.216193] kernel: libata version 3.00 loaded.
    [    0.216193] kernel: pci 0000:0f:00.0: vgaarb: setting as boot VGA device
    [    0.216193] kernel: pci 0000:0f:00.0: vgaarb: VGA device added: decodes=io+mem,owns=io+mem,locks=none
    [    0.216193] kernel: pci 0000:0f:00.0: vgaarb: bridge control possible
    [    0.216193] kernel: vgaarb: loaded
    [    0.216193] kernel: ACPI: bus type USB registered
    [    0.216193] kernel: usbcore: registered new interface driver usbfs
    [    0.216193] kernel: usbcore: registered new interface driver hub
    [    0.216193] kernel: usbcore: registered new device driver usb
    [    0.216193] kernel: pps_core: LinuxPPS API ver. 1 registered
    [    0.216193] kernel: pps_core: Software ver. 5.3.6 - Copyright 2005-2007 Rodolfo Giometti <giometti@linux.it>
    [    0.216193] kernel: PTP clock support registered
    [    0.216193] kernel: EDAC MC: Ver: 3.0.0
    [    0.216193] kernel: PCI: Using ACPI for IRQ routing
    [    0.218978] kernel: PCI: pci_cache_line_size set to 64 bytes
    [    0.219087] kernel: e820: reserve RAM buffer [mem 0x0009b000-0x0009ffff]
    [    0.219088] kernel: e820: reserve RAM buffer [mem 0x09d00000-0x0bffffff]
    [    0.219089] kernel: e820: reserve RAM buffer [mem 0x0a200000-0x0bffffff]
    [    0.219089] kernel: e820: reserve RAM buffer [mem 0x0b000000-0x0bffffff]
    [    0.219090] kernel: e820: reserve RAM buffer [mem 0xdc22f000-0xdfffffff]
    [    0.219090] kernel: e820: reserve RAM buffer [mem 0xdc519000-0xdfffffff]
    [    0.219091] kernel: e820: reserve RAM buffer [mem 0xdf000000-0xdfffffff]
    [    0.219092] kernel: e820: reserve RAM buffer [mem 0x101f380000-0x101fffffff]
    [    0.219174] kernel: NetLabel: Initializing
    [    0.219174] kernel: NetLabel:  domain hash size = 128
    [    0.219174] kernel: NetLabel:  protocols = UNLABELED CIPSOv4 CALIPSO
    [    0.219186] kernel: NetLabel:  unlabeled traffic allowed by default
    [    0.220050] kernel: hpet0: at MMIO 0xfed00000, IRQs 2, 8, 0
    [    0.220052] kernel: hpet0: 3 comparators, 32-bit 14.318180 MHz counter
    [    0.221056] kernel: clocksource: Switched to clocksource tsc-early
    [    0.230209] kernel: *** VALIDATE bpf ***
    [    0.230293] kernel: VFS: Disk quotas dquot_6.6.0
    [    0.230309] kernel: VFS: Dquot-cache hash table entries: 512 (order 0, 4096 bytes)
    [    0.230337] kernel: *** VALIDATE ramfs ***
    [    0.230339] kernel: *** VALIDATE hugetlbfs ***
    [    0.230415] kernel: AppArmor: AppArmor Filesystem Enabled
    [    0.230444] kernel: pnp: PnP ACPI init
    [    0.230609] kernel: system 00:00: [mem 0xf8000000-0xfbffffff] has been reserved
    [    0.230614] kernel: system 00:00: Plug and Play ACPI device, IDs PNP0c01 (active)
    [    0.230711] kernel: pnp 00:01: Plug and Play ACPI device, IDs PNP0b00 (active)
    [    0.231027] kernel: system 00:02: [io  0x0a00-0x0a0f] has been reserved
    [    0.231028] kernel: system 00:02: [io  0x0a10-0x0a2f] has been reserved
    [    0.231029] kernel: system 00:02: [io  0x0a30-0x0a4f] has been reserved
    [    0.231030] kernel: system 00:02: [io  0x0a50-0x0a6f] has been reserved
    [    0.231031] kernel: system 00:02: [io  0x0a70-0x0a7f] has been reserved
    [    0.231032] kernel: system 00:02: [io  0x0a80-0x0a8f] has been reserved
    [    0.231035] kernel: system 00:02: Plug and Play ACPI device, IDs PNP0c02 (active)
    [    0.231256] kernel: pnp 00:03: [dma 0 disabled]
    [    0.231292] kernel: pnp 00:03: Plug and Play ACPI device, IDs PNP0501 (active)
    [    0.231501] kernel: pnp 00:04: [dma 0 disabled]
    [    0.231531] kernel: pnp 00:04: Plug and Play ACPI device, IDs PNP0501 (active)
    [    0.231723] kernel: pnp 00:05: [dma 0 disabled]
    [    0.231743] kernel: pnp 00:05: Plug and Play ACPI device, IDs ITE8708 (active)
    [    0.232018] kernel: system 00:06: [io  0x04d0-0x04d1] has been reserved
    [    0.232019] kernel: system 00:06: [io  0x040b] has been reserved
    [    0.232020] kernel: system 00:06: [io  0x04d6] has been reserved
    [    0.232021] kernel: system 00:06: [io  0x0c00-0x0c01] has been reserved
    [    0.232022] kernel: system 00:06: [io  0x0c14] has been reserved
    [    0.232023] kernel: system 00:06: [io  0x0c50-0x0c51] has been reserved
    [    0.232024] kernel: system 00:06: [io  0x0c52] has been reserved
    [    0.232025] kernel: system 00:06: [io  0x0c6c] has been reserved
    [    0.232026] kernel: system 00:06: [io  0x0c6f] has been reserved
    [    0.232027] kernel: system 00:06: [io  0x0cd0-0x0cd1] has been reserved
    [    0.232027] kernel: system 00:06: [io  0x0cd2-0x0cd3] has been reserved
    [    0.232028] kernel: system 00:06: [io  0x0cd4-0x0cd5] has been reserved
    [    0.232029] kernel: system 00:06: [io  0x0cd6-0x0cd7] has been reserved
    [    0.232030] kernel: system 00:06: [io  0x0cd8-0x0cdf] has been reserved
    [    0.232031] kernel: system 00:06: [io  0x0800-0x089f] has been reserved
    [    0.232032] kernel: system 00:06: [io  0x0b00-0x0b0f] has been reserved
    [    0.232033] kernel: system 00:06: [io  0x0b20-0x0b3f] has been reserved
    [    0.232033] kernel: system 00:06: [io  0x0900-0x090f] has been reserved
    [    0.232034] kernel: system 00:06: [io  0x0910-0x091f] has been reserved
    [    0.232036] kernel: system 00:06: [mem 0xfec00000-0xfec00fff] could not be reserved
    [    0.232037] kernel: system 00:06: [mem 0xfec01000-0xfec01fff] could not be reserved
    [    0.232038] kernel: system 00:06: [mem 0xfedc0000-0xfedc0fff] has been reserved
    [    0.232039] kernel: system 00:06: [mem 0xfee00000-0xfee00fff] has been reserved
    [    0.232040] kernel: system 00:06: [mem 0xfed80000-0xfed8ffff] could not be reserved
    [    0.232041] kernel: system 00:06: [mem 0xfec10000-0xfec10fff] has been reserved
    [    0.232042] kernel: system 00:06: [mem 0xfeb00000-0xfeb00fff] has been reserved
    [    0.232043] kernel: system 00:06: [mem 0xff000000-0xffffffff] has been reserved
    [    0.232046] kernel: system 00:06: Plug and Play ACPI device, IDs PNP0c02 (active)
    [    0.232550] kernel: pnp: PnP ACPI: found 7 devices
    [    0.233601] kernel: thermal_sys: Registered thermal governor 'fair_share'
    [    0.233602] kernel: thermal_sys: Registered thermal governor 'bang_bang'
    [    0.233602] kernel: thermal_sys: Registered thermal governor 'step_wise'
    [    0.233603] kernel: thermal_sys: Registered thermal governor 'user_space'
    [    0.233603] kernel: thermal_sys: Registered thermal governor 'power_allocator'
    [    0.238120] kernel: clocksource: acpi_pm: mask: 0xffffff max_cycles: 0xffffff, max_idle_ns: 2085701024 ns
    [    0.238153] kernel: pci 0000:00:01.1: bridge window [io  0x1000-0x0fff] to [bus 01] add_size 1000
    [    0.238156] kernel: pci 0000:00:01.1: bridge window [mem 0x00100000-0x000fffff 64bit pref] to [bus 01] add_size 200000 add_align 100000
    [    0.238157] kernel: pci 0000:00:01.2: bridge window [io  0x1000-0x0fff] to [bus 02] add_size 1000
    [    0.238159] kernel: pci 0000:00:01.2: bridge window [mem 0x00100000-0x000fffff 64bit pref] to [bus 02] add_size 200000 add_align 100000
    [    0.238163] kernel: pci 0000:00:01.3: bridge window [mem 0x00100000-0x000fffff 64bit pref] to [bus 03-0d] add_size 200000 add_align 100000
    [    0.238164] kernel: pci 0000:00:01.4: bridge window [io  0x1000-0x0fff] to [bus 0e] add_size 1000
    [    0.238166] kernel: pci 0000:00:01.4: bridge window [mem 0x00100000-0x000fffff 64bit pref] to [bus 0e] add_size 200000 add_align 100000
    [    0.238167] kernel: pci 0000:00:03.2: bridge window [io  0x1000-0x0fff] to [bus 10] add_size 1000
    [    0.238168] kernel: pci 0000:00:03.2: bridge window [mem 0x00100000-0x000fffff 64bit pref] to [bus 10] add_size 200000 add_align 100000
    [    0.238170] kernel: pci 0000:00:03.2: bridge window [mem 0x00100000-0x000fffff] to [bus 10] add_size 200000 add_align 100000
    [    0.238171] kernel: pci 0000:00:03.3: bridge window [io  0x1000-0x0fff] to [bus 11] add_size 1000
    [    0.238172] kernel: pci 0000:00:03.3: bridge window [mem 0x00100000-0x000fffff 64bit pref] to [bus 11] add_size 200000 add_align 100000
    [    0.238173] kernel: pci 0000:00:03.4: bridge window [io  0x1000-0x0fff] to [bus 12] add_size 1000
    [    0.238174] kernel: pci 0000:00:03.4: bridge window [mem 0x00100000-0x000fffff 64bit pref] to [bus 12] add_size 200000 add_align 100000
    [    0.238185] kernel: pci 0000:00:01.1: BAR 15: assigned [mem 0xe0000000-0xe01fffff 64bit pref]
    [    0.238188] kernel: pci 0000:00:01.2: BAR 15: assigned [mem 0xe0200000-0xe03fffff 64bit pref]
    [    0.238191] kernel: pci 0000:00:01.3: BAR 15: assigned [mem 0xe0400000-0xe05fffff 64bit pref]
    [    0.238193] kernel: pci 0000:00:01.4: BAR 15: assigned [mem 0xe0600000-0xe07fffff 64bit pref]
    [    0.238195] kernel: pci 0000:00:03.2: BAR 14: assigned [mem 0xe0800000-0xe09fffff]
    [    0.238197] kernel: pci 0000:00:03.2: BAR 15: assigned [mem 0xe0a00000-0xe0bfffff 64bit pref]
    [    0.238199] kernel: pci 0000:00:03.3: BAR 15: assigned [mem 0xe0c00000-0xe0dfffff 64bit pref]
    [    0.238201] kernel: pci 0000:00:03.4: BAR 15: assigned [mem 0xe0e00000-0xe0ffffff 64bit pref]
    [    0.238204] kernel: pci 0000:00:01.1: BAR 13: assigned [io  0x1000-0x1fff]
    [    0.238206] kernel: pci 0000:00:01.2: BAR 13: assigned [io  0x2000-0x2fff]
    [    0.238208] kernel: pci 0000:00:01.4: BAR 13: assigned [io  0x3000-0x3fff]
    [    0.238209] kernel: pci 0000:00:03.2: BAR 13: assigned [io  0x4000-0x4fff]
    [    0.238211] kernel: pci 0000:00:03.3: BAR 13: assigned [io  0x5000-0x5fff]
    [    0.238213] kernel: pci 0000:00:03.4: BAR 13: assigned [io  0x6000-0x6fff]
    [    0.238216] kernel: pci 0000:00:01.1: PCI bridge to [bus 01]
    [    0.238217] kernel: pci 0000:00:01.1:   bridge window [io  0x1000-0x1fff]
    [    0.238220] kernel: pci 0000:00:01.1:   bridge window [mem 0xf7f00000-0xf7ffffff]
    [    0.238222] kernel: pci 0000:00:01.1:   bridge window [mem 0xe0000000-0xe01fffff 64bit pref]
    [    0.238225] kernel: pci 0000:00:01.2: PCI bridge to [bus 02]
    [    0.238226] kernel: pci 0000:00:01.2:   bridge window [io  0x2000-0x2fff]
    [    0.238228] kernel: pci 0000:00:01.2:   bridge window [mem 0xf7e00000-0xf7efffff]
    [    0.238230] kernel: pci 0000:00:01.2:   bridge window [mem 0xe0200000-0xe03fffff 64bit pref]
    [    0.238234] kernel: pci 0000:06:03.0: PCI bridge to [bus 07]
    [    0.238236] kernel: pci 0000:06:03.0:   bridge window [io  0xe000-0xefff]
    [    0.238241] kernel: pci 0000:06:03.0:   bridge window [mem 0xf7500000-0xf75fffff]
    [    0.238251] kernel: pci 0000:06:07.0: PCI bridge to [bus 08]
    [    0.238254] kernel: pci 0000:06:07.0:   bridge window [io  0xd000-0xdfff]
    [    0.238259] kernel: pci 0000:06:07.0:   bridge window [mem 0xf7400000-0xf74fffff]
    [    0.238269] kernel: pci 0000:05:00.0: PCI bridge to [bus 06-08]
    [    0.238271] kernel: pci 0000:05:00.0:   bridge window [io  0xd000-0xefff]
    [    0.238276] kernel: pci 0000:05:00.0:   bridge window [mem 0xf7400000-0xf75fffff]
    [    0.238286] kernel: pci 0000:04:00.0: PCI bridge to [bus 05-08]
    [    0.238287] kernel: pci 0000:04:00.0:   bridge window [io  0xd000-0xefff]
    [    0.238291] kernel: pci 0000:04:00.0:   bridge window [mem 0xf7400000-0xf75fffff]
    [    0.238297] kernel: pci 0000:0a:03.0: PCI bridge to [bus 0b]
    [    0.238300] kernel: pci 0000:0a:03.0:   bridge window [io  0xc000-0xcfff]
    [    0.238305] kernel: pci 0000:0a:03.0:   bridge window [mem 0xf7300000-0xf73fffff]
    [    0.238315] kernel: pci 0000:0a:07.0: PCI bridge to [bus 0c]
    [    0.238317] kernel: pci 0000:0a:07.0:   bridge window [io  0xb000-0xbfff]
    [    0.238322] kernel: pci 0000:0a:07.0:   bridge window [mem 0xf7200000-0xf72fffff]
    [    0.238332] kernel: pci 0000:09:00.0: PCI bridge to [bus 0a-0c]
    [    0.238334] kernel: pci 0000:09:00.0:   bridge window [io  0xb000-0xcfff]
    [    0.238339] kernel: pci 0000:09:00.0:   bridge window [mem 0xf7200000-0xf73fffff]
    [    0.238349] kernel: pci 0000:04:01.0: PCI bridge to [bus 09-0c]
    [    0.238351] kernel: pci 0000:04:01.0:   bridge window [io  0xb000-0xcfff]
    [    0.238354] kernel: pci 0000:04:01.0:   bridge window [mem 0xf7200000-0xf73fffff]
    [    0.238360] kernel: pci 0000:04:04.0: PCI bridge to [bus 0d]
    [    0.238369] kernel: pci 0000:03:00.2: PCI bridge to [bus 04-0d]
    [    0.238371] kernel: pci 0000:03:00.2:   bridge window [io  0xb000-0xefff]
    [    0.238374] kernel: pci 0000:03:00.2:   bridge window [mem 0xf7200000-0xf75fffff]
    [    0.238380] kernel: pci 0000:00:01.3: PCI bridge to [bus 03-0d]
    [    0.238381] kernel: pci 0000:00:01.3:   bridge window [io  0xb000-0xefff]
    [    0.238383] kernel: pci 0000:00:01.3:   bridge window [mem 0xf7200000-0xf76fffff]
    [    0.238385] kernel: pci 0000:00:01.3:   bridge window [mem 0xe0400000-0xe05fffff 64bit pref]
    [    0.238387] kernel: pci 0000:00:01.4: PCI bridge to [bus 0e]
    [    0.238388] kernel: pci 0000:00:01.4:   bridge window [io  0x3000-0x3fff]
    [    0.238391] kernel: pci 0000:00:01.4:   bridge window [mem 0xf7d00000-0xf7dfffff]
    [    0.238392] kernel: pci 0000:00:01.4:   bridge window [mem 0xe0600000-0xe07fffff 64bit pref]
    [    0.238395] kernel: pci 0000:00:03.1: PCI bridge to [bus 0f]
    [    0.238397] kernel: pci 0000:00:03.1:   bridge window [io  0xf000-0xffff]
    [    0.238399] kernel: pci 0000:00:03.1:   bridge window [mem 0xf6000000-0xf70fffff]
    [    0.238401] kernel: pci 0000:00:03.1:   bridge window [mem 0xe8000000-0xf1ffffff 64bit pref]
    [    0.238403] kernel: pci 0000:00:03.2: PCI bridge to [bus 10]
    [    0.238404] kernel: pci 0000:00:03.2:   bridge window [io  0x4000-0x4fff]
    [    0.238407] kernel: pci 0000:00:03.2:   bridge window [mem 0xe0800000-0xe09fffff]
    [    0.238408] kernel: pci 0000:00:03.2:   bridge window [mem 0xe0a00000-0xe0bfffff 64bit pref]
    [    0.238411] kernel: pci 0000:00:03.3: PCI bridge to [bus 11]
    [    0.238412] kernel: pci 0000:00:03.3:   bridge window [io  0x5000-0x5fff]
    [    0.238415] kernel: pci 0000:00:03.3:   bridge window [mem 0xf7c00000-0xf7cfffff]
    [    0.238416] kernel: pci 0000:00:03.3:   bridge window [mem 0xe0c00000-0xe0dfffff 64bit pref]
    [    0.238419] kernel: pci 0000:00:03.4: PCI bridge to [bus 12]
    [    0.238420] kernel: pci 0000:00:03.4:   bridge window [io  0x6000-0x6fff]
    [    0.238422] kernel: pci 0000:00:03.4:   bridge window [mem 0xf7b00000-0xf7bfffff]
    [    0.238424] kernel: pci 0000:00:03.4:   bridge window [mem 0xe0e00000-0xe0ffffff 64bit pref]
    [    0.238427] kernel: pci 0000:00:07.1: PCI bridge to [bus 13]
    [    0.238429] kernel: pci 0000:00:07.1:   bridge window [mem 0xf7700000-0xf79fffff]
    [    0.238433] kernel: pci 0000:00:08.1: PCI bridge to [bus 14]
    [    0.238435] kernel: pci 0000:00:08.1:   bridge window [mem 0xf7a00000-0xf7afffff]
    [    0.238440] kernel: pci_bus 0000:00: resource 4 [io  0x0000-0x03af window]
    [    0.238441] kernel: pci_bus 0000:00: resource 5 [io  0x03e0-0x0cf7 window]
    [    0.238442] kernel: pci_bus 0000:00: resource 6 [io  0x03b0-0x03df window]
    [    0.238443] kernel: pci_bus 0000:00: resource 7 [io  0x0d00-0xffff window]
    [    0.238443] kernel: pci_bus 0000:00: resource 8 [mem 0x000a0000-0x000bffff window]
    [    0.238444] kernel: pci_bus 0000:00: resource 9 [mem 0x000c0000-0x000dffff window]
    [    0.238445] kernel: pci_bus 0000:00: resource 10 [mem 0xe0000000-0xf7ffffff window]
    [    0.238446] kernel: pci_bus 0000:01: resource 0 [io  0x1000-0x1fff]
    [    0.238447] kernel: pci_bus 0000:01: resource 1 [mem 0xf7f00000-0xf7ffffff]
    [    0.238448] kernel: pci_bus 0000:01: resource 2 [mem 0xe0000000-0xe01fffff 64bit pref]
    [    0.238449] kernel: pci_bus 0000:02: resource 0 [io  0x2000-0x2fff]
    [    0.238449] kernel: pci_bus 0000:02: resource 1 [mem 0xf7e00000-0xf7efffff]
    [    0.238450] kernel: pci_bus 0000:02: resource 2 [mem 0xe0200000-0xe03fffff 64bit pref]
    [    0.238451] kernel: pci_bus 0000:03: resource 0 [io  0xb000-0xefff]
    [    0.238452] kernel: pci_bus 0000:03: resource 1 [mem 0xf7200000-0xf76fffff]
    [    0.238452] kernel: pci_bus 0000:03: resource 2 [mem 0xe0400000-0xe05fffff 64bit pref]
    [    0.238453] kernel: pci_bus 0000:04: resource 0 [io  0xb000-0xefff]
    [    0.238454] kernel: pci_bus 0000:04: resource 1 [mem 0xf7200000-0xf75fffff]
    [    0.238455] kernel: pci_bus 0000:05: resource 0 [io  0xd000-0xefff]
    [    0.238456] kernel: pci_bus 0000:05: resource 1 [mem 0xf7400000-0xf75fffff]
    [    0.238457] kernel: pci_bus 0000:06: resource 0 [io  0xd000-0xefff]
    [    0.238457] kernel: pci_bus 0000:06: resource 1 [mem 0xf7400000-0xf75fffff]
    [    0.238458] kernel: pci_bus 0000:07: resource 0 [io  0xe000-0xefff]
    [    0.238459] kernel: pci_bus 0000:07: resource 1 [mem 0xf7500000-0xf75fffff]
    [    0.238460] kernel: pci_bus 0000:08: resource 0 [io  0xd000-0xdfff]
    [    0.238461] kernel: pci_bus 0000:08: resource 1 [mem 0xf7400000-0xf74fffff]
    [    0.238462] kernel: pci_bus 0000:09: resource 0 [io  0xb000-0xcfff]
    [    0.238462] kernel: pci_bus 0000:09: resource 1 [mem 0xf7200000-0xf73fffff]
    [    0.238463] kernel: pci_bus 0000:0a: resource 0 [io  0xb000-0xcfff]
    [    0.238464] kernel: pci_bus 0000:0a: resource 1 [mem 0xf7200000-0xf73fffff]
    [    0.238465] kernel: pci_bus 0000:0b: resource 0 [io  0xc000-0xcfff]
    [    0.238465] kernel: pci_bus 0000:0b: resource 1 [mem 0xf7300000-0xf73fffff]
    [    0.238466] kernel: pci_bus 0000:0c: resource 0 [io  0xb000-0xbfff]
    [    0.238467] kernel: pci_bus 0000:0c: resource 1 [mem 0xf7200000-0xf72fffff]
    [    0.238468] kernel: pci_bus 0000:0e: resource 0 [io  0x3000-0x3fff]
    [    0.238469] kernel: pci_bus 0000:0e: resource 1 [mem 0xf7d00000-0xf7dfffff]
    [    0.238470] kernel: pci_bus 0000:0e: resource 2 [mem 0xe0600000-0xe07fffff 64bit pref]
    [    0.238471] kernel: pci_bus 0000:0f: resource 0 [io  0xf000-0xffff]
    [    0.238471] kernel: pci_bus 0000:0f: resource 1 [mem 0xf6000000-0xf70fffff]
    [    0.238472] kernel: pci_bus 0000:0f: resource 2 [mem 0xe8000000-0xf1ffffff 64bit pref]
    [    0.238473] kernel: pci_bus 0000:10: resource 0 [io  0x4000-0x4fff]
    [    0.238474] kernel: pci_bus 0000:10: resource 1 [mem 0xe0800000-0xe09fffff]
    [    0.238475] kernel: pci_bus 0000:10: resource 2 [mem 0xe0a00000-0xe0bfffff 64bit pref]
    [    0.238476] kernel: pci_bus 0000:11: resource 0 [io  0x5000-0x5fff]
    [    0.238476] kernel: pci_bus 0000:11: resource 1 [mem 0xf7c00000-0xf7cfffff]
    [    0.238477] kernel: pci_bus 0000:11: resource 2 [mem 0xe0c00000-0xe0dfffff 64bit pref]
    [    0.238478] kernel: pci_bus 0000:12: resource 0 [io  0x6000-0x6fff]
    [    0.238479] kernel: pci_bus 0000:12: resource 1 [mem 0xf7b00000-0xf7bfffff]
    [    0.238479] kernel: pci_bus 0000:12: resource 2 [mem 0xe0e00000-0xe0ffffff 64bit pref]
    [    0.238480] kernel: pci_bus 0000:13: resource 1 [mem 0xf7700000-0xf79fffff]
    [    0.238481] kernel: pci_bus 0000:14: resource 1 [mem 0xf7a00000-0xf7afffff]
    [    0.238560] kernel: NET: Registered protocol family 2
    [    0.238781] kernel: tcp_listen_portaddr_hash hash table entries: 32768 (order: 7, 524288 bytes, linear)
    [    0.239305] kernel: TCP established hash table entries: 524288 (order: 10, 4194304 bytes, linear)
    [    0.239873] kernel: TCP bind hash table entries: 65536 (order: 8, 1048576 bytes, linear)
    [    0.239957] kernel: TCP: Hash tables configured (established 524288 bind 65536)
    [    0.240070] kernel: UDP hash table entries: 32768 (order: 8, 1048576 bytes, linear)
    [    0.240230] kernel: UDP-Lite hash table entries: 32768 (order: 8, 1048576 bytes, linear)
    [    0.240368] kernel: NET: Registered protocol family 1
    [    0.240373] kernel: NET: Registered protocol family 44
    [    0.240619] kernel: pci 0000:0f:00.0: Video device with shadowed ROM at [mem 0x000c0000-0x000dffff]
    [    0.240632] kernel: pci 0000:0f:00.1: D0 power state depends on 0000:0f:00.0
    [    0.240788] kernel: PCI: CLS 64 bytes, default 64
    [    0.240825] kernel: Trying to unpack rootfs image as initramfs...
    [    0.356119] kernel: Freeing initrd memory: 52420K
    [    0.356149] kernel: pci 0000:00:00.2: AMD-Vi: IOMMU performance counters supported
    [    0.356489] kernel: pci 0000:00:01.0: Adding to iommu group 0
    [    0.356508] kernel: pci 0000:00:01.1: Adding to iommu group 0
    [    0.356525] kernel: pci 0000:00:01.2: Adding to iommu group 0
    [    0.356541] kernel: pci 0000:00:01.3: Adding to iommu group 0
    [    0.356559] kernel: pci 0000:00:01.4: Adding to iommu group 0
    [    0.356656] kernel: pci 0000:00:02.0: Adding to iommu group 1
    [    0.356814] kernel: pci 0000:00:03.0: Adding to iommu group 2
    [    0.356832] kernel: pci 0000:00:03.1: Adding to iommu group 2
    [    0.356850] kernel: pci 0000:00:03.2: Adding to iommu group 2
    [    0.356867] kernel: pci 0000:00:03.3: Adding to iommu group 2
    [    0.356886] kernel: pci 0000:00:03.4: Adding to iommu group 2
    [    0.357011] kernel: pci 0000:00:04.0: Adding to iommu group 3
    [    0.357105] kernel: pci 0000:00:07.0: Adding to iommu group 4
    [    0.357122] kernel: pci 0000:00:07.1: Adding to iommu group 4
    [    0.357254] kernel: pci 0000:00:08.0: Adding to iommu group 5
    [    0.357271] kernel: pci 0000:00:08.1: Adding to iommu group 5
    [    0.357366] kernel: pci 0000:00:14.0: Adding to iommu group 6
    [    0.357382] kernel: pci 0000:00:14.3: Adding to iommu group 6
    [    0.357541] kernel: pci 0000:00:18.0: Adding to iommu group 7
    [    0.357559] kernel: pci 0000:00:18.1: Adding to iommu group 7
    [    0.357576] kernel: pci 0000:00:18.2: Adding to iommu group 7
    [    0.357591] kernel: pci 0000:00:18.3: Adding to iommu group 7
    [    0.357606] kernel: pci 0000:00:18.4: Adding to iommu group 7
    [    0.357622] kernel: pci 0000:00:18.5: Adding to iommu group 7
    [    0.357637] kernel: pci 0000:00:18.6: Adding to iommu group 7
    [    0.357652] kernel: pci 0000:00:18.7: Adding to iommu group 7
    [    0.357665] kernel: pci 0000:01:00.0: Adding to iommu group 0
    [    0.357678] kernel: pci 0000:02:00.0: Adding to iommu group 0
    [    0.357692] kernel: pci 0000:03:00.0: Adding to iommu group 0
    [    0.357704] kernel: pci 0000:03:00.1: Adding to iommu group 0
    [    0.357715] kernel: pci 0000:03:00.2: Adding to iommu group 0
    [    0.357728] kernel: pci 0000:04:00.0: Adding to iommu group 0
    [    0.357740] kernel: pci 0000:04:01.0: Adding to iommu group 0
    [    0.357752] kernel: pci 0000:04:04.0: Adding to iommu group 0
    [    0.357767] kernel: pci 0000:05:00.0: Adding to iommu group 0
    [    0.357783] kernel: pci 0000:06:03.0: Adding to iommu group 0
    [    0.357797] kernel: pci 0000:06:07.0: Adding to iommu group 0
    [    0.357817] kernel: pci 0000:07:00.0: Adding to iommu group 0
    [    0.357836] kernel: pci 0000:08:00.0: Adding to iommu group 0
    [    0.357851] kernel: pci 0000:09:00.0: Adding to iommu group 0
    [    0.357865] kernel: pci 0000:0a:03.0: Adding to iommu group 0
    [    0.357879] kernel: pci 0000:0a:07.0: Adding to iommu group 0
    [    0.357900] kernel: pci 0000:0b:00.0: Adding to iommu group 0
    [    0.357919] kernel: pci 0000:0c:00.0: Adding to iommu group 0
    [    0.357931] kernel: pci 0000:0e:00.0: Adding to iommu group 0
    [    0.357944] kernel: pci 0000:0f:00.0: Adding to iommu group 2
    [    0.357954] kernel: pci 0000:0f:00.1: Adding to iommu group 2
    [    0.357966] kernel: pci 0000:11:00.0: Adding to iommu group 2
    [    0.357979] kernel: pci 0000:12:00.0: Adding to iommu group 2
    [    0.357989] kernel: pci 0000:13:00.0: Adding to iommu group 4
    [    0.358000] kernel: pci 0000:13:00.2: Adding to iommu group 4
    [    0.358010] kernel: pci 0000:13:00.3: Adding to iommu group 4
    [    0.358021] kernel: pci 0000:14:00.0: Adding to iommu group 5
    [    0.358031] kernel: pci 0000:14:00.2: Adding to iommu group 5
    [    0.358041] kernel: pci 0000:14:00.3: Adding to iommu group 5
    [    0.358304] kernel: pci 0000:00:00.2: AMD-Vi: Found IOMMU cap 0x40
    [    0.358305] kernel: pci 0000:00:00.2: AMD-Vi: Extended features (0xf77ef22294ada):
    [    0.358305] kernel:  PPR NX GT IA GA PC GA_vAPIC
    [    0.358307] kernel: AMD-Vi: Interrupt remapping enabled
    [    0.358308] kernel: AMD-Vi: Virtual APIC enabled
    [    0.358386] kernel: AMD-Vi: Lazy IO/TLB flushing enabled
    [    0.359302] kernel: amd_uncore: AMD NB counters detected
    [    0.359305] kernel: amd_uncore: AMD LLC counters detected
    [    0.359538] kernel: perf/amd_iommu: Detected AMD IOMMU #0 (2 banks, 4 counters/bank).
    [    0.359586] kernel: check: Scanning for low memory corruption every 60 seconds
    [    0.361104] kernel: Initialise system trusted keyrings
    [    0.361115] kernel: Key type blacklist registered
    [    0.361151] kernel: workingset: timestamp_bits=36 max_order=24 bucket_order=0
    [    0.362142] kernel: zbud: loaded
    [    0.362433] kernel: squashfs: version 4.0 (2009/01/31) Phillip Lougher
    [    0.362555] kernel: fuse: init (API version 7.31)
    [    0.362569] kernel: *** VALIDATE fuse ***
    [    0.362571] kernel: *** VALIDATE fuse ***
    [    0.362651] kernel: Platform Keyring initialized
    [    0.366738] kernel: Key type asymmetric registered
    [    0.366739] kernel: Asymmetric key parser 'x509' registered
    [    0.366746] kernel: Block layer SCSI generic (bsg) driver version 0.4 loaded (major 244)
    [    0.366798] kernel: io scheduler mq-deadline registered
    [    0.367924] kernel: pcieport 0000:00:01.1: PME: Signaling with IRQ 26
    [    0.367948] kernel: pcieport 0000:00:01.1: pciehp: Slot #0 AttnBtn- PwrCtrl- MRL- AttnInd- PwrInd- HotPlug+ Surprise- Interlock- NoCompl+ LLActRep+
    [    0.368135] kernel: pcieport 0000:00:01.2: PME: Signaling with IRQ 27
    [    0.368154] kernel: pcieport 0000:00:01.2: pciehp: Slot #0 AttnBtn- PwrCtrl- MRL- AttnInd- PwrInd- HotPlug+ Surprise- Interlock- NoCompl+ LLActRep+
    [    0.368330] kernel: pcieport 0000:00:01.3: PME: Signaling with IRQ 28
    [    0.368345] kernel: pcieport 0000:00:01.3: pciehp: Slot #0 AttnBtn- PwrCtrl- MRL- AttnInd- PwrInd- HotPlug+ Surprise- Interlock- NoCompl+ LLActRep+
    [    0.368915] kernel: pcieport 0000:00:01.4: PME: Signaling with IRQ 29
    [    0.368935] kernel: pcieport 0000:00:01.4: pciehp: Slot #0 AttnBtn- PwrCtrl- MRL- AttnInd- PwrInd- HotPlug+ Surprise- Interlock- NoCompl+ LLActRep+
    [    0.369118] kernel: pcieport 0000:00:03.1: PME: Signaling with IRQ 30
    [    0.369140] kernel: pcieport 0000:00:03.1: pciehp: Slot #0 AttnBtn- PwrCtrl- MRL- AttnInd- PwrInd- HotPlug+ Surprise- Interlock- NoCompl+ LLActRep+
    [    0.369324] kernel: pcieport 0000:00:03.2: PME: Signaling with IRQ 31
    [    0.369338] kernel: pcieport 0000:00:03.2: pciehp: Slot #0 AttnBtn- PwrCtrl- MRL- AttnInd- PwrInd- HotPlug+ Surprise- Interlock- NoCompl+ LLActRep+
    [    0.369917] kernel: pcieport 0000:00:03.3: PME: Signaling with IRQ 32
    [    0.369934] kernel: pcieport 0000:00:03.3: pciehp: Slot #0 AttnBtn- PwrCtrl- MRL- AttnInd- PwrInd- HotPlug+ Surprise- Interlock- NoCompl+ LLActRep+
    [    0.370115] kernel: pcieport 0000:00:03.4: PME: Signaling with IRQ 33
    [    0.370133] kernel: pcieport 0000:00:03.4: pciehp: Slot #0 AttnBtn- PwrCtrl- MRL- AttnInd- PwrInd- HotPlug+ Surprise- Interlock- NoCompl+ LLActRep+
    [    0.370311] kernel: pcieport 0000:00:07.1: PME: Signaling with IRQ 34
    [    0.370923] kernel: pcieport 0000:00:08.1: PME: Signaling with IRQ 36
    [    0.372383] kernel: shpchp: Standard Hot Plug PCI Controller Driver version: 0.4
    [    0.372427] kernel: vesafb: mode is 640x480x32, linelength=2560, pages=0
    [    0.372427] kernel: vesafb: scrolling: redraw
    [    0.372429] kernel: vesafb: Truecolor: size=8:8:8:8, shift=24:16:8:0
    [    0.372436] kernel: vesafb: framebuffer at 0xf1000000, mapped to 0x00000000ae86fef3, using 1216k, total 1216k
    [    0.372462] kernel: fbcon: Deferring console take-over
    [    0.372463] kernel: fb0: VESA VGA frame buffer device
    [    0.372562] kernel: input: Power Button as /devices/LNXSYSTM:00/LNXSYBUS:00/PNP0C0C:00/input/input0
    [    0.372576] kernel: ACPI: Power Button [PWRB]
    [    0.372602] kernel: input: Power Button as /devices/LNXSYSTM:00/LNXPWRBN:00/input/input1
    [    0.372626] kernel: ACPI: Power Button [PWRF]
    [    0.373686] kernel: Serial: 8250/16550 driver, 32 ports, IRQ sharing enabled
    [    0.394605] kernel: 00:03: ttyS0 at I/O 0x3f8 (irq = 4, base_baud = 115200) is a 16550A
    [    0.415554] kernel: 00:04: ttyS1 at I/O 0x2f8 (irq = 3, base_baud = 115200) is a 16550A
    [    0.416648] kernel: Linux agpgart interface v0.103
    [    0.419317] kernel: loop: module loaded
    [    0.419470] kernel: libphy: Fixed MDIO Bus: probed
    [    0.419471] kernel: tun: Universal TUN/TAP device driver, 1.6
    [    0.419498] kernel: PPP generic driver version 2.4.2
    [    0.419532] kernel: VFIO - User Level meta-driver version: 0.3
    [    0.419580] kernel: ehci_hcd: USB 2.0 'Enhanced' Host Controller (EHCI) Driver
    [    0.419582] kernel: ehci-pci: EHCI PCI platform driver
    [    0.419591] kernel: ehci-platform: EHCI generic platform driver
    [    0.419595] kernel: ohci_hcd: USB 1.1 'Open' Host Controller (OHCI) Driver
    [    0.419596] kernel: ohci-pci: OHCI PCI platform driver
    [    0.419603] kernel: ohci-platform: OHCI generic platform driver
    [    0.419607] kernel: uhci_hcd: USB Universal Host Controller Interface driver
    [    0.419687] kernel: xhci_hcd 0000:03:00.0: xHCI Host Controller
    [    0.419692] kernel: xhci_hcd 0000:03:00.0: new USB bus registered, assigned bus number 1
    [    0.475087] kernel: xhci_hcd 0000:03:00.0: hcc params 0x0200ef81 hci version 0x110 quirks 0x0000000048000410
    [    0.475236] kernel: usb usb1: New USB device found, idVendor=1d6b, idProduct=0002, bcdDevice= 5.04
    [    0.475237] kernel: usb usb1: New USB device strings: Mfr=3, Product=2, SerialNumber=1
    [    0.475239] kernel: usb usb1: Product: xHCI Host Controller
    [    0.475239] kernel: usb usb1: Manufacturer: Linux 5.4.0-39-generic xhci-hcd
    [    0.475240] kernel: usb usb1: SerialNumber: 0000:03:00.0
    [    0.475311] kernel: hub 1-0:1.0: USB hub found
    [    0.475323] kernel: hub 1-0:1.0: 10 ports detected
    [    0.475593] kernel: xhci_hcd 0000:03:00.0: xHCI Host Controller
    [    0.475596] kernel: xhci_hcd 0000:03:00.0: new USB bus registered, assigned bus number 2
    [    0.475598] kernel: xhci_hcd 0000:03:00.0: Host supports USB 3.1 Enhanced SuperSpeed
    [    0.475623] kernel: usb usb2: We don't know the algorithms for LPM for this host, disabling LPM.
    [    0.475638] kernel: usb usb2: New USB device found, idVendor=1d6b, idProduct=0003, bcdDevice= 5.04
    [    0.475639] kernel: usb usb2: New USB device strings: Mfr=3, Product=2, SerialNumber=1
    [    0.475640] kernel: usb usb2: Product: xHCI Host Controller
    [    0.475640] kernel: usb usb2: Manufacturer: Linux 5.4.0-39-generic xhci-hcd
    [    0.475641] kernel: usb usb2: SerialNumber: 0000:03:00.0
    [    0.475696] kernel: hub 2-0:1.0: USB hub found
    [    0.475703] kernel: hub 2-0:1.0: 4 ports detected
    [    0.475765] kernel: usb: port power management may be unreliable
    [    0.475875] kernel: xhci_hcd 0000:13:00.3: xHCI Host Controller
    [    0.475878] kernel: xhci_hcd 0000:13:00.3: new USB bus registered, assigned bus number 3
    [    0.475970] kernel: xhci_hcd 0000:13:00.3: hcc params 0x0270f665 hci version 0x100 quirks 0x0000000040000410
    [    0.476069] kernel: usb usb3: New USB device found, idVendor=1d6b, idProduct=0002, bcdDevice= 5.04
    [    0.476070] kernel: usb usb3: New USB device strings: Mfr=3, Product=2, SerialNumber=1
    [    0.476071] kernel: usb usb3: Product: xHCI Host Controller
    [    0.476072] kernel: usb usb3: Manufacturer: Linux 5.4.0-39-generic xhci-hcd
    [    0.476073] kernel: usb usb3: SerialNumber: 0000:13:00.3
    [    0.476129] kernel: hub 3-0:1.0: USB hub found
    [    0.476135] kernel: hub 3-0:1.0: 4 ports detected
    [    0.476279] kernel: xhci_hcd 0000:13:00.3: xHCI Host Controller
    [    0.476281] kernel: xhci_hcd 0000:13:00.3: new USB bus registered, assigned bus number 4
    [    0.476283] kernel: xhci_hcd 0000:13:00.3: Host supports USB 3.0 SuperSpeed
    [    0.476292] kernel: usb usb4: We don't know the algorithms for LPM for this host, disabling LPM.
    [    0.476306] kernel: usb usb4: New USB device found, idVendor=1d6b, idProduct=0003, bcdDevice= 5.04
    [    0.476307] kernel: usb usb4: New USB device strings: Mfr=3, Product=2, SerialNumber=1
    [    0.476308] kernel: usb usb4: Product: xHCI Host Controller
    [    0.476309] kernel: usb usb4: Manufacturer: Linux 5.4.0-39-generic xhci-hcd
    [    0.476309] kernel: usb usb4: SerialNumber: 0000:13:00.3
    [    0.476367] kernel: hub 4-0:1.0: USB hub found
    [    0.476373] kernel: hub 4-0:1.0: 4 ports detected
    [    0.476499] kernel: i8042: PNP: No PS/2 controller found.
    [    0.476500] kernel: i8042: Probing ports directly.
    [    1.212012] kernel: usb 1-4: new full-speed USB device number 2 using xhci_hcd
    [    1.493492] kernel: i8042: No controller found
    [    1.493526] kernel: tsc: Refined TSC clocksource calibration: 2994.376 MHz
    [    1.493534] kernel: clocksource: tsc: mask: 0xffffffffffffffff max_cycles: 0x2b2984d0609, max_idle_ns: 440795359805 ns
    [    1.493567] kernel: clocksource: Switched to clocksource tsc
    [    1.493581] kernel: mousedev: PS/2 mouse device common for all mice
    [    1.493666] kernel: rtc_cmos 00:01: RTC can wake from S4
    [    1.493937] kernel: rtc_cmos 00:01: registered as rtc0
    [    1.493949] kernel: rtc_cmos 00:01: alarms up to one month, y3k, 114 bytes nvram, hpet irqs
    [    1.493953] kernel: i2c /dev entries driver
    [    1.493989] kernel: device-mapper: uevent: version 1.0.3
    [    1.494028] kernel: device-mapper: ioctl: 4.41.0-ioctl (2019-09-16) initialised: dm-devel@redhat.com
    [    1.494039] kernel: platform eisa.0: Probing EISA bus 0
    [    1.494041] kernel: platform eisa.0: EISA: Cannot allocate resource for mainboard
    [    1.494042] kernel: platform eisa.0: Cannot allocate resource for EISA slot 1
    [    1.494043] kernel: platform eisa.0: Cannot allocate resource for EISA slot 2
    [    1.494044] kernel: platform eisa.0: Cannot allocate resource for EISA slot 3
    [    1.494044] kernel: platform eisa.0: Cannot allocate resource for EISA slot 4
    [    1.494045] kernel: platform eisa.0: Cannot allocate resource for EISA slot 5
    [    1.494046] kernel: platform eisa.0: Cannot allocate resource for EISA slot 6
    [    1.494047] kernel: platform eisa.0: Cannot allocate resource for EISA slot 7
    [    1.494047] kernel: platform eisa.0: Cannot allocate resource for EISA slot 8
    [    1.494048] kernel: platform eisa.0: EISA: Detected 0 cards
    [    1.494175] kernel: ledtrig-cpu: registered to indicate activity on CPUs
    [    1.494246] kernel: drop_monitor: Initializing network drop monitor service
    [    1.494375] kernel: NET: Registered protocol family 10
    [    1.500021] kernel: Segment Routing with IPv6
    [    1.500037] kernel: NET: Registered protocol family 17
    [    1.500076] kernel: Key type dns_resolver registered
    [    1.500890] kernel: RAS: Correctable Errors collector initialized.
    [    1.500923] kernel: microcode: CPU0: patch_level=0x08001137
    [    1.500928] kernel: microcode: CPU1: patch_level=0x08001137
    [    1.500933] kernel: microcode: CPU2: patch_level=0x08001137
    [    1.500937] kernel: microcode: CPU3: patch_level=0x08001137
    [    1.500942] kernel: microcode: CPU4: patch_level=0x08001137
    [    1.500946] kernel: microcode: CPU5: patch_level=0x08001137
    [    1.500951] kernel: microcode: CPU6: patch_level=0x08001137
    [    1.500955] kernel: microcode: CPU7: patch_level=0x08001137
    [    1.500958] kernel: microcode: CPU8: patch_level=0x08001137
    [    1.500962] kernel: microcode: CPU9: patch_level=0x08001137
    [    1.500967] kernel: microcode: CPU10: patch_level=0x08001137
    [    1.500970] kernel: microcode: CPU11: patch_level=0x08001137
    [    1.500974] kernel: microcode: CPU12: patch_level=0x08001137
    [    1.500977] kernel: microcode: CPU13: patch_level=0x08001137
    [    1.500981] kernel: microcode: CPU14: patch_level=0x08001137
    [    1.500984] kernel: microcode: CPU15: patch_level=0x08001137
    [    1.501010] kernel: microcode: Microcode Update Driver: v2.2.
    [    1.501013] kernel: IPI shorthand broadcast: enabled
    [    1.501019] kernel: sched_clock: Marking stable (1529000551, -27995801)->(1503526357, -2521607)
    [    1.501075] kernel: registered taskstats version 1
    [    1.501084] kernel: Loading compiled-in X.509 certificates
    [    1.502479] kernel: Loaded X.509 cert 'Build time autogenerated kernel key: fc5d997396576c92a1f0014cbc11f394a5a98020'
    [    1.502534] kernel: zswap: loaded using pool lzo/zbud
    [    1.502592] kernel: Key type ._fscrypt registered
    [    1.502592] kernel: Key type .fscrypt registered
    [    1.507492] kernel: Key type big_key registered
    [    1.509876] kernel: Key type encrypted registered
    [    1.509877] kernel: AppArmor: AppArmor sha1 policy hashing enabled
    [    1.509881] kernel: ima: No TPM chip found, activating TPM-bypass!
    [    1.509884] kernel: ima: Allocated hash algorithm: sha1
    [    1.509890] kernel: ima: No architecture policies found
    [    1.509897] kernel: evm: Initialising EVM extended attributes:
    [    1.509897] kernel: evm: security.selinux
    [    1.509898] kernel: evm: security.SMACK64
    [    1.509898] kernel: evm: security.SMACK64EXEC
    [    1.509898] kernel: evm: security.SMACK64TRANSMUTE
    [    1.509899] kernel: evm: security.SMACK64MMAP
    [    1.509899] kernel: evm: security.apparmor
    [    1.509899] kernel: evm: security.ima
    [    1.509900] kernel: evm: security.capability
    [    1.509900] kernel: evm: HMAC attrs: 0x1
    [    1.510459] kernel: PM:   Magic number: 8:365:486
    [    1.510548] kernel: memory memory507: hash matches
    [    1.510673] kernel: rtc_cmos 00:01: setting system clock to 2020-06-26T14:27:26 UTC (1593181646)
    [    1.510934] kernel: acpi_cpufreq: overriding BIOS provided _PSD data
    [    1.512392] kernel: Freeing unused decrypted memory: 2040K
    [    1.512753] kernel: Freeing unused kernel image memory: 2712K
    [    1.544018] kernel: Write protecting the kernel read-only data: 22528k
    [    1.544389] kernel: Freeing unused kernel image memory: 2008K
    [    1.544578] kernel: Freeing unused kernel image memory: 1192K
    [    1.553480] kernel: x86/mm: Checked W+X mappings: passed, no W+X pages found.
    [    1.553481] kernel: Run /init as init process
    [    1.554495] kernel: usb 1-4: New USB device found, idVendor=046d, idProduct=c52b, bcdDevice=12.01
    [    1.554497] kernel: usb 1-4: New USB device strings: Mfr=1, Product=2, SerialNumber=0
    [    1.554499] kernel: usb 1-4: Product: USB Receiver
    [    1.554500] kernel: usb 1-4: Manufacturer: Logitech
    [    1.570549] kernel: hidraw: raw HID events driver (C) Jiri Kosina
    [    1.622338] kernel: usbcore: registered new interface driver usbhid
    [    1.622339] kernel: usbhid: USB HID core driver
    [    1.625127] kernel: input: Logitech USB Receiver as /devices/pci0000:00/0000:00:01.3/0000:03:00.0/usb1/1-4/1-4:1.0/0003:046D:C52B.0001/input/input2
    [    1.634471] kernel: ahci 0000:01:00.0: version 3.0
    [    1.634641] kernel: ahci 0000:01:00.0: SSS flag set, parallel bus scan disabled
    [    1.634713] kernel: ahci 0000:01:00.0: AHCI 0001.0301 32 slots 12 ports 6 Gbps 0xff3 impl SATA mode
    [    1.634715] kernel: ahci 0000:01:00.0: flags: 64bit ncq sntf stag led clo only pio sxs 
    [    1.635490] kernel: scsi host0: ahci
    [    1.635609] kernel: scsi host1: ahci
    [    1.635700] kernel: scsi host2: ahci
    [    1.635782] kernel: scsi host3: ahci
    [    1.636352] kernel: scsi host4: ahci
    [    1.636435] kernel: scsi host5: ahci
    [    1.636505] kernel: scsi host6: ahci
    [    1.636599] kernel: scsi host7: ahci
    [    1.636667] kernel: scsi host8: ahci
    [    1.639254] kernel: piix4_smbus 0000:00:14.0: SMBus Host Controller at 0xb00, revision 0
    [    1.639256] kernel: piix4_smbus 0000:00:14.0: Using register 0x02 for SMBus port selection
    [    1.639342] kernel: scsi host9: ahci
    [    1.639442] kernel: scsi host10: ahci
    [    1.639613] kernel: scsi host11: ahci
    [    1.639649] kernel: ata1: SATA max UDMA/133 abar m8192@0xf7f80000 port 0xf7f80100 irq 54
    [    1.639651] kernel: ata2: SATA max UDMA/133 abar m8192@0xf7f80000 port 0xf7f80180 irq 54
    [    1.639652] kernel: ata3: DUMMY
    [    1.639653] kernel: ata4: DUMMY
    [    1.639654] kernel: ata5: SATA max UDMA/133 abar m8192@0xf7f80000 port 0xf7f80300 irq 54
    [    1.639656] kernel: ata6: SATA max UDMA/133 abar m8192@0xf7f80000 port 0xf7f80380 irq 54
    [    1.639658] kernel: ata7: SATA max UDMA/133 abar m8192@0xf7f80000 port 0xf7f80400 irq 54
    [    1.639659] kernel: ata8: SATA max UDMA/133 abar m8192@0xf7f80000 port 0xf7f80480 irq 54
    [    1.639661] kernel: ata9: SATA max UDMA/133 abar m8192@0xf7f80000 port 0xf7f80500 irq 54
    [    1.639662] kernel: ata10: SATA max UDMA/133 abar m8192@0xf7f80000 port 0xf7f80580 irq 54
    [    1.639663] kernel: ata11: SATA max UDMA/133 abar m8192@0xf7f80000 port 0xf7f80600 irq 54
    [    1.639665] kernel: ata12: SATA max UDMA/133 abar m8192@0xf7f80000 port 0xf7f80680 irq 54
    [    1.639855] kernel: ahci 0000:02:00.0: SSS flag set, parallel bus scan disabled
    [    1.639939] kernel: ahci 0000:02:00.0: AHCI 0001.0301 32 slots 12 ports 6 Gbps 0xff3 impl SATA mode
    [    1.639940] kernel: ahci 0000:02:00.0: flags: 64bit ncq sntf stag led clo only pio sxs 
    [    1.640752] kernel: scsi host12: ahci
    [    1.640932] kernel: scsi host13: ahci
    [    1.641005] kernel: scsi host14: ahci
    [    1.646218] kernel: scsi host15: ahci
    [    1.646577] kernel: scsi host16: ahci
    [    1.647564] kernel: scsi host17: ahci
    [    1.648245] kernel: scsi host18: ahci
    [    1.648426] kernel: scsi host19: ahci
    [    1.652053] kernel: scsi host20: ahci
    [    1.656046] kernel: scsi host21: ahci
    [    1.656143] kernel: scsi host22: ahci
    [    1.660025] kernel: scsi host23: ahci
    [    1.660073] kernel: ata13: SATA max UDMA/133 abar m8192@0xf7e80000 port 0xf7e80100 irq 56
    [    1.660075] kernel: ata14: SATA max UDMA/133 abar m8192@0xf7e80000 port 0xf7e80180 irq 56
    [    1.660076] kernel: ata15: DUMMY
    [    1.660076] kernel: ata16: DUMMY
    [    1.660078] kernel: ata17: SATA max UDMA/133 abar m8192@0xf7e80000 port 0xf7e80300 irq 56
    [    1.660080] kernel: ata18: SATA max UDMA/133 abar m8192@0xf7e80000 port 0xf7e80380 irq 56
    [    1.660082] kernel: ata19: SATA max UDMA/133 abar m8192@0xf7e80000 port 0xf7e80400 irq 56
    [    1.660083] kernel: ata20: SATA max UDMA/133 abar m8192@0xf7e80000 port 0xf7e80480 irq 56
    [    1.660085] kernel: ata21: SATA max UDMA/133 abar m8192@0xf7e80000 port 0xf7e80500 irq 56
    [    1.660087] kernel: ata22: SATA max UDMA/133 abar m8192@0xf7e80000 port 0xf7e80580 irq 56
    [    1.660088] kernel: ata23: SATA max UDMA/133 abar m8192@0xf7e80000 port 0xf7e80600 irq 56
    [    1.660090] kernel: ata24: SATA max UDMA/133 abar m8192@0xf7e80000 port 0xf7e80680 irq 56
    [    1.660308] kernel: ahci 0000:03:00.1: SSS flag set, parallel bus scan disabled
    [    1.660343] kernel: ahci 0000:03:00.1: AHCI 0001.0301 32 slots 8 ports 6 Gbps 0x33 impl SATA mode
    [    1.660344] kernel: ahci 0000:03:00.1: flags: 64bit ncq sntf stag pm led clo only pmp pio slum part sxs deso sadm sds apst 
    [    1.660908] kernel: scsi host24: ahci
    [    1.661056] kernel: scsi host25: ahci
    [    1.661128] kernel: scsi host26: ahci
    [    1.661221] kernel: scsi host27: ahci
    [    1.661294] kernel: scsi host28: ahci
    [    1.661404] kernel: scsi host29: ahci
    [    1.661470] kernel: scsi host30: ahci
    [    1.661545] kernel: scsi host31: ahci
    [    1.661581] kernel: ata25: SATA max UDMA/133 abar m131072@0xf7680000 port 0xf7680100 irq 57
    [    1.661583] kernel: ata26: SATA max UDMA/133 abar m131072@0xf7680000 port 0xf7680180 irq 57
    [    1.661583] kernel: ata27: DUMMY
    [    1.661583] kernel: ata28: DUMMY
    [    1.661585] kernel: ata29: SATA max UDMA/133 abar m131072@0xf7680000 port 0xf7680300 irq 57
    [    1.661586] kernel: ata30: SATA max UDMA/133 abar m131072@0xf7680000 port 0xf7680380 irq 57
    [    1.661586] kernel: ata31: DUMMY
    [    1.661587] kernel: ata32: DUMMY
    [    1.661772] kernel: ahci 0000:0e:00.0: SSS flag set, parallel bus scan disabled
    [    1.661851] kernel: ahci 0000:0e:00.0: AHCI 0001.0301 32 slots 12 ports 6 Gbps 0xff3 impl SATA mode
    [    1.661852] kernel: ahci 0000:0e:00.0: flags: 64bit ncq sntf stag led clo only pio sxs 
    [    1.662649] kernel: scsi host32: ahci
    [    1.664181] kernel: scsi host33: ahci
    [    1.664254] kernel: scsi host34: ahci
    [    1.664326] kernel: scsi host35: ahci
    [    1.664393] kernel: scsi host36: ahci
    [    1.664486] kernel: scsi host37: ahci
    [    1.664556] kernel: scsi host38: ahci
    [    1.664636] kernel: scsi host39: ahci
    [    1.664700] kernel: scsi host40: ahci
    [    1.664764] kernel: scsi host41: ahci
    [    1.672022] kernel: scsi host42: ahci
    [    1.672097] kernel: scsi host43: ahci
    [    1.672128] kernel: ata33: SATA max UDMA/133 abar m8192@0xf7d80000 port 0xf7d80100 irq 59
    [    1.672130] kernel: ata34: SATA max UDMA/133 abar m8192@0xf7d80000 port 0xf7d80180 irq 59
    [    1.672130] kernel: ata35: DUMMY
    [    1.672131] kernel: ata36: DUMMY
    [    1.672132] kernel: ata37: SATA max UDMA/133 abar m8192@0xf7d80000 port 0xf7d80300 irq 59
    [    1.672134] kernel: ata38: SATA max UDMA/133 abar m8192@0xf7d80000 port 0xf7d80380 irq 59
    [    1.672135] kernel: ata39: SATA max UDMA/133 abar m8192@0xf7d80000 port 0xf7d80400 irq 59
    [    1.672137] kernel: ata40: SATA max UDMA/133 abar m8192@0xf7d80000 port 0xf7d80480 irq 59
    [    1.672139] kernel: ata41: SATA max UDMA/133 abar m8192@0xf7d80000 port 0xf7d80500 irq 59
    [    1.672140] kernel: ata42: SATA max UDMA/133 abar m8192@0xf7d80000 port 0xf7d80580 irq 59
    [    1.672142] kernel: ata43: SATA max UDMA/133 abar m8192@0xf7d80000 port 0xf7d80600 irq 59
    [    1.672143] kernel: ata44: SATA max UDMA/133 abar m8192@0xf7d80000 port 0xf7d80680 irq 59
    [    1.672295] kernel: ahci 0000:11:00.0: SSS flag set, parallel bus scan disabled
    [    1.672372] kernel: ahci 0000:11:00.0: AHCI 0001.0301 32 slots 12 ports 6 Gbps 0xff3 impl SATA mode
    [    1.672374] kernel: ahci 0000:11:00.0: flags: 64bit ncq sntf stag led clo only pio sxs 
    [    1.673216] kernel: scsi host44: ahci
    [    1.673281] kernel: scsi host45: ahci
    [    1.673354] kernel: scsi host46: ahci
    [    1.673431] kernel: scsi host47: ahci
    [    1.673507] kernel: scsi host48: ahci
    [    1.673577] kernel: scsi host49: ahci
    [    1.680023] kernel: scsi host50: ahci
    [    1.680092] kernel: scsi host51: ahci
    [    1.680174] kernel: scsi host52: ahci
    [    1.680242] kernel: scsi host53: ahci
    [    1.680320] kernel: scsi host54: ahci
    [    1.680388] kernel: scsi host55: ahci
    [    1.680428] kernel: ata45: SATA max UDMA/133 abar m8192@0xf7c80000 port 0xf7c80100 irq 61
    [    1.680429] kernel: ata46: SATA max UDMA/133 abar m8192@0xf7c80000 port 0xf7c80180 irq 61
    [    1.680430] kernel: ata47: DUMMY
    [    1.680430] kernel: ata48: DUMMY
    [    1.680431] kernel: ata49: SATA max UDMA/133 abar m8192@0xf7c80000 port 0xf7c80300 irq 61
    [    1.680433] kernel: ata50: SATA max UDMA/133 abar m8192@0xf7c80000 port 0xf7c80380 irq 61
    [    1.680434] kernel: ata51: SATA max UDMA/133 abar m8192@0xf7c80000 port 0xf7c80400 irq 61
    [    1.680436] kernel: ata52: SATA max UDMA/133 abar m8192@0xf7c80000 port 0xf7c80480 irq 61
    [    1.680437] kernel: ata53: SATA max UDMA/133 abar m8192@0xf7c80000 port 0xf7c80500 irq 61
    [    1.680439] kernel: ata54: SATA max UDMA/133 abar m8192@0xf7c80000 port 0xf7c80580 irq 61
    [    1.680440] kernel: ata55: SATA max UDMA/133 abar m8192@0xf7c80000 port 0xf7c80600 irq 61
    [    1.680441] kernel: ata56: SATA max UDMA/133 abar m8192@0xf7c80000 port 0xf7c80680 irq 61
    [    1.680587] kernel: ahci 0000:12:00.0: SSS flag set, parallel bus scan disabled
    [    1.680667] kernel: ahci 0000:12:00.0: AHCI 0001.0301 32 slots 12 ports 6 Gbps 0xff3 impl SATA mode
    [    1.680669] kernel: ahci 0000:12:00.0: flags: 64bit ncq sntf stag led clo only pio sxs 
    [    1.681547] kernel: scsi host56: ahci
    [    1.681632] kernel: scsi host57: ahci
    [    1.681708] kernel: scsi host58: ahci
    [    1.681789] kernel: scsi host59: ahci
    [    1.681857] kernel: scsi host60: ahci
    [    1.681935] kernel: scsi host61: ahci
    [    1.682012] kernel: scsi host62: ahci
    [    1.682102] kernel: scsi host63: ahci
    [    1.682170] kernel: scsi host64: ahci
    [    1.682256] kernel: scsi host65: ahci
    [    1.682337] kernel: scsi host66: ahci
    [    1.682420] kernel: scsi host67: ahci
    [    1.682458] kernel: ata57: SATA max UDMA/133 abar m8192@0xf7b80000 port 0xf7b80100 irq 63
    [    1.682459] kernel: ata58: SATA max UDMA/133 abar m8192@0xf7b80000 port 0xf7b80180 irq 63
    [    1.682460] kernel: ata59: DUMMY
    [    1.682460] kernel: ata60: DUMMY
    [    1.682462] kernel: ata61: SATA max UDMA/133 abar m8192@0xf7b80000 port 0xf7b80300 irq 63
    [    1.682463] kernel: ata62: SATA max UDMA/133 abar m8192@0xf7b80000 port 0xf7b80380 irq 63
    [    1.682464] kernel: ata63: SATA max UDMA/133 abar m8192@0xf7b80000 port 0xf7b80400 irq 63
    [    1.682466] kernel: ata64: SATA max UDMA/133 abar m8192@0xf7b80000 port 0xf7b80480 irq 63
    [    1.682467] kernel: ata65: SATA max UDMA/133 abar m8192@0xf7b80000 port 0xf7b80500 irq 63
    [    1.682469] kernel: ata66: SATA max UDMA/133 abar m8192@0xf7b80000 port 0xf7b80580 irq 63
    [    1.682470] kernel: ata67: SATA max UDMA/133 abar m8192@0xf7b80000 port 0xf7b80600 irq 63
    [    1.682472] kernel: ata68: SATA max UDMA/133 abar m8192@0xf7b80000 port 0xf7b80680 irq 63
    [    1.682589] kernel: ahci 0000:14:00.2: AHCI 0001.0301 32 slots 1 ports 6 Gbps 0x1 impl SATA mode
    [    1.682591] kernel: ahci 0000:14:00.2: flags: 64bit ncq sntf ilck pm led clo only pmp fbs pio slum part 
    [    1.682732] kernel: scsi host68: ahci
    [    1.682770] kernel: ata69: SATA max UDMA/133 abar m4096@0xf7a08000 port 0xf7a08100 irq 64
    [    1.684115] kernel: hid-generic 0003:046D:C52B.0001: input,hidraw0: USB HID v1.11 Keyboard [Logitech USB Receiver] on usb-0000:03:00.0-4/input0
    [    1.684380] kernel: input: Logitech USB Receiver Mouse as /devices/pci0000:00/0000:00:01.3/0000:03:00.0/usb1/1-4/1-4:1.1/0003:046D:C52B.0002/input/input3
    [    1.684482] kernel: input: Logitech USB Receiver Consumer Control as /devices/pci0000:00/0000:00:01.3/0000:03:00.0/usb1/1-4/1-4:1.1/0003:046D:C52B.0002/input/input4
    [    1.696015] kernel: usb 1-10: new high-speed USB device number 3 using xhci_hcd
    [    1.744095] kernel: input: Logitech USB Receiver System Control as /devices/pci0000:00/0000:00:01.3/0000:03:00.0/usb1/1-4/1-4:1.1/0003:046D:C52B.0002/input/input5
    [    1.744205] kernel: hid-generic 0003:046D:C52B.0002: input,hiddev0,hidraw1: USB HID v1.11 Mouse [Logitech USB Receiver] on usb-0000:03:00.0-4/input1
    [    1.744443] kernel: hid-generic 0003:046D:C52B.0003: hiddev1,hidraw2: USB HID v1.11 Device [Logitech USB Receiver] on usb-0000:03:00.0-4/input2
    [    1.871753] kernel: usb 1-10: New USB device found, idVendor=1005, idProduct=b155, bcdDevice= 1.00
    [    1.871755] kernel: usb 1-10: New USB device strings: Mfr=1, Product=2, SerialNumber=3
    [    1.871756] kernel: usb 1-10: Product: USB DISK MODULE
    [    1.871757] kernel: usb 1-10: Manufacturer:         
    [    1.871758] kernel: usb 1-10: SerialNumber: 19008818A6D2B616
    [    1.952403] kernel: logitech-djreceiver 0003:046D:C52B.0003: hiddev0,hidraw0: USB HID v1.11 Device [Logitech USB Receiver] on usb-0000:03:00.0-4/input2
    [    1.954130] kernel: ata1: SATA link down (SStatus 0 SControl 300)
    [    1.974138] kernel: ata13: SATA link down (SStatus 0 SControl 300)
    [    1.974140] kernel: ata25: SATA link down (SStatus 0 SControl 300)
    [    1.986138] kernel: ata33: SATA link down (SStatus 0 SControl 300)
    [    1.994127] kernel: ata69: SATA link down (SStatus 0 SControl 300)
    [    1.994139] kernel: ata45: SATA link down (SStatus 0 SControl 300)
    [    1.994142] kernel: ata57: SATA link up 6.0 Gbps (SStatus 133 SControl 300)
    [    1.996272] kernel: ata57.00: supports DRM functions and may not be fully accessible
    [    1.996711] kernel: ata57.00: ATA-11: Samsung SSD 860 EVO 4TB, RVT03B6Q, max UDMA/133
    [    1.996713] kernel: ata57.00: 7814037168 sectors, multi 1: LBA48 NCQ (depth 32), AA
    [    1.998498] kernel: ata57.00: supports DRM functions and may not be fully accessible
    [    2.000406] kernel: ata57.00: configured for UDMA/133
    [    2.074458] kernel: input: Logitech Unifying Device. Wireless PID:1028 Mouse as /devices/pci0000:00/0000:00:01.3/0000:03:00.0/usb1/1-4/1-4:1.2/0003:046D:C52B.0003/0003:046D:1028.0004/input/input7
    [    2.074527] kernel: hid-generic 0003:046D:1028.0004: input,hidraw1: USB HID v1.11 Mouse [Logitech Unifying Device. Wireless PID:1028] on usb-0000:03:00.0-4/input2:1
    [    2.076520] kernel: input: Logitech Unifying Device. Wireless PID:4003 Keyboard as /devices/pci0000:00/0000:00:01.3/0000:03:00.0/usb1/1-4/1-4:1.2/0003:046D:C52B.0003/0003:046D:4003.0005/input/input11
    [    2.076570] kernel: input: Logitech Unifying Device. Wireless PID:4003 Consumer Control as /devices/pci0000:00/0000:00:01.3/0000:03:00.0/usb1/1-4/1-4:1.2/0003:046D:C52B.0003/0003:046D:4003.0005/input/input12
    [    2.076606] kernel: input: Logitech Unifying Device. Wireless PID:4003 System Control as /devices/pci0000:00/0000:00:01.3/0000:03:00.0/usb1/1-4/1-4:1.2/0003:046D:C52B.0003/0003:046D:4003.0005/input/input13
    [    2.076647] kernel: hid-generic 0003:046D:4003.0005: input,hidraw2: USB HID v1.11 Keyboard [Logitech Unifying Device. Wireless PID:4003] on usb-0000:03:00.0-4/input2:2
    [    2.096476] kernel: dca service started, version 1.12.1
    [    2.096533] kernel: usb-storage 1-10:1.0: USB Mass Storage device detected
    [    2.096757] kernel: scsi host69: usb-storage 1-10:1.0
    [    2.096829] kernel: usbcore: registered new interface driver usb-storage
    [    2.099575] kernel: usbcore: registered new interface driver uas
    [    2.100094] kernel: igb: Intel(R) Gigabit Ethernet Network Driver - version 5.6.0-k
    [    2.100096] kernel: igb: Copyright (c) 2007-2014 Intel Corporation.
    [    2.129672] kernel: pps pps0: new PPS source ptp0
    [    2.129696] kernel: igb 0000:07:00.0: added PHC on eth0
    [    2.129697] kernel: igb 0000:07:00.0: Intel(R) Gigabit Ethernet Network Connection
    [    2.129698] kernel: igb 0000:07:00.0: eth0: (PCIe:2.5Gb/s:Width x1) 24:5e:be:36:22:44
    [    2.129699] kernel: igb 0000:07:00.0: eth0: PBA No: FFFFFF-0FF
    [    2.129700] kernel: igb 0000:07:00.0: Using MSI-X interrupts. 2 rx queue(s), 2 tx queue(s)
    [    2.159128] kernel: pps pps1: new PPS source ptp1
    [    2.159151] kernel: igb 0000:08:00.0: added PHC on eth1
    [    2.159152] kernel: igb 0000:08:00.0: Intel(R) Gigabit Ethernet Network Connection
    [    2.159153] kernel: igb 0000:08:00.0: eth1: (PCIe:2.5Gb/s:Width x1) 24:5e:be:36:22:45
    [    2.159154] kernel: igb 0000:08:00.0: eth1: PBA No: FFFFFF-0FF
    [    2.159155] kernel: igb 0000:08:00.0: Using MSI-X interrupts. 2 rx queue(s), 2 tx queue(s)
    [    2.182339] kernel: input: Logitech M570 as /devices/pci0000:00/0000:00:01.3/0000:03:00.0/usb1/1-4/1-4:1.2/0003:046D:C52B.0003/0003:046D:1028.0004/input/input17
    [    2.182417] kernel: logitech-hidpp-device 0003:046D:1028.0004: input,hidraw1: USB HID v1.11 Mouse [Logitech M570] on usb-0000:03:00.0-4/input2:1
    [    2.188355] kernel: pps pps2: new PPS source ptp2
    [    2.188377] kernel: igb 0000:0b:00.0: added PHC on eth2
    [    2.188378] kernel: igb 0000:0b:00.0: Intel(R) Gigabit Ethernet Network Connection
    [    2.188379] kernel: igb 0000:0b:00.0: eth2: (PCIe:2.5Gb/s:Width x1) 24:5e:be:36:22:46
    [    2.188380] kernel: igb 0000:0b:00.0: eth2: PBA No: FFFFFF-0FF
    [    2.188381] kernel: igb 0000:0b:00.0: Using MSI-X interrupts. 2 rx queue(s), 2 tx queue(s)
    [    2.217801] kernel: pps pps3: new PPS source ptp3
    [    2.217823] kernel: igb 0000:0c:00.0: added PHC on eth3
    [    2.217824] kernel: igb 0000:0c:00.0: Intel(R) Gigabit Ethernet Network Connection
    [    2.217825] kernel: igb 0000:0c:00.0: eth3: (PCIe:2.5Gb/s:Width x1) 24:5e:be:36:22:47
    [    2.217826] kernel: igb 0000:0c:00.0: eth3: PBA No: FFFFFF-0FF
    [    2.217827] kernel: igb 0000:0c:00.0: Using MSI-X interrupts. 2 rx queue(s), 2 tx queue(s)
    [    2.225434] kernel: igb 0000:07:00.0 enp7s0: renamed from eth0
    [    2.264152] kernel: igb 0000:08:00.0 enp8s0: renamed from eth1
    [    2.266128] kernel: ata2: SATA link down (SStatus 0 SControl 300)
    [    2.344082] kernel: igb 0000:0b:00.0 enp11s0: renamed from eth2
    [    2.372084] kernel: igb 0000:0c:00.0 enp12s0: renamed from eth3
    [    2.490426] kernel: input: Logitech K270 as /devices/pci0000:00/0000:00:01.3/0000:03:00.0/usb1/1-4/1-4:1.2/0003:046D:C52B.0003/0003:046D:4003.0005/input/input18
    [    2.490469] kernel: logitech-hidpp-device 0003:046D:4003.0005: input,hidraw2: USB HID v1.11 Keyboard [Logitech K270] on usb-0000:03:00.0-4/input2:2
    [    2.582136] kernel: ata5: SATA link down (SStatus 0 SControl 300)
    [    2.894129] kernel: ata6: SATA link down (SStatus 0 SControl 300)
    [    3.108193] kernel: scsi 69:0:0:0: Direct-Access              USB DISK MODULE  PMAP PQ: 0 ANSI: 0 CCS
    [    3.108344] kernel: sd 69:0:0:0: Attached scsi generic sg0 type 0
    [    3.109235] kernel: sd 69:0:0:0: [sda] 8060928 512-byte logical blocks: (4.13 GB/3.84 GiB)
    [    3.111921] kernel: sd 69:0:0:0: [sda] Write Protect is off
    [    3.111922] kernel: sd 69:0:0:0: [sda] Mode Sense: 23 00 00 00
    [    3.114594] kernel: sd 69:0:0:0: [sda] No Caching mode page found
    [    3.114596] kernel: sd 69:0:0:0: [sda] Assuming drive cache: write through
    [    3.155576] kernel:  sda: sda1
    [    3.161364] kernel: sd 69:0:0:0: [sda] Attached SCSI removable disk
    [    3.206148] kernel: ata7: SATA link down (SStatus 0 SControl 300)
    [    3.518128] kernel: ata8: SATA link down (SStatus 0 SControl 300)
    [    3.830134] kernel: ata9: SATA link down (SStatus 0 SControl 300)
    [    4.142126] kernel: ata10: SATA link down (SStatus 0 SControl 300)
    [    4.454129] kernel: ata11: SATA link down (SStatus 0 SControl 300)
    [    4.766133] kernel: ata12: SATA link down (SStatus 0 SControl 300)
    [    5.078129] kernel: ata14: SATA link down (SStatus 0 SControl 300)
    [    5.390129] kernel: ata17: SATA link down (SStatus 0 SControl 300)
    [    5.702131] kernel: ata18: SATA link down (SStatus 0 SControl 300)
    [    6.014127] kernel: ata19: SATA link down (SStatus 0 SControl 300)
    [    6.326127] kernel: ata20: SATA link down (SStatus 0 SControl 300)
    [    6.638131] kernel: ata21: SATA link down (SStatus 0 SControl 300)
    [    6.950138] kernel: ata22: SATA link down (SStatus 0 SControl 300)
    [    7.262127] kernel: ata23: SATA link down (SStatus 0 SControl 300)
    [    7.574127] kernel: ata24: SATA link down (SStatus 0 SControl 300)
    [    7.886121] kernel: ata26: SATA link down (SStatus 0 SControl 300)
    [    8.360016] kernel: ata29: SATA link up 6.0 Gbps (SStatus 133 SControl 300)
    [    8.361293] kernel: ata29.00: ATA-11: WDC  WDS100T2B0B-00YS70, 401020WD, max UDMA/133
    [    8.361294] kernel: ata29.00: 1953525168 sectors, multi 1: LBA48 NCQ (depth 32), AA
    [    8.363497] kernel: ata29.00: configured for UDMA/133
    [    8.363569] kernel: scsi 28:0:0:0: Direct-Access     ATA      WDC  WDS100T2B0B 20WD PQ: 0 ANSI: 5
    [    8.363676] kernel: sd 28:0:0:0: Attached scsi generic sg1 type 0
    [    8.363784] kernel: sd 28:0:0:0: [sdb] 1953525168 512-byte logical blocks: (1.00 TB/932 GiB)
    [    8.363793] kernel: sd 28:0:0:0: [sdb] Write Protect is off
    [    8.363794] kernel: sd 28:0:0:0: [sdb] Mode Sense: 00 3a 00 00
    [    8.363809] kernel: sd 28:0:0:0: [sdb] Write cache: enabled, read cache: enabled, doesn't support DPO or FUA
    [    8.389257] kernel:  sdb: sdb1
    [    8.389519] kernel: sd 28:0:0:0: [sdb] Attached SCSI disk
    [    8.678125] kernel: ata30: SATA link down (SStatus 0 SControl 300)
    [    8.990130] kernel: ata34: SATA link down (SStatus 0 SControl 300)
    [    9.302127] kernel: ata37: SATA link down (SStatus 0 SControl 300)
    [    9.614130] kernel: ata38: SATA link down (SStatus 0 SControl 300)
    [    9.926129] kernel: ata39: SATA link down (SStatus 0 SControl 300)
    [   10.238128] kernel: ata40: SATA link down (SStatus 0 SControl 300)
    [   10.550128] kernel: ata41: SATA link down (SStatus 0 SControl 300)
    [   10.862128] kernel: ata42: SATA link down (SStatus 0 SControl 300)
    [   11.174130] kernel: ata43: SATA link down (SStatus 0 SControl 300)
    [   11.486127] kernel: ata44: SATA link down (SStatus 0 SControl 300)
    [   11.798126] kernel: ata46: SATA link down (SStatus 0 SControl 300)
    [   12.110130] kernel: ata49: SATA link down (SStatus 0 SControl 300)
    [   12.422127] kernel: ata50: SATA link down (SStatus 0 SControl 300)
    [   12.734126] kernel: ata51: SATA link down (SStatus 0 SControl 300)
    [   13.046130] kernel: ata52: SATA link down (SStatus 0 SControl 300)
    [   13.358126] kernel: ata53: SATA link down (SStatus 0 SControl 300)
    [   13.670125] kernel: ata54: SATA link down (SStatus 0 SControl 300)
    [   13.982127] kernel: ata55: SATA link down (SStatus 0 SControl 300)
    [   14.294126] kernel: ata56: SATA link down (SStatus 0 SControl 300)
    [   14.294246] kernel: scsi 56:0:0:0: Direct-Access     ATA      Samsung SSD 860  3B6Q PQ: 0 ANSI: 5
    [   14.294388] kernel: ata57.00: Enabling discard_zeroes_data
    [   14.294392] kernel: sd 56:0:0:0: Attached scsi generic sg2 type 0
    [   14.294455] kernel: sd 56:0:0:0: [sdc] 7814037168 512-byte logical blocks: (4.00 TB/3.64 TiB)
    [   14.294466] kernel: sd 56:0:0:0: [sdc] Write Protect is off
    [   14.294468] kernel: sd 56:0:0:0: [sdc] Mode Sense: 00 3a 00 00
    [   14.294482] kernel: sd 56:0:0:0: [sdc] Write cache: enabled, read cache: enabled, doesn't support DPO or FUA
    [   14.308083] kernel: ata57.00: Enabling discard_zeroes_data
    [   14.310672] kernel: ata57.00: Enabling discard_zeroes_data
    [   14.311699] kernel: sd 56:0:0:0: [sdc] supports TCG Opal
    [   14.311701] kernel: sd 56:0:0:0: [sdc] Attached SCSI disk
    [   14.606129] kernel: ata58: SATA link up 6.0 Gbps (SStatus 133 SControl 300)
    [   14.608382] kernel: ata58.00: supports DRM functions and may not be fully accessible
    [   14.608819] kernel: ata58.00: ATA-11: Samsung SSD 860 EVO 4TB, RVT03B6Q, max UDMA/133
    [   14.608820] kernel: ata58.00: 7814037168 sectors, multi 1: LBA48 NCQ (depth 32), AA
    [   14.610606] kernel: ata58.00: supports DRM functions and may not be fully accessible
    [   14.612511] kernel: ata58.00: configured for UDMA/133
    [   14.612576] kernel: scsi 57:0:0:0: Direct-Access     ATA      Samsung SSD 860  3B6Q PQ: 0 ANSI: 5
    [   14.612678] kernel: ata58.00: Enabling discard_zeroes_data
    [   14.612681] kernel: sd 57:0:0:0: Attached scsi generic sg3 type 0
    [   14.612735] kernel: sd 57:0:0:0: [sdd] 7814037168 512-byte logical blocks: (4.00 TB/3.64 TiB)
    [   14.612744] kernel: sd 57:0:0:0: [sdd] Write Protect is off
    [   14.612745] kernel: sd 57:0:0:0: [sdd] Mode Sense: 00 3a 00 00
    [   14.612764] kernel: sd 57:0:0:0: [sdd] Write cache: enabled, read cache: enabled, doesn't support DPO or FUA
    [   14.628076] kernel: ata58.00: Enabling discard_zeroes_data
    [   14.628622] kernel: ata58.00: Enabling discard_zeroes_data
    [   14.629651] kernel: sd 57:0:0:0: [sdd] supports TCG Opal
    [   14.629652] kernel: sd 57:0:0:0: [sdd] Attached SCSI disk
    [   14.926131] kernel: ata61: SATA link down (SStatus 0 SControl 300)
    [   15.238128] kernel: ata62: SATA link down (SStatus 0 SControl 300)
    [   15.550128] kernel: ata63: SATA link down (SStatus 0 SControl 300)
    [   15.862132] kernel: ata64: SATA link down (SStatus 0 SControl 300)
    [   16.174151] kernel: ata65: SATA link down (SStatus 0 SControl 300)
    [   16.486126] kernel: ata66: SATA link down (SStatus 0 SControl 300)
    [   16.798125] kernel: ata67: SATA link down (SStatus 0 SControl 300)
    [   17.110132] kernel: ata68: SATA link down (SStatus 0 SControl 300)
