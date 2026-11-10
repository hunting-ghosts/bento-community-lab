# Bento 1.6.1: native internals

How the stock firmware's model, Undo, UI and audio engine work, as far as we have read them.
Same image and evidence levels as [MEMORY_MAP.md](MEMORY_MAP.md). Unless marked otherwise,
everything here is **STATIC** (read from the disassembly). Most of it is used by Bento Playground
features that work in the emulator and on hardware, but that does not prove every single detail
on this page. A name in `code` style is our own label, not a symbol from the firmware.

## Model and object tree (M4)

- `MODEL` is the root object; word 0 of `MODEL` is the model tree root.
- Children: 0x0812D78A(root, i, &obj). Child count at root+0x54, tracks at +0x34. A container's
  type count is at +0x30.
- Every object: vtable at +0, **unique handle at +0x10**, from a global counter in the
  constructor 0x0812D8AC. **A clone gets a new handle.**
- vtable+0x14 = recursive find(obj, handle, &out). vtable+0 = type (0x18 = sequence/clip,
  0x2E = track).
- 0x0812D80E(root, handle, &out): direct children only. 0x0812EA72(root, seq, &node, create):
  the automation node of a clip.
- Properties: get 0x08133614(MODEL, &seq, prop, &out), per object 0x0812DC74(obj, prop).
  Property 0x73 = track number / row, 0x62 = clip length.
- Tracks: 0x08133960(MODEL, t, &owner). Clips of a track: 0x0813391A(MODEL, &owner, j, &seq).
- Clip data deep copy 0x0812E270 (switch at 0x0812E382). Type-5 automation node: 0x814 bytes
  (alloc size 0x0812E8C6, copy 0x0812E3B6, zero 0x0812E8E6).

## Undo and clone (M4)

- Sequence Undo buffer at `MODEL`+0xB26C (tree at +8, pool 0x30000); second buffer `MODEL`+0xB1F8.
  **Undo is single-level**: 0x08124A6A clears the buffer.
- Clip snapshot: 0x081338DC(MODEL, &seq) → 0x08124A7E clones it into the Undo tree.
- Undo apply 0x0813B21C: deletes the current clip (0x0812D54E), clones the snapshot back
  (0x08124B76, new handle), sets 0x73, installs it via 0x08136928, which calls 0x081367E8
  (playback rebuild).
- Generic clone 0x0812EA2A(tree, type, parent, source, &out), used by snapshot, restore and clip
  copy/paste. Wrapping it is the way to carry your own per-clip data through copy and Undo.

## UI (M4)

- Header template records at +0x74 (6 × 48 bytes: +0x20 hidden, +0x21 lit, +0x22 tab group,
  +0x2C id). Popup id at +0x194. Root header object 0x30034D48. Root vtable slot 13 (0x0817FFB8)
  handles event 0x68 {id +8, on +0xC}.
- Piano-roll grid, vtable 0x081B2BCC: touch down 0x08174999, move 0x08175F39, up 0x08176A29.
  +0xA50 box mode, +0x9C0 mode (1 tap, 2 note, 5 rectangle), anchor +0xA48/+0xA4C, rectangle
  +0xA34. Box apply 0x08174CE8.
- Pad lights: `MODEL`+0x19C4, 16 bytes per pad, flushed by 0x081257F0. Track colours
  0x08140F50 / 0x08140EF0 / 0x08140FB4, off 0x08140FBC(0).
- Encoder/model event dispatcher 0x081403EC. Outer button consumer 0x0813F98C (all hardware
  buttons pass through here). Header refresh 0x081599BC.
- The piano-roll button handler 0x0815B548 ignores button codes 0xAF/0xB0.
- Probability / condition label table at 0x081B6000: 135 string pointers (Always, 1..99 %, the
  A:B conditions). The native list count is set at 0x08165D40.

## Voice start and envelopes (M7)

Read 8 Nov 2026 while hunting clicks.

- Synth ADSR objects at voice +0x184 (Env1) and +0x280 (Env2). Trigger 0x080874C0: phase (+4) = 1;
  if the attack time is non-zero, the value (+0x28) **and its output smoother (+0x1C)** go to 0
  (attack 0: jumps to 1.0). Reset 0x08087540 (phase 0, value 0, smoother kept). Release 0x08087A80
  (phase 4, time scaled by the current level). Render loop 0x08087580 (smoother target +0x20,
  glide path 0x08087780).
- Melodic voice start 0x0807BFE2..0x0807C09C (second path 0x0807E53E..0x0807E5DC): resets both
  envelopes, the LFOs (0x080880A0 at +0x380 / +0x480), filter states (0x0805B9E0), +0x730
  (0x08088340), +0x770 (0x0807F520) and the oscillators (+0x178, vtable slot 2), then triggers
  both envelopes. **A voice that is still sounding is fully hard-reset here.**
- Sampler per-voice start ramp (voice+0x58): trigger 0x080639A0 (same pattern), reset 0x08063A00.
  Callers include the sampler key path 0x08094BF8 and 0x08093D80.

## Sequencer playback (M7)

Read 8 Nov 2026 while building ratchets.

- Per-sequence block function 0x08060160(s, clock, router, output) loops over segments and calls
  the renderer 0x08060820(s, clock, offset, frames, router) from its only call site 0x080601E8
  (only while s+0x20 is set).
- After all segments it releases every active note whose end falls in the block
  (0x08060490..0x0806050A): 0x080A2220 end → position, note-off 0x0805FE00(s, pitch, frame, pad,
  router), then table compaction 0x08060620.
- Note-on inside the renderer at 0x08060B9A: r0 s, r1 pitch, r2 velocity (event +0x12; 0 means
  100), r3 frame (0x080A2200 of the 64-bit note time at sp+0x20), [sp] pad, [sp+4] router,
  r6 clock. Event copy at sp+0x28: time +0x2C, length +0x30, pitch +0x38, velocity +0x3A,
  lock word +0x3C.
- After the note-on, the active-note end = time-at-frame + scaled length. Step clips use whole
  steps only (a sub-step length gives end = start). Absolute clips use ticks.
- A new note on a key that is still in the table first gets a note-off at the segment start
  (0x08060A4C).
- The native probability byte is event +1; the gate decision happens in the renderer.

## Sampler note-on and voice stealing (M7)

Read 9 Nov 2026.

- One router per audio block: +0 clock object, +4 per-track event queues (0x604 bytes each:
  24-byte events, count at +0x600; post 0x08065600, read 0x080655A0), +8.. a sorted
  {handle, index} map with its count at +0x508.
- Track output buffer = 0x0809F0C0(router, root+0x670) = router+0x5AC+20·i. Voice scratch
  0x0809EE00(router, 2) = router+0x764+40. Temp router+0xE1C.
- Sampler block process 0x0808E9A0(root, router) dispatches events by type (table 0x0808EA04):
  0x12 sequencer note → key note-on 0x08094760; 0x4E pad → 0x08093BA0; 0x13 note-off → 0x08094640.
  Then the render loop 0x0808FC80: every active voice → 0x08091820 → post-processing 0x08092D00
  mixes into the track output; then 0x08093780 removes finished voices.
- Voice list at root+0x20 (count root+0x60). It grows on demand from the global pool
  0x08072D40(root+0x18, handle, &voice), at most 16 voices. Polyphony at root+0xF4 (constructor
  0x08095460, default 4; 1 = mono path 0x08094D00). Ready flag root+0x1D. Voice age:
  counter (root+0x18)+0xC708 → voice+8.
- Allocation: a free voice first. If polyphony is full, 0x08093A60(root, poly−1) releases the
  oldest; in mono, the active voice is released (0x08095178).
- Release = 0x08063960(voice+0x58): ramp state +4 = 4, flag +0x18 = 1, remaining +0x10 =
  total +0x14 = 256 samples (~5 ms). The ramp gain (0x08063A40) ends at about 6 % and the
  voice is then switched off. That last step is a one-sample jump unless the source fades
  itself.
- **A retrigger never continues the same voice** (except when the pool is exhausted and a
  releasing voice is reused). Each voice owns a descriptor object at voice+0x10 (set by the
  constructor 0x08072A00). The pad's sample is loaded into it by 0x080872C0 **after** the
  voice-start code at 0x08094ADE / 0x08093CDA, and 0x08054F60 has already reset it before that.
- A sample's level follows velocity only through the track's native velocity routing
  (voice+0xB4/+0xB8 is a modulation source object; the sampler has no direct velocity read).

## Mixer and modulation routing (M7)

Evidence: the same instructions exist in 1.6.1 and 1.6.8 (see
[../firmware-1.6.8/CHANGES.md](../firmware-1.6.8/CHANGES.md)).

- Track render: bus index at root+0x284, handle at root+0xC.
- Router bus inputs: router+0x5AC+bus·20 (0x0809F0C0).
- Mixer strip: +0x50 and +0x80 send controls, +0x98 duck state (mixer loop around 0x080A0522).
- Modulation route record: +2, +0xC key, +0x10 slot, +0x14 source, +0x16 amount
  (route command 0x0807C120).
- Automation lanes A..D values: root+0x140+8·(source−9) (source 9 = lane A).
- Sequence player: current clip at +0x86C0, clip stride 0x848.
