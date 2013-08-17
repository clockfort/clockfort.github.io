---
layout: post
title: "Hacking a Lanner MR-950"
date: 2013-08-11 11:36
comments: true
categories: [linux,porting,hacking]
---

I recently bought a Lanner MR-950 "Octeon Powered Security Appliance" for super-cheaps off of ebay. It's a 16-core MIPS64 SoC with 10x 1-gig copper ports, DDR2 modules, dual redundant PSUs, etc.  ([Basic Datasheet](http://www.quantum.com.pl/baza/pdf/5f0ca0cc39ca9a492838d80453fc034ba5936ba3.pdf))

After consulting some chipset documentation to see what was supported, I also bought from eBay to populate the board 2x2GB DDR2-667 PC2-5300 ECC Registered DIMMs, and a 16GB compact flash card from someone on ebay kind enough to list read/write IOPS for their products :-)

Some excited pictures from receiving the item!
{% img center //blog.clockfort.com/images/posts/lanner_mr950.jpg Lanner MR-950 on my couch %}
{% img center //blog.clockfort.com/images/posts/lanner_mr950_guts.jpg Lanner MR-950's guts %}


... the dent in the LED panel above isn't an artifact of my wide-angle camera lens, this little guy has seen some action. With that done, now onto making him boot Linux.


I have some friends who work for / own a company that makes a neat custom [NAS solution on top of Cavium Octeon II SoCs](https://www.exablox.com/products/), and I happen to know that their boxes default to 115200bps serial for debugging, rather than the relatively industry-standard 9600/8/n/1. Sure enough, after this informed lucky guess on serial settings, after turning the box on (and finding the switch to turn off an ear-piercing alarm that I somehow triggered by booting the box), text started streaming in over my janky USB→DB9 Serial→Cisco rollover→RJ45→RJ45/DB9 console adapter setup.

``` text
\0x00


U-Boot 1.1.1 (Development build) (Build time: Mar 15 2007
 - 03:44:07)


WARNING:
WARNING: Measured DDR clock mismatch! expected: 266 MHz, measured: 196 MHz, cpu clock: 499 MHz
WARNING: Using measured clock for configuration.
WARNING:

Warning: Clock descriptor tuple not found in eeprom, using defaults


WARNING: memory configured for 196 mhz clock.  
If this is not the actual memory clock
poor performance and memory instability may result.  
The memory speed must be specified in the board eeprom


Warning: Board descriptor tuple not found in eeprom, using defaults
EBT3000 board revision major:1, minor:0, serial #: unknown
OCTEON CN38XX-NSP revision: 3(pass 4), Core clock: 499 MHz, DDR clock: 196 MHz (392 Mhz data rate)

Unsupported SDRAM Width, 4.  Must be 8 or 16.
hanging...
##
# ERROR ### Please RESET the board ###

~\0x00

```

Darn. I unfortunately understand this message, and it means I need to both:

 * Somehow find and buy 240pin DDR2 DIMMs that are registered, ECC, 2GB or less, AND meet the new apparent requirement of also being "low density" (aka having a larger number of low-density chips rather than high-density ones, which will be accessed over a large data width). This will probably be difficult to find.
 * Either buy slower RAM (I don't want to do that!) or figure out how the ram speed is stored/encoded in the EEPROM and change it.

Whelp. At least old DDR2 server ram is very cheap used, albeit it will be very difficult to find the particular combination of features I need (used RAM sellers aren't usually forthcoming about the exact chip configuration of their DIMMs...).

Stay tuned! I'll be back in around a week and a half, hopefully with much better news, after receiving some new RAM.
