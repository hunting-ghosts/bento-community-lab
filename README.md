# Bento Community Lab

Firmware research for the **1010music Bento**, from the work on
[Bento Playground](https://github.com/hunting-ghosts/bento-playground). For understanding or
modifying the Bento firmware. If you want to *use* the mods, go to Bento Playground.

## Research

**[research/](research/)** collects what we know about the firmware, written for other modders:

- **1.6.1:** memory map, areas you must never write, memory protection, the stock crash record,
  SD-card access, and how the model, Undo, UI, sequencer, sampler and mixer work inside.
- **1.6.8:** what changed compared with 1.6.1 (free space, RAM, renumbered keys and ids, changed
  structures and functions) and where every patch site moved.
- **Porting:** a method for moving a mod to a new firmware version without guessing, with
  read-only tools.
- Ghidra scripts and a display emulator.

Every finding says how it is known: seen on hardware, shown in an emulator, or read from the
code. No 1010music firmware is included; you need your own copy of the official update files.

Findings and corrections are welcome as pull requests.

## Also worth a look: bento_enhanced

**[romanjeff/bento_enhanced](https://github.com/romanjeff/bento_enhanced)** is a Bento
firmware project well worth reading alongside this repository. It has things nobody else has:

- **Aftertouch sequencing**: per-note pad pressure recorded in the piano roll and played back,
  on External tracks (poly aftertouch or channel pressure) and on Bento's own instruments, plus
  live poly aftertouch out of External tracks.
- **Its own firmware documentation**: the two-chip hardware layout and the finding that the UI
  runs on the M4 core, the UI framework (scene graph, property systems, encoder behaviour), the
  modulation source table, External tracks and MIDI out, project settings, the master compressor.
- **Ghidra tooling** for the 1.6.1 image, including the anchor and vtable comparison approach
  that our own 1.6.1 → 1.6.8 cross-check re-implements.


## Code snapshot (October 2026)

The source code in this repository is a **frozen snapshot** of the early mod work (P27,
26 October 2026, for stock firmware 1.6.1). It is not maintained here and is not a release.
Current features and installable versions are in
[Bento Playground](https://github.com/hunting-ghosts/bento-playground).

The snapshot stays because other projects build on it.
[romanjeff/bento_enhanced](https://github.com/romanjeff/bento_enhanced) ports several of its
features (trigger conditions, inspect-a-step, hold shortcuts, step-lock UI), and its firmware
notes build partly on the design records in `docs/releases/`.

- Fixed reference: tag `snapshot-2026-10`.
- These paths will not move: `k16_work/`, `eu4_work/`, `chord_work/`, `research/ghidra_scripts/`,
  `docs/releases/`.
- An earlier snapshot (CH10) is at commit
  [`302ebf4`](https://github.com/hunting-ghosts/bento-community-lab/tree/302ebf4a7e63f857c509705933f81e000dbd8a6d).
- What the snapshot contains and how far it worked at the time:
  [features and status](docs/FEATURES_AND_STATUS.md), [known issues](docs/KNOWN_ISSUES.md),
  [build instructions](docs/QUICKSTART.md), [developer guide](docs/DEVELOPER_GUIDE.md).
  These documents describe the snapshot as of October 2026.
- The [design preview](design/bento-playground/README.md) is an early visual study, not the
  current design.

## Disclaimer

Unofficial; not affiliated with or endorsed by 1010music. Firmware modifications are at your own
risk and can cause instability or data loss. Never modify or overwrite the bootloader. Keep
backups of the original firmware and your projects, and know the device's recovery process
before flashing anything. Nothing in this repository flashes a device.

Much of this project was developed with AI coding assistants and tested step by step on hardware.

## Related projects

- [hunting-ghosts/bento-playground](https://github.com/hunting-ghosts/bento-playground): the
  current mod, for musicians
- [romanjeff/bento_enhanced](https://github.com/romanjeff/bento_enhanced): aftertouch sequencing,
  sequencer extras and its own firmware documentation
- Blackbox mods: [viktorakos95/blackbox-mod](https://github.com/viktorakos95/blackbox-mod),
  [pocketrave/blackbox](https://github.com/pocketrave/blackbox),
  [Blindsmyth/custom-blackbox-fw](https://github.com/Blindsmyth/custom-blackbox-fw)

## License

GPL-3.0-or-later. Bundled third-party code keeps its original MIT / LGPL-2.1 licences; see
[THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md).
