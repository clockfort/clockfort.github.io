---
layout: post
title: "Making a Custom Linux Build"
date: 2013-08-16 15:33
comments: true
categories: [Linux, hacking, hardware]
---

I'm borrowing the OpenWRT project's surprisingly easy to use cross-compilation toolchain to make a custom 3.10 Linux kernel, ramdisk, and root image for this device. The end result isn't going to look anything like OpenWRT, though, more like a cross-Linux-from-Scratch install, since my needs are very different than normal OpenWRT uses (I am not limited to a horrid platform with ~16MB of flash, ~8MB of RAM, a single slow ARM CPU; I have GB/TB of disks, 4GB of RAM, 16 CPUs... and a weird architecture) so it's pretty much me having to cross-compile and build a root filesystem disk image of an entire standard linux distro from source for a fairly rare MIPS platform.

There are a lot of pains about this - kernel config is a little weird for this platform, and I found a few places that won't work (or even compile) for MIPS/Octeon platform. The kernel itself is easy, though, compared to building out a good-sized userspace for this platform. Some applications have x86/ARM/etc specific code and can't compile for MIPS; some applications have bad building dependency graphs and can't be built in parallel; other applications would work fine on MIPS, and compile reliably in parallel, but their trunk builds are currently broken for one reason or another. Or others use outdated kernel interfaces and can't build against 3.10 kernels.

So, as a result, the build process is:

* Do kernel config, select packages to build, add in a few extras that I'll need that aren't tracked by OpenWRT

* Start building

* Stop on an error

* Fix error if it's a package I need, or toss out the application if it's a feature that is more of a "would-be-nice" than a "must-have"

* Start building again with insane verbosity options enabled

* Stop on an error... (repeat for hours and hours)


Pro-tip: There are an absurd number of wireless kernel modules that don't compile/work on MIPS64. x86 binary firmware blobs are stupid.

After finding a kernel config with the maximum number of features I wanted (read: lots of weird networking / routing stuff, as well as heavy amounts of kernel debugging tool support) that didn't make my $(CC) barf all over the screen, and patching the build systems for a few applications I wanted to use (for some reason the HEAD of ISC's DHCPD has some non-gnu-make specific shit in it, among a few other things that are a little off), I finally managed to get a working bunch of build scripts. That said, the actual compilation process takes well over an hour on my i7, mostly due to restrictive -j1 settings on my part, because I was tired of dealing with a few packages that didn't properly enumerate their dependency graphs. This process also spits out over 50 MEGABYTES of logging to STDOUT/STDERR. No big deal.

I actually loaded an ELF binary of my 3.10 kernel build + an initrd in case things went to shit onto a small FAT boot partition on compact flash this time, rather than TFTPing, since these images are fairly large. I also placed an ext4 partition with a decently populated root environment on the CF card as well. Related to that ext4 partition - I had a few problems creating that as well. I wanted to have the smallest possible ext4 partition so if didn't have to copy much data over network/IDE to CF card/whatever, and wanted it to be later expandable, so I really pushed up the inode to block ratio in order to have a working filesystem that could support many files when it was later expanded. It didn't appreciate my initial settings (`genext2fs: too much overhead, try fewer inodes or more blocks.`) but I compromised a little and it gave in to what I wanted.


``` text Kernel Log Snippet
[  356.227919] Data bus error, epc == ffffffff8134a948, ra =
= ffffffff8134a854
[  356.234721] Oops[#1]:
[  356.236986] CPU: 0 PID: 1 Comm: swapper/0 Not tainted 3.10.0 #2
[  356.242891] task: a80000003b840b38 ti: a80000003b844000 task.ti: a80000003b844000
[  356.250358] $ 0   : 0000000000000000 0000000010108ce1 0000000000000000 00011b00f0081000
[  356.258353] $ 4   : ffffffff814a0000 0000000000000000 0000000000000004 0000000000000002
[  356.266347] $ 8   : a80000003b847ca0 0000000000000001 7669636520303030 303a30313a30392e
[  356.274341] $12   : 0000000000000000 ffffffff8be48698 0000000000001000 0000000000000000
[  356.282335] $16   : a80000003b8b6000 90011b00f0081000 000000000000001e 0000000000002edf
[  356.290330] $20   : ffffffff8be40000 ffffffffffffffff 0000000000000040 ffffffff8be40000
[  356.298324] $24   : 0000000000000001 0000000000000000                                  
[  356.306318] $28   : a80000003b844000 a80000003b847d10 ffffffff814ec4e0 ffffffff8134a854
[  356.314313] Hi    : 0000000000000000
[  356.317875] Lo    : 0000000000000000
[  356.321458] epc   : ffffffff8134a948 quirk_usb_early_handoff+0x22c/0x730
[  356.328129]     Not tainted
[  356.330920] ra    : ffffffff8134a854 quirk_usb_early_handoff+0x138/0x730
[  356.337600] Status: 10108ce3\0x09KX SX UX KERNEL EXL IE 
[  356.342553] Cause : 4080801c
[  356.345422] PrId  : 000d0003 (Cavium Octeon)
[  356.349678] Modules linked in:
[  356.352725] Process swapper/0 (pid: 1, threadinfo=a80000003b844000, task=a80000003b840b38, tls=0000000000000000)
[  356.362885] Stack : ffffffff81520000 ffffffff8141e2a0 a80000003b8b6090 ffffffff814c8030
\0x09  ffffffff814c8048 a80000003b8b6000 000000000000ffff ffffffff8be40000
\0x09  ffffffffffffffff 0000000000000040 ffffffff8be40000 ffffffff812e3b60
\0x09  0000000000000010 a80000003b8b6000 0000000000000010 ffffffff8be80000
\0x09  ffffffff81520000 ffffffff814a7b68 0000000000000006 ffffffff81543094
\0x09  0000000000000000 ffffffff8137a410 ffffffff8be40000 ffffffff8154302c
\0x09  0000000000000006 0000000000000000 ffffffff8be40000 ffffffff8110bae0
\0x09  ffffffff8be40000 ffffffff81558e90 ffffffff81568888 0000000000000006
\0x09  ffffffff81558e48 ffffffff8152fb0c ffffffff8be40000 0000000000000000
\0x09  0000000000000000 0000000000000000 0000000000000000 0000000000000000
\0x09  ...
[  356.427883] Call Trace:
[  356.430326] [<ffffffff8134a948>] quirk_usb_early_handoff+0x22c/0x730
[  356.436672] [<ffffffff812e3b60>] pci_fixup_device+0x150/0x1ac
[  356.442409] [<ffffffff81543094>] pci_apply_final_quirks+0x68/0x128
[  356.448576] [<ffffffff8110bae0>] do_one_initcall+0x88/0x120
[  356.454141] [<ffffffff8152fb0c>] kernel_init_freeable+0x140/0x1fc
[  356.460221] [<ffffffff811067c4>] kernel_init+0x14/0x110
[  356.465431] [<ffffffff81100948>] ret_from_kernel_thread+0x10/0x18
[  356.471505] 
[  356.472980] 
Code: 2412001e  ae220008  8e220008 <7c0210a0> 00221402  c8400005  24040001  0c443ee2  2652ffff 
[  356.487055] ---[ end trace 2a21ae29f86033b8 ]---
```

Hahahahaha damn it.
Good news is, though, the PCI bus works perfectly, and it even loaded a network driver for a device! Also the RTC works. So I won't have to live in 1970. I wouldn't like that, they don't have cool MIPS machines yet.

Bad news is, besides the kernel oops from the USB implementation, is that while all 16 CPUs were detected properly (unlike OpenBSD), the heirarchical RCU (currently the default) I configured forced NUM_CPUS back down to 1 from 16. Oh well, easy enough to switch it to classic.

Anyways, we were running Linux on this Cavium Octeon based Lanner MR-950 for an entire 356 milliseconds, so I'd call that a win.
