---
title: Endianness
---

## Definition
### Order in which bytes within a multibyte value is stored in RAM
### Little Endian
#### Least significant bytes first in the lower memory addresses
#### e.g. x86 processors
### Big Endian
#### Most significant bytes first in the lower memory addresses
#### e.g. Most RISC processors (SPARC, Power, PowerPC, MIPS), many configurable as either big or little
### E.g. for a value of 0x12345678 at address x
#### Little Endian: 0x78 at address x, 0x56 at address x+1, 0x34 at address x+2, 0x12 at address x+3
#### Big Endian: 0x12 at address x, 0x34 at address x+1, 0x56 at address x+2, 0x78 at address x+3
## Intuition
### For little endian architecture, the "least" significant bytes go first and vice versa for big endian architectures
### Endianness is at the byte-level, there will not be any swapping of bit positions within a byte value
## References
### https://en.wikipedia.org/wiki/Endianness
