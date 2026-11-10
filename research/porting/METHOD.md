# Porting a mod to a new Bento firmware version

How we moved Bento Playground's patches from 1.6.1 to 1.6.8 without guessing. The method is
general: it should work for the next official release too. The results for 1.6.8 are in
[../firmware-1.6.8/](../firmware-1.6.8/).

## Principles

1. **Port onto the new, untouched stock image.** Never transplant 1010music's changes into an
   old modded image: you would lose their fixes.
2. **Pin exact image hashes per version.** Unknown or modified input images must fail the build.
3. **Generate, don't hand-list.** Every address the mod touches or uses is found by a tool, so
   nothing is forgotten.
4. **A location is not a behaviour.** "Found the same code" means the address is right. Whether
   the function still means the same thing needs a separate check: callers, struct offsets,
   tests.
5. **Keep the old version's build byte-identical** while adding the new one.

## Step 1: inventory

List everything the mod depends on in the old image:

- every write against stock (patch sites, code placed in zero areas, appended code);
- every native address the added code uses: calls, jumps, literal words, MOVW/MOVT pairs,
  pointer-looking data words (found by a Capstone sweep over all added bytes);
- every address-like hex constant in the mod's source files (file:line).

For Playground on 1.6.1 that was 194 patch sites, 29 zero-area blocks, 1 appended block and
732 native flash addresses.

## Step 2: match the two stock images (`tools/fwmatch.py`)

1. Normalise both images: mask BL/BLX/B.W branch immediates, MOVW/MOVT immediates and every
   aligned word that looks like a flash/RAM/SDRAM pointer.
2. Take 24-byte windows that are unique in **both** images as anchors and grow them into
   maximal equal blocks (old start, old end, shift).
3. An address maps through the block that contains it. The result says whether the whole
   containing function is identical apart from relocations (high confidence) or only part of it.
4. Pointer words and branch targets inside matched blocks give independent old → new evidence
   for data, RAM and function addresses.
5. RAM addresses map through native literal-pool evidence.

For 1.6.1 → 1.6.8: 2,120 blocks covering about 96 % of the old image.

## Step 3: find patch sites by signature (`tools/hooksig.py`)

For each patch site, `hooksig.py build` stores a context signature: the bytes around the site
with relocation fields wildcarded. `hooksig.py find` then looks for each signature in the new
image, trying these tiers in order (the first tier with any candidate decides):

| Tier | Meaning |
|---|---|
| exact | context and site bytes match |
| context | context matches, the site bytes changed |
| pre / post | only the context before, or only after, matches |
| pointer | a patched table slot: the functions in the neighbouring slots are found first, then the table by its relocated pointer run |
| skeleton | the instruction sequence matches with all immediates ignored |

Anything found by a tier other than `exact` needs a person to look at it.

**Careful:** the sites file written by `build` contains stock byte excerpts (that is what a
signature is). Keep it on your own machine and don't publish it. The reports `find` writes
contain addresses only.

## Step 4: check meaning, not just location

Locations were cross-checked in several independent ways:

- **Callers and pointer literals**: for every mapped address, what native callers and pointer
  literals in both images say.
- **Class layouts**: field offsets of the native classes whose vtables are patched, read from
  their methods in both images.
- **Struct offsets**: every non-address constant ≥ 0x80 in the mod sources (struct offsets, ids)
  checked against the native instructions that use it in both versions.
- **Parameter keys**: aligned by the order of native registry registrations (name + type).
- **UI ids**: menu tables matched by label.
- **Model event codes**: the dispatcher's jump tables compared case by case.
- **Second matcher**: instruction-shape fingerprints plus vtable slot agreement (the approach of
  romanjeff's `bento_enhanced` Ghidra tools, re-implemented without Ghidra). It agreed with the
  byte-block matcher on every function entry both could see.

Version-specific constants (keys, ids, offsets) cannot be translated by search-and-replace: the
same number means a key in one line and a colour or size in another. We put them behind named
definitions per firmware version instead.

## Step 5: space

New firmware is usually bigger. Measure the free space in the new image before porting:
1.6.8 grew by 12 KB, which shifted the end of the M7 code and of the whole image. Code placed in
zero runs right after the old image end has to move.

## Pitfall: Keystone mis-encodes conditional Thumb-2 branches

If you assemble patches with **Keystone**: conditional Thumb-2 branches (encoding T3) to
**absolute targets** come out wrong. Example: `bne.w 0x08062080` assembled at 0x08062466 jumps to
0x080C44EA. We found two such branches in our own builds. They had been there for weeks; whether
they ever caused a problem on the device is unknown.

Fix: branch to a local label and jump from there (`bls <label>; …; label: b.w <target>`), and
check every Keystone-assembled branch after assembly. Disassembling the result and comparing the
target is enough.

## Tools in this folder

| File | What it does |
|---|---|
| [tools/fwmatch.py](tools/fwmatch.py) | Relocation-masked matcher of two main images. Library: `Matcher(old_bytes, new_bytes).map(addr)` |
| [tools/hooksig.py](tools/hooksig.py) | `build`: signatures of all patch sites of a build; `find`: locate them in another stock image |
| [tools/test_hooksig.py](tools/test_hooksig.py) | Self-test on a synthetic image, no firmware needed: `python3 tools/test_hooksig.py` |
| [tools/hook_features.json](tools/hook_features.json) | Example rules that group patch sites by feature |

Requirements: Python 3 and `capstone` (`pip install capstone`). The tools only read images;
nothing here writes firmware. You need your own copy of the official firmware.

`hooksig.py build` expects a `FULL_MANIFEST.json` describing your build:

```json
{
  "stock_sha256": "<sha256 of the stock image>",
  "writes": [
    {"address": 134479030, "size": 4, "original_hex": "…", "replacement_hex": "…", "reason": "what this patch is for"}
  ]
}
```
