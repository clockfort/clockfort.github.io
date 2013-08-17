---
layout: post
title: "Picky Software to go with Picky Hardware"
date: 2013-08-15 17:46
comments: true
categories: [OpenBSD, hacking, hardware]
---

Good news! Grabbed a snapshot kernel build from OpenBSD's trunk (currently only the development version, 5.4, has the beginnings of support for any Cavium SoCs at all) and managed to TFTPboot into that kernel and a small ramdisk.
I did have some problems; everyone on the internet seems to say that telling u-boot's TFTP client to download to memory address 0 is a sort of auto-find-an-address-that-won't-stomp-on-uboot's-own-memory, but this was not the case for me. Manually inserting an ELF binary (in this case, kernel + small ramdisk) at 0x21000000 and booting it with the ELF-binary loader (`bootoctlinux`) did work though. (I got the 0x21000000 address from whoever owned this board last, as there were some macros left in the preboot environment using that address)

... that's where the fun stops though.

``` text
octcf0 at iobus0 base 0x1d000800 irq 0: Doesn't support 8bit card
: card not present
```

(from OpenBSD/src/sys/arch/octeon/dev/octcf.c:)
``` c OpenBSD/src/sys/arch/octeon/dev/octcf.c

for (i = 0; i < 8; i++) {
	uint64_t cfg = 
	*(uint64_t *)PHYS_TO_XKPHYS(
		OCTEON_MIO_BOOT_BASE + MIO_BOOT_REG_CFG(i), CCA_NC);

	if ((cfg & BOOT_CFG_BASE_MASK) ==
		(OCTEON_CF_BASE >> BOOT_CFG_BASE_SHIFT)) {
		if ((cfg & BOOT_CFG_WIDTH_MASK) == 0)
			printf(": Doesn't support 8bit card\n",
				wd->sc_dev.dv_xname);
		break;
	}
}
```

ARGH. The hardware can do it, but OpenBSD is so damn picky about what it will take. USB didn't come up on boot either, and neither did the SATA controller, and neither did the network, so my options of what to boot from are basically using u-boot to manually load ramdisks into ram and booting them... or nothing.

Also bad:
``` text
cpu0 at mainbus0: Unknown CPU type (0x0) rev 0.3 499 MHz, Software FP emulation
vendor "Cavium", unknown product 0x0005 (class processor subclass MIPS, rev 0x03) at pci0 dev 0 function 0 not configured
unsupported octeon model: 0xd0003
```

The OpenBSD kernel doesn't know what this processor is :-(  Also as a result it's using only two cores, and is software-emulating floating point instructions, since it doesn't know how to interact with the hardware.


Not the worst, mostly just annoying:
``` text

0:0:0: mem address conflict 0x11000000/0x1000

0:0:0: mem address conflict 0x18000000/0x8000000

0:5:0: bridge mem address conflict 0x21000000/0x300000

0:5:0: bridge mem address conflict 0x10000000/0x100000

WARNING: No TOD clock, believing file system.
```

