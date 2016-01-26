LPC13xx Bootloader Reverse Engineering
======================================

This article describes my research into internals of bootloader on NXP's lpc1313 and lpc1343 chips.

What is it?
-----------
Bootloader on lpc13xx chips is located in internal ROM at `0x1fff0000` and is `0x4000` (16kB) in size. It contains code for:
 - Checksum validation, CRP handling and handover to user code.
 - UART bootloader
 - USB bootloader
 - IAP routines

Existing work
-------------
None that I'm aware of. Googling for undocumented peripheral registers,
values, addresses returns no results or something completely unrelated.

My work
-------
I've used simple utilities to dump the FLASH contents to serial, and then transform to binaries objdump could understand.

The attached disassembly is from lpc1313, although it also contains USB code, so I think it's safe to assume the bootloader is shared amongst different chips.

Bootloader is using quite a few undocumented peripheral registers and
currently what stands out the most for me is the secret FLASH described below.

The CRP checks look simple enough and don't seem to contain logic bugs. I do
wonder how SWD is handled (is it disabled soon after boot, which would be bad
from security point of view, or does it have to be enabled?)

Secret FLASH area
-----------------
When disassembling I've noticed some special accesses are taking place.

It appears `0x0000` area can be remapped to some secret flash, if you write
enable bit `0x40` at location `0x4003c000` (undocumented address, although close
to other FLASH settings):
```
unsigned *f = (unsigned*)0x4003c000;
*f |= 0x40;
```

Note that since this code will basically make user FLASH unreachable, it
should be executed from RAM.

This secret FLASH area is 2k long, with most entries being `0xffffffff`. It
contains some settings that are used by the bootloader and has a description
of FLASH sectors. Accessing beyond 2k of this region just starts returning the same contents over and over again.

TODO
----
 - What are differences between lpc131x and lpc134x chips? Is it possible USB is just disabled? Compare bootloader dumps and secret flash dumps.
 - Can secret FLASH be changed?
 - Where is SWD enabled/disabled?
 - Create short code (binary even?) to easily dump contents of different chips
   and start building a database of bootloaders and special FLASH areas.
 - What's ISP's "T" command?


Extra resources:
----------------
 - [objdump with comments](boot_disassembly.txt)
 - [secret FLASH dump with comments](secret_flash_lpc1313.txt)
