# Lessons from two Blackbox Mk1 firmware mods

> **Historical document (mid-October 2026).** Written at the start of the project, before any
> patch ran on hardware. Kept as it was, apart from small wording edits for publication.
> Several readings here were corrected later; the current state is in
> [../../firmware-1.6.1/](../../firmware-1.6.1/).

A **navigation aid** for the two projects' source, not a substitute for reading it. These mods are for the **original Blackbox Mk1 stock 3.1.9** and **not binary-compatible with Bento 1.6.1**.

## PocketRave blackbox (`blackbox-main`)

**Architecture:** Python emits ARM Thumb-2 instructions via a custom encoder (`mod/thumb.py`). Hooks are defined against expected original bytes (`mod/bbpatch.py`); patching should stop on wrong input, code caves append after stock image. `apply.py` and `docs/patcher.js` implement CLI/browser patching. Tests reside under `tests/`.

**Representative references:**
- `mod/trig.py`: per-note condition logic (`1:2..8:8`, first/not-first, previous/not-previous, fill/not-fill), `BACK` press/long/release handling and momentary FILL, MIDI fill controls; useful pattern for modifier/trigger work.
- `mod/steplen.py`, `mod/colx.py`, `mod/launch.py`: per-clip sequence lengths, performance launching, polyrhythm.
- `mod/modcc.py`, `mod/fxpage.py`: MIDI CC mappings of sends, reuse of existing modulation/UI facilities; mappings saved with presets.
- `mod/touch.py`, `mod/menu.py`, `mod/seqscroll.py`: UI navigation and gesture/control hooks.
- `mod/bbpatch.py`, `mod/build.py`, `tools/patchlib.py`: patching/validation and versioned distribution.

## blackbox-mod (`blackbox-mod-main`)

**Architecture:** Compiled C/ARM assembly (`src/`, `build.sh`), Python patch descriptions in `patches/`, multiple targeted branch hooks, appends code cave, and in a newer release experiments with separate flash region `BBEXTRA.BIN`. Source README cautions that JOEBOX 1 is **on hold as of 2026-10-14 due to restart loops on some units**; don't present all documented features as stable.

**Representative references:**
- `src/insert.c`, `src/insert_dsp.c`, `src/insert_thunk.S`, `patches/` insert hooks: sound-path interception at a **per-pad** mix stage before send/output routing; adds phaser, flanger and gate. Bento needs its own **per-track** mixing path discovery, no direct reuse of Blackbox offsets.
- `src/cond.c`, `patches/cond.py`, `test_cond.py`: note playback conditions and logic tests.
- `src/chord.c`, `patches/chord.py`, `test_chord.py`: chord shapes linked to scale/root, record chords as note events.
- `src/solo.c`, `src/slice.c`, `src/duck.c`, `src/lfo.c`, `src/master.c`, `src/comp.c`, `src/clip.c`, `src/cpu.c`: control/UI/audio tweaks including extra modulation, FX, dynamic processing, CPU load meter.
- `emu.py`, `test_*.py`: useful examples of running firmware paths or logic in automated testing.
- `apply.py`, `patch.py`, `make_patch.py`, `docs/`: firmware fingerprint checks and reproducible patch delivery.

## Transfer relevance, carefully qualified

| Desired Bento capability | Proven by Blackbox source? | Unsolved Bento work |
|---|---|---|
| TRACKS+PAD, PROJ as SHIFT | Precedent: button event hooks, press/hold/release handling | Identify actual Bento input dispatcher and track-selection **setter**; preserve original behavior |
| Visual mapping editor | Browser patchers demonstrate client-side editing/patch distribution; source has limited parameter mapping | Build Bento runtime action table/parser; prove appropriate hooks and configuration file loading |
| Conditional triggers | Both mods implement Blackbox-specific note conditions using existing note probability field | Inspect Bento event format, probability code, range checks, save/load behavior, possible `evtcond` semantics |
| Independent clip length/polymeter | PocketRave has per-clip length controls | Trace Bento sequence launch/length synchronization and project serialization |
| Per-track EQ/FX | blackbox-mod inserts **per-pad** DSP on Mk1 | Locate Bento per-track pre-send/post-voice mixed stream; profile CPU and buffers |
| Chord mode | blackbox-mod integrates chord shapes into Keys screen | Bento UI integration and note/pad mapping; source algorithms may inspire design |
| Extra synth engines | Neither repository demonstrates native Bento-compatible synthesis engine insertion | Major new MIDI/voice/DSP/UI/preset/lifecycle engineering; defer |

## Concrete coding practices worth copying

1. **Byte fingerprinting + precondition asserts** for exact firmware before touching bytes. Never apply hooks on the basis of version text alone.
2. **Minimal, reversible, independent hooks**, preserving untouched execution paths.
3. **Automated tests/emulation** for conditions and event state machines, separating algorithm correctness from hardware compatibility.
4. **Documented source of original hook bytes and control flow** and an exact patch-build receipt.
5. **Instrument CPU usage and crash reports** before introducing DSP load; retain manufacturer's firmware image and verify actual recovery first.
6. **Treat code cave, alternative flash and update process as chip/firmware-specific**. No copy-paste from Mk1 into Bento.

## Upstream context

Blackbox Mk1 mods should be studied for *methods*. Bento/Blackbox 2 are reported to share code, but this claim and any available Blackbox 2 source/firmware comparison are not independently contained in these user-uploaded Mk1 archives. Do not treat it as proven by these ZIPs. If a separate Blackbox 2 firmware is lawfully provided later, cross-compare its code structures after verifying architecture/build.
