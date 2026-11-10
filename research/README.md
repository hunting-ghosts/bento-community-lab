# Bento firmware research

What we know about the 1010music Bento firmware, collected while building
[Bento Playground](https://github.com/hunting-ghosts/bento-playground). Written for other
modders: addresses, structures, limits, methods and tools.

No 1010music firmware is included anywhere in this repository. To use any of this you need your
own copy of the official update files. Every address refers to an exact image, identified by its
SHA-256.

## Evidence levels

Each finding is marked with how it is known:

| Mark | Meaning |
|---|---|
| **HW** | observed on a real Bento |
| **EMU** | shown by running the stock code in an emulator |
| **STATIC** | read from the disassembly (instructions, callers, pointer literals) |

A STATIC finding can still be wrong. If you find a mistake, please open an issue or a pull request.

## Contents

| Folder | What's in it |
|---|---|
| [firmware-1.6.1/MEMORY_MAP.md](firmware-1.6.1/MEMORY_MAP.md) | Flash layout, areas you must never write, the M4 memory protection, RAM, the stock crash record, SD-card access |
| [firmware-1.6.1/NATIVE_INTERNALS.md](firmware-1.6.1/NATIVE_INTERNALS.md) | Model/object tree, Undo, UI objects, voice start and envelopes, sequencer playback, sampler voice handling, mixer and modulation routing |
| [firmware-1.6.8/CHANGES.md](firmware-1.6.8/CHANGES.md) | What changed from 1.6.1 to 1.6.8: free space, RAM, renumbered parameter keys and UI ids, changed structs and functions, an address table |
| [firmware-1.6.8/HOOK_MAP.md](firmware-1.6.8/HOOK_MAP.md) | 196 patch sites with their 1.6.8 location and match quality |
| [porting/METHOD.md](porting/METHOD.md) | How to move a mod to a new firmware version without guessing, plus a Keystone pitfall |
| [porting/tools/](porting/tools/) | Image matcher and patch-site signature finder (Python, read-only) |
| [ghidra_scripts/](ghidra_scripts/) | Read-only Ghidra report scripts for the 1.6.1 image |
| [display_emulator/](display_emulator/) | Unicorn-based emulator that runs the stock UI drawing code and renders screens |
| [GENERATIVE_SEQUENCING_2026-10-21.md](GENERATIVE_SEQUENCING_2026-10-21.md) | Open-source sequencer code worth studying (Euclid, arpeggiator, randomness) and its licences |
| [history/](history/) | Early research notes from October 2026, kept for reference |

## Related work

- [romanjeff/bento_enhanced](https://github.com/romanjeff/bento_enhanced): the other main source
  of Bento firmware knowledge. Read its `docs/` alongside these pages: hardware and the two-chip
  split (`architecture.md`), the UI framework (`ui-architecture.md`), modulation sources
  (`mod-matrix.md`), External tracks and MIDI out, project settings and the master compressor.
  Its Ghidra tools (`tools/ghidra-1.6.1/`) are where the anchor and vtable comparison approach
  used in our [porting cross-check](porting/METHOD.md) comes from.

## Status

Last updated 10 November 2026. These pages are updated together with major Bento Playground
releases, not continuously.
