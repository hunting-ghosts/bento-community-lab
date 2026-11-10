# What has already been established — 13 October 2026 checkpoint

> **Historical document (mid-October 2026).** Written at the start of the project, before any
> patch ran on hardware. Kept as it was, apart from small wording edits for publication.
> Several readings here were corrected later; the current state is in
> [../../firmware-1.6.1/](../../firmware-1.6.1/).

Summary of the first Ghidra research notebook and its full-survey addendum (not published).

## Exact firmware files

| File | Bytes | SHA256 |
|---|---:|---|
| `1.6.1/bento1.bin` | 1,523,516 | `14475e9d5068f5659d0fe0b00fc1278c1479c43ed9fda3d40654d5f3a7a6bbb3` |
| `1.6.1/bento2.bin` | 85,696 | `8100906327093463fb58f531ee28583423a9881ea3a7d5b77874a7700735e99d` |

The main `bento1` raw image was imported in Ghidra as little-endian `ARM:LE:32:Cortex` at `0x08040000`; Ghidra's analyzer made a single RWX block `0x08040000–0x081B3F3B`. That does **not** confirm the physical system memory map. Three plausible ARM vector tables appear at offsets `0x0`, `0xA0000` and `0xC0000`, i.e. `0x08040000`, `0x080E0000`, `0x08100000`. Candidate reset addresses (Thumb bit cleared): `0x080403CC`, `0x080E03C2`, `0x081003CC`. Ghidra's previous automatic analysis did NOT recognize those entries as functions. The smaller `bento2` has candidate base `0x90000000`, reset word `0x900003C3`, and the label `BENTOCONTROLLER 1.0.15-QSPI`. Processor/core allocations remain **unverified**.

## Survey scope

Earlier full survey reports **3,728 candidate functions** (543 region 1, 408 region 2, 2,777 region 3). 43 target decompilations attempted; `FUN_080683E0` timed out, `FUN_081639A8` decompilation truncated. Function boundaries, arguments and prototypes can be wrong: inspect ARM instructions to confirm.

## Important addresses and evidence classification

| Address | Current reading | Confidence, scope |
|---|---|---|
| `0x081639A8` | Registers many parameter descriptors via `FUN_08163952`, `FUN_081638CC`, `FUN_0816397E` | Strong inference from sampled calls, not complete type model |
| `0x0816FC52` | Walks linked listeners via field `+0x18`, performs virtual call at vtable `+0x34` | **CORRECTION:** event/notification dispatch, NOT proven menu append |
| `0x081605B0` | Composes messages with codes `0x1001`, `0x1002`, `0x1003`, `0x1004`, `0x1006`; dispatches via `0x0816FC52` | Context action logic likely; exact message meanings still uncertain |
| `0x0814B630` | Reads property `0x131`, caches at `+0x75B8` and updates eight widgets at `0x750` stride | **Selection display**, NOT yet selected-track setter |
| `0x081736B4` | Sets byte `+0x3DD` and dirty flag `+0x42C`, calls UI refresh | Strong widget state update |
| `0x08133614` | Gets model property, chooses local/fallback retrieval | Getter, not setter |
| `0x081372EC` | Detects sequence-layer length mismatch and triggers retry when short | Strong; UI↔engine protocol not fully mapped |
| `0x08136928` | Called from mismatch path; potential resend initiation | Provisional |
| `0x0810389C` | RIFF/SoundFont2 (`sfbk`) parser reading `sdta`, `pdta`, instruments and sample zones | Strong evidence for parsing logic, **not** a user-available SF2 synth |
| `0x080A1F00` | Decodes MIDI status/data bytes | Strong sampled evidence |
| `0x080EFBD4` | Exception/privilege/RTOS-like path | Still unclear role |

## Useful but unconfirmed symbol clues

`evtcond` exists and has parameter registry xrefs, but it was also present in firmware 1.3 and 1.5.32; its existence alone DOES NOT mean conditional triggers are implemented. Same for `fx3send`/`fx4send`, which do NOT establish 4 active FX send buses. `padbrightness`, `duckattack`, `padmute` appear in newer firmware; their actual feature availability is confirmed separately in 1.6 release notes, but exact code path still needs tracing. `SessionMgr ... UI ... engine ... still short after resend` has an actual executable handler, supporting an explicit sequence-length synchronization/retry mechanism.

## Why older interpretations changed

Early targeted Ghidra report described `FUN_081605B0` as a menu builder and `FUN_0816FC52` as the method adding menu entries. The **later full Ghidra survey** found linked listener iteration and virtual dispatch in `0x0816FC52`: do not use the earlier description without reevaluation. Earlier UI function `0x0814B630` seemed related to selection, but is a UI selection **refresh**, not the function that changes track selection. This significantly affects the next work.

## Not achieved yet

- No independently verified pad/button input dispatcher, selected-track state setter or TRACKS+pad hook.
- No patched image, emulated actual Bento firmware, flashed custom firmware or tested recovery pathway.
- No mapped data protocol between controller vs UI vs sequencer/audio cores.
- No validated memory headroom, CPU headroom, channel-strip insertion point or ported synth engine.

## Other source files

`Bento_16_Track_Trace.java`, `Bento_16_Full_Survey.java`, `Bento_16_Deep_Dive.java`, `Bento_16_Ghidra_Report.java`; earlier kits with `analyze_bento_1_6.py`, address map JSON, and pre-1.6 comparator. `Original_Ghidra_Error_Log.txt` contains unrelated Ghidra Java/extension compilation errors seen during setup and is troubleshooting evidence, not firmware data.
