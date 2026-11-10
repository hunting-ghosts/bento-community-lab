# Bento 1.6.8 compared with 1.6.1

What changed between the official 1.6.1 and 1.6.8 firmware, from the point of view of someone
who patches the firmware. Collected while porting Bento Playground (November 2026).

**Evidence: STATIC and EMU only.** Everything here comes from comparing the two stock images
and from emulator tests. Nothing on this page has been confirmed on hardware yet. Where a
finding came from tests on our own code, that is said.

## Images

| | 1.6.1 | 1.6.8 |
|---|---|---|
| Main image `bento1.bin` | 1,523,516 B, SHA-256 `14475e9d…a7a6bbb3` | 1,535,540 B, SHA-256 `cb8f830ec3ab45a92d120ae9688e380573b24ad88710a3ad3d21af6ffb3c8d0d` |
| Image end | 0x081B3F3C | 0x081B6E34 |
| End of M7 code/data | 0x080CF499 | 0x080CFC59 |
| Controller image `bento2.bin` | | **byte-identical to 1.6.1** |

1.6.8 is 12,024 bytes larger: M7 +1,984, M4 +10,040.

## How much stayed the same

After masking relocations (branch targets, MOVW/MOVT immediates, pointer words), 24-byte windows
that are unique in both images anchor **2,120 identical blocks covering about 96 % of 1.6.1**.
Most of the code is therefore the same, only at different addresses. See [../porting/METHOD.md](../porting/METHOD.md) for how
this was measured.

Two independent methods agree on the function mapping:

- our relocation-masked byte-block matcher (`fwmatch.py`);
- an instruction-shape fingerprint matcher plus vtable slot correspondence, re-implemented from
  the approach in [romanjeff/bento_enhanced](https://github.com/romanjeff/bento_enhanced)
  (`port_anchors.py`, `vtable_compare.py`).

On the 242 function entries both can see: 242 agree, 0 disagree. On 31 patched vtable slots:
28 agree, 0 disagree, 3 tables only one method could match.

## Free space in 1.6.8

| Area | From | To | Bytes |
|---|---|---|---|
| Bank 1 (M7 side), after the M7 image | 0x080CFC70 | 0x080E0000 | 66,448 (not one block, see below) |
| Bank 2, after the image | 0x081B6E50 | 0x081E0000 | 168,368 |

- The rules from 1.6.1 still apply: 0x080E0000..0x080FFFFF is the separate SD-card mode program,
  and the application must end before 0x081E0000.
- Everything in 0x080D0000..0x080E0000 that was zero in 1.6.1 is still zero in 1.6.8. The only
  new pointer into bank 1's free area is again the zero-length data-section marker (now
  0x080CFC5C, was 0x080CF49C).
- Code that used the zero run just after the 1.6.1 M7 end (0x080CF4A0..) has to move: 1.6.8's
  M7 code now ends at 0x080CFC59, leaving 912 bytes before 0x080D0000.

## RAM

- **M7 RAM layout unchanged**: startup copy/clear tables have identical sizes, heap literal sites
  are identical.
- **M4 .bss grew by 0x19B8.** M4 globals above about 0x3000EC70 moved in steps
  (+0xC … +0x19B8). M4 initial SP 0x30040000 → 0x3003C000.
- In the region 0x30034D48..0x30035F40 (root header object and neighbours) one object grew by
  0x18; several addresses there are not resolved yet.

## Parameter keys renumbered

Mapped by aligning the native parameter registry (271 registrations in 1.6.1, 270 in 1.6.8,
by name and type):

| 1.6.1 key | 1.6.8 key |
|---|---|
| < 0x139 | unchanged |
| 0x139, 0x13A, 0x13B (In 1 / In 2 / In 3) | merged into one "Input Gain" at 0x139 |
| 0x13C..0x1D8 | key − 2 |
| ≥ 0x1D9 | key − 1 |
| (new) | 0x1D7 "LastDate" |

Example: Mixer Track keys 0x13F / 0x140 / 0x141 become 0x13D / 0x13E / 0x13F; 0x134 stays.

1.6.8 registers 0x19A, 0x1D2 and 0x1D3 natively. **If your mod registered private keys in the
free gaps of 1.6.1, check them against 1.6.8**: shifted native keys can land on them.
Registrations without a constant key were not decoded, so free-looking keys are not proven free.

## UI / action ids renumbered

Menu tables matched by label (46 ids), and the code agrees:

| 1.6.1 id | 1.6.8 id |
|---|---|
| ≤ 0x1011 | unchanged |
| 0x1013..0x103B | + 2 |
| ≥ 0x103C | + 3 |

New native menu entries: "Cut Group" and "Pack". The highest native menu id went from 0x1047 to
0x104A. Example: the SD-card burger key 0x101D became 0x101F.

## Model event codes

Jump tables of the model event dispatcher (0x081403EC in 1.6.1, 0x08141EA4 in 1.6.8) compared
case by case: codes up to 0x100 keep their handlers; 0x101..0x106 became 0x102..0x107; 0x10C
became 0x10D.

## Struct layouts that changed

- `MODEL` Undo buffer +0xB26C → +0xC3C0.
- `MODEL` pad lights +0x19C4 → +0x19D0.
- An object offset 0x36C4 → 0x382C. A root "armed" field 0x117C → 0x1194.
- Pad module / voice fields +0x2C8.. → +0x308.. (+0x40). Of the M7 functions that Playground
  patches or calls, only these three pad module functions changed their offsets.
- Header record field +0x6B → +0x83 (seen in the header apply function).
- The wave pane class (vtable 0x081B3A18 in 1.6.1) changed layout.
- No changed fields in any method of: mixer strip (0x081B2A4C), step cell (0x0819E814), track
  column (0x081B38D0), piano grid (0x081B2BCC).

The M7 mixer, routing, sampler and automation offsets listed in
[../firmware-1.6.1/NATIVE_INTERNALS.md](../firmware-1.6.1/NATIVE_INTERNALS.md) are **unchanged**:
the same instructions exist in both images (sampler code 0x0808E000..0x08093000 is
instruction-identical at +0xB00).

## Functions that changed in ways that matter for hooks

- **Outer button consumer** 0x0813F98C → 0x08141408: the switch now loads the button code
  from memory (event read from [r6]). Everything that hooks button handling needs a new adapter.
- **Undo apply** 0x0813B21C (hook site 0x0813B290 → 0x0813CA4E): new register use, stack frame
  and Undo buffer offset. Same stack slots (+0xC, +0x10, +0x114) at that site. The clip snapshot
  0x081338DC only matches with short context.
- **Undo during recording**: 1.6.1 refuses Undo while a take is being recorded; 1.6.8 discards
  the take through a new function (0x08138548) and returns before the clone-back. (EMU)
- **Pad release after the keyboard octave changed**: 1.6.8 posts the pitch stored at the press;
  1.6.1 posts the new pitch. (EMU)
- **Chord / pad module**: the call site 0x0808414A no longer exists in the same form. Pad post
  sites 0x08084918 / 0x0808494E / 0x0808414A / 0x08084180 → 0x0808537E / 0x080853B4 /
  0x08084BC6 / 0x08084BFC. The posted record layout (type, 0, handle, pitch, velocity) is
  unchanged.
- **Mod Matrix rows** 0x0814CE80 → 0x0814EAEA; synth-page key code around 0x0815F5AE /
  0x0815F68E changed; Mod-mode view vtable slots 0x0819E894 / 0x0819E8C4.
- **Header actions** 0x08134800 → 0x08135D74 (new prologue). **Header apply** 0x0817FAF8 →
  0x08181C7C.
- **Burger menu tables** (Undo / Select / Length): 0x08198DB0 → 0x0819B8E8, 0x08198770 →
  0x0819B2A8, 0x081981F8 → 0x0819AD30. Same structure, ids renumbered.

## New native behaviour that can collide with mods

From our reading of the code and the changed functions, not from official release notes:

- The Launch screen lights pads for playing clips.
- The bank screen's Left/Right now step through all 16 pads.
- Mod Seq Slide (knob 7 in Granular/Wavetable Mod mode).
- Undo during recording (above).

## Address map

The full table of 196 patch sites, with their 1.6.8 location, match confidence and whether the
original bytes are still identical, is in [HOOK_MAP.md](HOOK_MAP.md). A selection of commonly
useful native addresses:

| What | 1.6.1 | 1.6.8 |
|---|---|---|
| Native note entry (pitch) | 0x0805F8E0 | 0x0805FC80 |
| Note-off | 0x0805FE00 | 0x080601A0 |
| Renderer call site in the block function | 0x080601E8 | 0x08060588 |
| Sequencer renderer | 0x08060820 | 0x08060C40 |
| Renderer note-on site | 0x08060B9A | 0x08060FBA |
| Router event post | 0x08065600 | 0x08065B20 |
| Track render entry | 0x0807A4AC | 0x0807AC6C |
| Modulation route command | 0x0807C120 | 0x0807C8A0 |
| Track output buffer | 0x0809F0C0 | 0x0809FCC0 |
| Mixer loop | 0x080A0522 | 0x080A1142 |
| Outer button consumer | 0x0813F98C | 0x08141408 |
| Model event dispatcher | 0x081403EC | 0x08141EA4 |
