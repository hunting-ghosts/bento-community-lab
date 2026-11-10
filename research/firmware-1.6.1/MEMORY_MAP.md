# Bento 1.6.1: memory map and hard limits

Main application image `bento1.bin` from the official 1.6.1 update
(1,523,516 bytes, SHA-256 `14475e9d5068f5659d0fe0b00fc1278c1479c43ed9fda3d40654d5f3a7a6bbb3`),
loaded at **0x08040000**. All addresses below are for this exact image.

Evidence levels used on every page of this folder:

- **HW**: observed on a real Bento.
- **EMU**: shown by running the stock code in an emulator.
- **STATIC**: read from the disassembly (instructions, callers, pointer literals).

`bento1.bin` runs on a dual-core STM32H747 (Cortex-M7 + Cortex-M4). The M7 runs audio and the
sequencer engine, the M4 runs the UI; both cores' code lives in this one image. The controller
image `bento2.bin` runs on a separate chip. Hardware details and how they were determined:
[romanjeff/bento_enhanced docs/architecture.md](https://github.com/romanjeff/bento_enhanced/blob/main/docs/architecture.md).

## Flash layout

| Range | What it is | Rule | Evidence |
|---|---|---|---|
| 0x08000000..0x0803FFFF | bootloader | **never write** | safety rule |
| 0x08040000..~0x080CF499 | M7 code and data | stock | STATIC |
| 0x080CF4A0..0x080DFFFF | mostly zero-filled in stock (several separate runs) | usable for added M7 code; check every run is zero and nothing points into it | STATIC, HW |
| 0x080E0000..0x080FFFFF | **separate program: SD-card / disk mode**, with its own vector table (SP 0x24029898, reset 0x080E03C3). The M7 boot starts it when RTC backup register BKP0R = 4 | **never write**: a build that placed code here boot-looped | HW |
| 0x08100000..0x081B3F3B | M4 (UI) code and data, image end | stock | STATIC |
| 0x081B3F3C..0x081DFFFF | free after the image | room for appended code | HW |
| 0x081E0000.. | last flash sector | **the application must end before 0x081E0000**. An image reaching 0x081F8380 failed the update with "couldn't verify" / error 19 | HW |

The zero runs inside 0x080CF4A0..0x080DFFFF are not one block. One pointer into the area exists
in stock: 0x08040568 holds 0x080CF49C, the load address of a zero-length data section in the
startup copy table. Everything else is unreferenced zero fill. (STATIC)

## M4 memory protection (MPU)

Read on 8 Nov 2026 by emulating the M4 MPU setup at 0x08114D74 (HAL `ConfigRegion` at
0x081153F4). `MPU_CTRL = 1`: enabled **without PRIVDEFENA**, so any M4 access outside these
regions faults, read or execute. (EMU)

| Region | Base | Size | Execute |
|---|---|---|---|
| 0 | 0x24040000 | 256 KB | no |
| 1 | **0x08100000** | **1 MB** | **yes** (flash bank 2 only) |
| 2 | 0x30000000 | 512 KB | no |
| 3 | 0xC0000000 | 128 MB | no |
| 4 | 0xC6000000 | 32 MB | no |
| 5 | 0x60000000 | 8 KB | no |
| 6 | 0x40000000 | 512 MB | no |
| 7 | 0x1FF0F000 | 8 KB | no |

Consequence: stock M4 (UI) code can neither run from nor read flash bank 1 (0x08000000..0x080FFFFF).
Widening region 1 to 0x08000000 / 2 MB makes the bank-1 zero runs usable for M4 code. Builds with
that change boot and run normally. (HW)

## RAM

- SDRAM 0xC0000000..: native bump allocator, heap pointer at 0x3000004C, end at 0x30000048,
  allocator function 0x08104C08. (STATIC)
- M7 stack 0x24048DA0..0x2404CDA0 (initial SP 0x2404CDA0). (STATIC)
- Areas cleared on **every** reset (STATIC):
  - M4 boot stub 0x081004B6 clears 0x24041440..0x240416C0 and 0x24041A00..0x24042000;
  - M7 boot 0x080404B6 clears 0x24040000..0x24041440 and 0x240416C0..0x24041A00.
- **0x24048800..0x24048A60 is not cleared by anything** and no stock code addresses it
  (literal and MOVW/MOVT scan): it survives a warm reset. Useful for a crash log. (STATIC; a crash
  log built on it works on hardware: HW)

## Stock crash record (RTC backup registers)

The stock M7 HardFault handler (0x0804AE40 → recorder 0x0804B3C0) writes a crash record into
the RTC backup registers, which survive a reset. RTC base 0x58004000, `BKPn` = 0x58004050 + 4·n. (STATIC)

| BKP | Content |
|---|---|
| 0..3 | boot words (**BKP0R = 4 starts disk mode**): never write |
| 4 | MMFAR / BFAR |
| 5 | CFSR |
| 6..13 | stacked r0 r1 r2 r3 r12 lr pc xpsr |
| 14..21 | r4..r11 |
| 22 | frame pointer |
| 23 | EXC_RETURN |
| 24 | crash count (incremented) |
| 25 | cause (argument r3) |
| 26 | SysTick CVR |
| 27 | core (2 is written here) |
| 31 | stacked pc |

Reading these after a crash gives the faulting pc without a debugger.

## SD card (FatFS)

| Function | Address |
|---|---|
| f_open (FIL is 0x58 bytes; mode 0x32 = FA_OPEN_APPEND \| FA_WRITE) | 0x08120148 |
| f_read | 0x08120464 |
| f_write | 0x08120604 |
| f_close | 0x08120974 |

- Project save 0x0813032C writes through 0x08130958 (call at 0x0813044C).
- Whole-file reader 0x08130A18 (open, read, close), called from 0x08130522, 0x08130774,
  0x08125A14 and 0x08168456.
- **FatFS takes no locks.** Other stock code (0x08103xxx, 0x0811xxxx) opens files too. Card access
  from added code is only safe on the UI task, directly after one of the stock calls above has
  returned. (STATIC; the crash log that does this works on hardware: HW)

## Version string

The native version-string pointer is read at 0x08131054 and 0x08162AD8. Pointing it at your own
string should change the version shown on the device. (STATIC; not yet confirmed on hardware)
