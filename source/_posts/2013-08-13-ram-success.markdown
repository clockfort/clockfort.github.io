---
layout: post
title: "Ram Success!"
date: 2013-08-13 20:23
comments: true
categories: [linux,porting,hacking]
---

Whoo! Got one of the two hard-to-find rare RAM sticks today (they shipped from different warehouses that each only had one stick left!). Stuck it in DIMM slot 0 (marked by "J1" on the PCB silkscreen) (the SoC apparently will not boot with memory only in DIMM1 slot) and everything worked great! The other 2GB (matched!) stick comes tomorrow.

Sure enough all that I needed was a slightly different chip configuration of RAM; this RAM is 8-bit wide rather than the previous easier to find ram with 4-bit width (higher density chip rank configuration).

``` text
\0x00
\0x00



U-Boot 1.1.1 (Development build) (Build time: Mar 15 2007 - 03:44:07)




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

DRAM:  2048 MB

Flash: 16 MB
Clearing DRAM.....
.
.
. 
done
BIST check passed.
Starting PCI
PCI Status: PCI-X 64-bit
PCI BAR 1: Memory 0x80080000  PCI 0x18000000
Net:   octeth0, octeth1, octeth2, octeth3, octeth4, octeth5, octeth6, octeth7
Bus 0: 
OK
 
  Device 0: Model: STEC M4 CF Firm: E6236-2 Ser#: STM000105607
            Type: Hard Disk
            Capacity: 14840.9 MB = 14.4 GB (30394224 x 512)
Octeon ebt3000# 
help

?       - alias for 'help'
askenv  - get environment variables from stdin
autoscr - run script from memory
base    - print or set address offset
bootelf - Boot from an ELF image in memory
bootoct - Boot from an Octeon Executive ELF image in memory
bootoctelf - Boot a generic ELF image in memory. NOTE: This command does not support
             simple executive applications, use bootoct for those.
bootoctlinux - Boot from a linux ELF image in memory
bootp   - boot image via network using BootP/TFTP protocol
cmp     - memory compare
coninfo - print console devices and informations
cp      - memory copy
crc32   - checksum calculation
dhcp    - invoke DHCP client to obtain IP/boot params
echo    - echo args to console
eeprom  - EEPROM sub-system
erase   - erase FLASH memory
fatinfo - print information about filesystem
fatload - load binary file from a dos filesystem
fatloadalloc - load binary file from a dos filesystem, and allocate
          a named bootmem block for file data
fatls   - list files in a directory (default /)
flinfo  - print FLASH memory information
go      - start application at address 'addr'
gunzip  - Uncompress an in memory gzipped file
help    - print online help
ide     - IDE sub-system
initflash - init flash when boot from Netrom
itest\0x09 - return true/false on integer compare
loadb   - load binary file over serial line (kermit mode)
loop    - infinite loop on address range
md      - memory display
mii     - MII utility commands
mm      - memory modify (auto-incrementing)
mtest   - simple RAM test
mw      - memory write (fill)
namedalloc    - Allocate a named bootmem block
namedfree    - Free a named bootmem block
namedprint    - Print list of named bootmem blocks
nm      - memory modify (constant address)
pci     - list and access PCI Configuraton Space
ping    - send ICMP ECHO_REQUEST to network host
printenv- print environment variables
protect - enable or disable FLASH write protection
rarpboot- boot image via network using RARP/TFTP protocol
read64    - read 64 bit word from 64 bit address
read64b    - read 8 bit word from 64 bit address
read_cmp    - read and compare memory to val
reset   - Perform RESET of the CPU
run     - run commands in an environment variable
saveenv - save environment variables to persistent storage
setenv  - set environment variables
sleep   - delay execution for some time
tftpboot- boot image via network using TFTP protocol
tlv_eeprom  - EEPROM data parsing for ebt3000 board
version - print monitor version
write64    - write 64 bit word to 64 bit address
write64b    - write 8 bit word to 64 bit address
Octeon ebt3000#
printenv

bootdelay=0
baudrate=115200
download_baudrate=115200
bootloader_flash_update_failsafe=protect off 0xbec00000 0xbec3FFFF;erase 0xbec00000 0xbec3FFFF;cp.b 0x100000 0xbec00000 0x40000
bootloader_flash_update=protect off 0xbec40000 0xbec7ffff;erase 0xbec40000 0xbec7ffff;cp.b 0x100000 0xbec40000 0x40000
linux_cf=fatload ide 0 21000000 vmlinux.64;bootoctlinux 21000000
burn_app=erase bf480000 +$(filesize);cp.b 100000 bf480000 $(filesize)
ls=fatls ide 0
bf=bootoct bf480000 forceboot numcores=$(numcores)
nuke_env=protect off BFBFE000 BFBFffff; erase BFBFE000 BFBFffff
autoload=n
octethethact=octeth0
ipaddr=172.21.44.136
turbo=tftpboot 0x21000000 turbo; bootoct 0x21000000 coremask=ffff
serverip=172.21.44.134
ethact=octeth4
numcores=16
stdin=serial
stdout=serial
stderr=serial

Environment size: 775/131068 bytes
Octeon ebt3000# 
```

Bonus! Looks like the firmware shell is working fine, and it detects my compact flash card fine, so that should work well as an eventual boot media. Until then, I'll probably have to TFTP boot from one of the 8 gig network ports available in this pre-boot environment :-)

Next step: the hard parts! Creating a bootable disk image that u-boot can understand, and a usable *nix system root partition to live in.
