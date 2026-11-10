# Patch sites: 1.6.1 → 1.6.8

Every place where Bento Playground (build R6.18) patches the stock 1.6.1 image, and where that
place is in 1.6.8. Generated automatically by `fwmatch.py` and `hooksig.py`
(see [../porting/](../porting/)); only this header was written by hand. Addresses only, no
firmware bytes.

**Evidence: STATIC.** Not hardware-tested on 1.6.8.

Columns:

- **1.6.1 / size**: patched address and number of bytes in stock 1.6.1.
- **1.6.8**: the same place in 1.6.8.
- **match**: how sure the location is. *high* = the whole containing function is identical
  apart from relocations; *medium / low / fuzzy* = only part of it matched.
- **guard**: *same-bytes* = the original bytes are identical in 1.6.8;
  *relocated-bytes* = identical except for branch or pointer targets;
  *different-bytes / no-site* = the code changed, so the patch needs a hand-written port.
- **status**: *auto* = accepted on evidence; *review* = located, then checked by hand for our
  port; *unresolved* = ported by hand (see [CHANGES.md](CHANGES.md)).
- **purpose**: our own feature label for the patch. It tells you what the site is for, not a
  native function name.

| 1.6.1 | size | 1.6.8 | match | guard | status | purpose |
|---|---|---|---|---|---|---|
| 0x80404b6 | 4 | 080404b6 | medium | same-bytes | review | startup branch |
| 0x8040588 | 4 | 08040588 | medium | same-bytes | review | Reserve 28064 bytes: kick, lock, condition and UI state; stack unchanged |
| 0x80446ba | 3 | 080446ba | high | same-bytes | auto | Playground palette / release identity |
| 0x80446c2 | 4 | 080446c2 | high | same-bytes | auto | Playground palette / release identity |
| 0x80446cc | 4 | 080446cc | high | same-bytes | auto | Playground palette / release identity |
| 0x8057d26 | 4 | 08057d06 | high | relocated-bytes | auto | Arp once per audio block before native graph consumers; chain unchanged graph traversal |
| 0x805f8e0 | 8 | 0805fc80 | high | same-bytes | auto | R6.6 transpose: tr_note_pitch at the native note entry |
| 0x805fe00 | 8 | 080601a0 | high | same-bytes | auto | R6.6 transpose: tr_off_pitch at the native note entry |
| 0x80601e8 | 4 | 08060588 | fuzzy-equal | relocated-bytes | review | R6.13 ratchet hits after each native sequencer segment |
| 0x8060820 | 4 | 08060c40 | high | same-bytes | auto | EU10 native timed-note renderer entry |
| 0x8060b64 | 4 | 08060f84 | high | same-bytes | auto | T2 native conditions |
| 0x8060b9a | 8 | 08060fba | high | relocated-bytes | auto | L3 exact-version native integration; P14 clear melodic pending word after accepted hit |
| 0x8061040 | 4 | 08061460 | high | same-bytes | auto | L3 M7 application hook |
| 0x8061740 | 4 | 08061b60 | high | same-bytes | auto | T2 native conditions |
| 0x8065600 | 4 | 08065b20 | high | same-bytes | auto | L3 exact-version native integration |
| 0x807a7c8 | 4 | 0807af88 | high | same-bytes | auto | P15 begin isolated note sends after note events |
| 0x807a80e | 10 | 0807afce | high | same-bytes | auto | P14 bind per-voice filter evaluation context; P14 leave per-voice evaluation context |
| 0x807aac0 | 4 | 0807b280 | high | relocated-bytes | auto | Native A-D send routes on generic instrument tracks |
| 0x807ab60 | 4 | 0807b320 | high | same-bytes | auto | P15 publish isolated send submixes after all voices |
| 0x807b1a4 | 4 | 0807b964 | high | same-bytes | auto | P14 voice gain before native pan/mix |
| 0x807c0f6 | 6 | 0807c876 | high | same-bytes | auto | P14 capture locks on initialized native melodic voice |
| 0x807ebe0 | 6 | 0807f360 | high | same-bytes | auto | P14 generic melodic note capture |
| 0x808414a | 4 | - | fuzzy-changed | no-site | unresolved | CH6 chord pad audition (R6.6 via transpose): track release |
| 0x8084180 | 4 | 08084bfc | low-aligned | relocated-bytes | review | CH6 chord pad audition (R6.6 via transpose): layer release |
| 0x8084918 | 4 | 0808537e | fuzzy-equal | relocated-bytes | review | CH6 chord pad audition (R6.6 via transpose): track press |
| 0x808494e | 4 | 080853b4 | low-aligned | relocated-bytes | review | CH6 chord pad audition (R6.6 via transpose): layer press |
| 0x80857d6 | 4 | 08086256 | low-aligned | relocated-bytes | review | P1 fixed velocity before the native pad press |
| 0x808773a | 4 | 0808817a | high | relocated-bytes | auto | P15 Env 1 phase entry: note-owned decay/release |
| 0x8087aae | 4 | 080884ee | high | relocated-bytes | auto | P15 Env 1 phase entry: note-owned decay/release |
| 0x808d0dc | 4 | 0808dbbc | high | same-bytes | auto | W22 publish PLAY-bank transitions once per validated raw input packet |
| 0x808d1fe | 4 | 0808dcde | high | same-bytes | auto | input branch |
| 0x808e9f0 | 4 | 0808f4f0 | medium | same-bytes | review | L3 exact-version native integration |
| 0x808ed22 | 4 | 0808f822 | medium | same-bytes | review | L3 exact-version native integration |
| 0x808f38e | 4 | 0808fe8e | medium | same-bytes | review | CH11 synth destinations in the native route command |
| 0x808f39c | 4 | 0808fe9c | medium | same-bytes | review | L3 M7 application hook |
| 0x808fc40 | 4 | 08090740 | medium | same-bytes | review | L3 exact-version native integration |
| 0x808fcc2 | 4 | 080907c2 | medium | same-bytes | review | L3 exact-version native integration |
| 0x8091820 | 4 | 08092320 | high | same-bytes | auto | K7 native per-voice render entry; original path on ordinary samples |
| 0x8093cda | 4 | 080947da | high | same-bytes | auto | L3 exact-version native integration |
| 0x8094ade | 4 | 080955de | high | same-bytes | auto | L3 exact-version native integration |
| 0x809fd80 | 4 | 080a09a0 | medium | same-bytes | review | L3 M7 application hook |
| 0x80a050c | 8 | 080a112c | medium | relocated-bytes | review | P15 snapshot native track gain/duck before state advances; TF2 per-track scratch after nat |
| 0x80a0526 | 4 | 080a1146 | low-aligned | relocated-bytes | review | P15 snapshot native track gain/duck before state advances |
| 0x80a0532 | 4 | 080a1152 | low-aligned | relocated-bytes | review | P15 per-note send bus mix; native fallback without send locks |
| 0x80a053e | 4 | 080a115e | low-aligned | relocated-bytes | review | P15 per-note send bus mix; native fallback without send locks |
| 0x80a055c | 6 | 080a117c | low | same-bytes | review | CH9 meter after Track FX: meter offset 0 (post-level buffer) |
| 0x80a0566 | 2 | 080a1186 | low | same-bytes | review | CH9 meter after Track FX: meter the post-level, post-Track-FX buffer |
| 0x80a05c0 | 4 | 080a11e0 | medium | same-bytes | review | CH9 meter after Track FX: other track type: same zero offset |
| 0x80a05e0 | 4 | 080a1200 | medium | same-bytes | review | CH9 meter after Track FX: other track type: same zero offset |
| 0x80a3bdc | 4 | 080a437c | high | same-bytes | auto | Reserve 28064 bytes: kick, lock, condition and UI state; stack unchanged |
| 0x80a3e94 | 4 | 080a4634 | high | same-bytes | auto | Reserve 28064 bytes: kick, lock, condition and UI state; stack unchanged |
| 0x80ac3e4 | 4 | 080acbb4 | medium | relocated-bytes | review | A1 WT oscillator: analog render |
| 0x80ac7e0 | 4 | 080acfb0 | medium | relocated-bytes | review | A1 Wavetable voice: note analog mode |
| 0x81004b6 | 4 | 081004b6 | medium | same-bytes | review | UI startup branch |
| 0x8104296 | 4 | 08104442 | high | relocated-bytes | auto | Carrier listing filter: browser_list_hook |
| 0x810dff4 | 4 | 0810e2c0 | high | same-bytes | auto | K7 record native file loading on eager sample load |
| 0x810e0b8 | 4 | 0810e384 | high | same-bytes | auto | K7 record native file loading on sample load, including project reload |
| 0x8114dbe | 4 | 0811512a | medium | same-bytes | review | MPU A/B: M4 MPU region 1 base 08100000 -> 08000000 |
| 0x8114dc4 | 2 | 08115130 | medium | same-bytes | review | MPU A/B: M4 MPU region 1 size 1 MB -> 2 MB |
| 0x81257f0 | 4 | 08125b5c | high | same-bytes | auto | W22 transient PLAY mute colours around native LED flush |
| 0x812a6bc | 4 | 0812b2e0 | medium | same-bytes | review | Cancel dequeued native Save/Save As before filesystem work if rejected macro data could no |
| 0x812d978 | 4 | 0812e868 | high | same-bytes | auto | Release frozen custom drum settings with their native copied cell |
| 0x812e296 | 4 | 0812f18a | high | same-bytes | auto | Preserve frozen custom drum settings across native Save/Undo cell copies |
| 0x812f4f0 | 4 | 081303e4 | low | same-bytes | review | L3 exact-version native integration |
| 0x812f5ea | 4 | 081304de | medium | same-bytes | review | K7 versioned kick attributes on native pad params save |
| 0x812f6a2 | 4 | 08130596 | medium | relocated-bytes | review | CH11 Mod Matrix: save synth destination names |
| 0x812faac | 4 | 081309a0 | high | same-bytes | auto | K7 restore kick attributes before native pad params load |
| 0x812fbb4 | 4 | 08130aa8 | high | same-bytes | auto | L3 exact-version native integration |
| 0x812fe52 | 4 | 08130d46 | high | relocated-bytes | auto | CH11 Mod Matrix: load synth destination names |
| 0x813044c | 4 | 08131340 | high | same-bytes | auto | R6.18 black box: crash/boot lines after stock card work (chain 0x8130958) |
| 0x8130522 | 4 | 08131416 | high | same-bytes | auto | R6.18 black box: crash/boot lines after stock card work (chain 0x8130a18) |
| 0x8130774 | 4 | 08131668 | high | same-bytes | auto | R6.18 black box: crash/boot lines after stock card work (chain 0x8130a18) |
| 0x8131054 | 3 | 08132014 | low | different-bytes | unresolved | Playground palette / release identity |
| 0x8133614 | 4 | 08134a70 | low-aligned | same-bytes | review | Read private Track FX values without changing native properties |
| 0x81338dc | 4 | 08134d38 | low-aligned | same-bytes | review | Capture per-sound cycle metadata with the native one-level note snapshot |
| 0x8134800 | 4 | 08135d74 | low | different-bytes | unresolved | Persistent header custom actions (Select/Chord) with visibility sync |
| 0x8134b58 | 4 | 081360d0 | high | same-bytes | auto | Use native parameter-row menu for Track FX only |
| 0x8139984 | 4 | 0813b0b8 | low-aligned | same-bytes | review | Unlit empty steps; native active-step track color remains unchanged |
| 0x813b290 | 4 | 0813ca4e | fuzzy-equal | same-bytes | review | Restore per-sound cycle metadata only after successful native note Undo |
| 0x813bd10 | 4 | 0813d504 | high | same-bytes | auto | Drum selection observer branch |
| 0x813c3f0 | 4 | 0813dbf0 | low-aligned | same-bytes | review | L3 exact-version native integration |
| 0x813f98c | 4 | - | fuzzy-changed | no-site | unresolved | consumer branch |
| 0x81403ec | 4 | 08141ea4 | low-aligned | same-bytes | review | A1 model events: Analog Sync stays private |
| 0x8140924 | 4 | 08142412 | fuzzy-equal | relocated-bytes | review | CH11 Mod Matrix: editing a synth route adopts the pad engine |
| 0x8140ef0 | 27 | 08142a38 | high | same-bytes | auto | Playground palette / release identity |
| 0x8140f0c | 24 | 08142a54 | high | same-bytes | auto | Playground palette / release identity |
| 0x8140f50 | 52 | 08142a98 | high | same-bytes | auto | Playground palette / release identity |
| 0x8141370 | 2 | 08142eb4 | low-aligned | same-bytes | review | A1 instrument menu F6 shows nine cells (native maximum) |
| 0x81415f0 | 4 | 08143138 | medium | same-bytes | review | EU10 four menu rows put fifth Select button directly right of Undo |
| 0x81415f8 | 2 | 08143140 | medium | same-bytes | review | EU10 populate fifth FA menu cell |
| 0x81420ec | 4 | 08143d44 | low | same-bytes | review | Leave custom mode on ordinary navigation |
| 0x8145d90 | 4 | 081479fc | high | same-bytes | auto | Handle native encoder/touch events only on custom FX rows |
| 0x814604e | 4 | 08147cba | medium | relocated-bytes | review | Bound four Perform rows above native footer; other row layouts unchanged |
| 0x814ce80 | 4 | - | fuzzy-changed | no-site | unresolved | CH11 synth pad rows in the native Mod Matrix |
| 0x81512a4 | 4 | 08152fa0 | low-aligned | same-bytes | review | Prepare built-in kick entry before native one-shot Load |
| 0x8152144 | 4 | 08153df4 | high | same-bytes | auto | K7 custom kick rows around original sample-editor refresh |
| 0x8152c50 | 4 | 08154970 | low-aligned | same-bytes | review | K7 synth encoder/pad/load events with ordinary-sample fallback |
| 0x8158a7c | 4 | 0815a7c0 | medium | same-bytes | review | T2 condition UI value conversion |
| 0x8158f1c | 4 | 0815ac60 | high | same-bytes | auto | T2 condition UI value conversion |
| 0x81599bc | 4 | 0815b730 | low-aligned | same-bytes | review | CH1 capture native pitched piano-roll context |
| 0x8159b80 | 4 | 0815b8f2 | low-aligned | same-bytes | review | T2 condition UI value conversion |
| 0x815a598 | 4 | 0815c304 | low-aligned | same-bytes | review | CH1 native Chord toolbar action; ordinary piano events fall through |
| 0x815b548 | 4 | 0815d2c0 | low-aligned | same-bytes | review | CH5 live chord Left/Right; other piano hardware events stay native |
| 0x815b9c0 | 6 | 0815d740 | high | same-bytes | auto | T2 selected-step matching with native held steps taking priority |
| 0x815bab0 | 4 | 0815d830 | high | same-bytes | auto | T2 edits use selected inspected hit when no pad is held |
| 0x815bde2 | 4 | 0815db62 | high | same-bytes | auto | EU10 generated playhead overrides native clip position only in Euclid bank |
| 0x815c11e | 2 | 0815de9e | high | same-bytes | auto | W14 centre native pad selector between name column and Euclidean tab |
| 0x815c27c | 4 | 0815dffc | medium | same-bytes | review | L3 exact-version native integration |
| 0x815c3d2 | 4 | 0815e152 | medium | same-bytes | review | T2 condition UI value conversion |
| 0x815c7c4 | 4 | 0815e544 | high | same-bytes | auto | T2 encoder edit suppresses native release toggle |
| 0x815c8ac | 4 | 0815e62c | high | same-bytes | auto | T2 encoder edit suppresses native release toggle |
| 0x815c8b4 | 4 | 0815e634 | high | same-bytes | auto | T2 condition UI value conversion |
| 0x815c990 | 4 | 0815e710 | medium | same-bytes | review | T2 encoder edit suppresses native release toggle |
| 0x815caa0 | 4 | 0815e820 | low-aligned | same-bytes | review | L3 exact-version native integration |
| 0x815d1ec | 4 | 0815ef98 | low-aligned | same-bytes | review | EU10 selection gesture consumes native toggle path |
| 0x815d22e | 4 | 0815efda | medium | relocated-bytes | review | T2 remember inspected native physical/touch step |
| 0x815d23c | 4 | 0815efe8 | medium | same-bytes | review | EU10 selection gesture consumes native toggle path |
| 0x815d29c | 4 | 0815f048 | medium | same-bytes | review | EU10 selection gesture consumes native toggle path |
| 0x815d2ea | 4 | 0815f096 | medium | relocated-bytes | review | T2 remember inspected native physical/touch step |
| 0x815d2f4 | 4 | 0815f0a0 | medium | same-bytes | review | EU10 selection gesture consumes native toggle path |
| 0x815d444 | 6 | 0815f1f0 | medium | same-bytes | review | EU10 selection gesture consumes native toggle path |
| 0x815d57e | 4 | 0815f32a | medium | same-bytes | review | EU10 native SEQ arrows swap encoder banks and reuse sequence navigation |
| 0x815d600 | 4 | 0815f3ac | low-aligned | same-bytes | review | EU10 native Select action dispatcher |
| 0x815d968 | 4 | 0815f714 | fuzzy-equal | relocated-bytes | review | W22 waveform text resolves current instrument without stale shared buffers |
| 0x815db22 | 4 | 0815f8ce | low-aligned | relocated-bytes | review | W22 waveform text resolves current instrument without stale shared buffers |
| 0x815dc0e | 4 | 0815f9ba | fuzzy-equal | relocated-bytes | review | W22 waveform text resolves current instrument without stale shared buffers |
| 0x815e1ae | 4 | 0815ff5e | medium | relocated-bytes | review | W22 waveform text resolves current instrument without stale shared buffers |
| 0x815e1b8 | 4 | 0815ff68 | medium | relocated-bytes | review | W22 waveform text resolves current instrument without stale shared buffers |
| 0x815e1e2 | 4 | 0815ff92 | medium | relocated-bytes | review | W22 waveform text resolves current instrument without stale shared buffers |
| 0x815f5ae | 4 | 08161364 | fuzzy-equal | relocated-bytes | review | A1 synth page keys: Granular stays native |
| 0x815f68e | 4 | 08161446 | fuzzy-equal | relocated-bytes | review | A1 synth page keys: Wavetable/Analog |
| 0x8162a9a | 4 | 0816491a | medium | same-bytes | review | Playground palette / release identity |
| 0x8162a9f | 7 | 0816491f | medium | same-bytes | review | Playground palette / release identity |
| 0x8162ad8 | 3 | 08164958 | low | different-bytes | unresolved | Playground palette / release identity |
| 0x8163470 | 4 | 08165310 | high | relocated-bytes | auto | G12 route hardware fingers through macro hit testing |
| 0x8165d40 | 4 | 08167b94 | fuzzy-equal | same-bytes | review | T2 extend native condition list count without changing other fields |
| 0x8165f8c | 4 | 08167e0c | low | relocated-bytes | review | T2 native on-screen probability/condition labels |
| 0x8168768 | 4 | 0816a768 | high | same-bytes | auto | K13 proven direct lower-editor draw filter |
| 0x816a97c | 4 | 0816c9c4 | high | same-bytes | auto | K13 proven direct lower-editor draw filter |
| 0x816ca0e | 4 | 0816eaea | high | relocated-bytes | auto | Carrier listing filter: browser_request |
| 0x8171e44 | 4 | 08173fa4 | high | same-bytes | auto | K15 fivefold custom synth encoder response |
| 0x8176ee2 | 2 | 08179062 | medium | same-bytes | review | G2 Playground piano grid: faint bar lines |
| 0x8179120 | 4 | 0817b29c | high | same-bytes | auto | K13 proven direct lower-editor draw filter |
| 0x8179564 | 4 | 0817b6e0 | high | same-bytes | auto | INST native static-label track colours |
| 0x8179770 | 4 | 0817b8ec | high | same-bytes | auto | INST native static-label track colours |
| 0x81797be | 4 | 0817b93a | high | same-bytes | auto | INST native static-label track colours |
| 0x81797e6 | 4 | 0817b962 | high | same-bytes | auto | INST native static-label track colours |
| 0x817ddfc | 4 | 0817ff7c | high | same-bytes | auto | K13 proven direct lower-editor draw filter |
| 0x817e780 | 4 | 08180900 | high | same-bytes | auto | K13 proven direct lower-editor draw filter |
| 0x817faf8 | 4 | 08181c7c | low-aligned | same-bytes | review | P12 custom controls at final native header application |
| 0x8183854 | 4 | 08185a78 | high | same-bytes | auto | K13 proven direct lower-editor draw filter |
| 0x818bad4 | 7 | 0818dd94 | low-aligned | different-bytes | unresolved | Visible experiment identifier; synth cave growth: Visible W22 Wavetable Position Motion an |
| 0x8198298 | 20 | - | unmatched | no-site | unresolved | A1 Analog entry in the instrument menu |
| 0x8198770 | 180 | - | unmatched | no-site | unresolved | EU10 Select beside Undo in existing blank F9 cell; preserve eight actions |
| 0x8198db0 | 100 | - | unmatched | no-site | unresolved | EU10 native Undo Select Length+ Length- Duplicate menu |
| 0x819dc50 | 4 | 081a0b18 | medium | relocated-bytes | review | P6 Drum Synth action on the pad page view |
| 0x819dce0 | 4 | 081a0ba8 | medium | relocated-bytes | review | Launch native layout, markers and group selection: pg_launch_tick |
| 0x819dd10 | 4 | 081a0bd8 | medium | relocated-bytes | review | Launch native layout, markers and group selection: pg_launch_event |
| 0x819dd20 | 12 | 081a0be8 | medium | relocated-bytes | review | Native mixer render adapter runs on UI thread; W7 Mixer: pg_mixer_paint; W7 Mixer: pg_mixe |
| 0x819dd30 | 12 | 081a0bf8 | medium | relocated-bytes | review | W7 Mixer: pg_mixer_down; W7 Mixer: pg_mixer_move; W7 Mixer: pg_mixer_up |
| 0x819dd48 | 4 | 081a0c10 | medium | relocated-bytes | review | W7 Mixer: pg_mixer_hit |
| 0x819dd50 | 8 | 081a0c18 | medium | relocated-bytes | review | W7 Mixer: pg_mixer_receive; W7 Mixer: pg_mixer_event |
| 0x819e52c | 4 | 081a13f4 | low | relocated-bytes | review | A1 instrument menu: Analog action, Wavetable clears it |
| 0x819e7b4 | 4 | 081a167c | low | relocated-bytes | review | W14 HW3 melodic Euclid velocity glyph after native piano child drawing |
| 0x819e818 | 8 | 081a16e0 | low | relocated-bytes | review | G1 Playground step cell: pg_cell_paint; G1 Playground step cell: pg_cell_walk |
| 0x819e858 | 4 | 081a1720 | medium | relocated-bytes | review | EU10 sequencer-only velocity glyph after native child drawing |
| 0x819e894 | 4 | 081a175c | medium | relocated-bytes | review | P8 mod mode: mod_view_tick |
| 0x819e8c4 | 4 | 081a178c | medium | relocated-bytes | review | P8 mod mode: mod_view_event |
| 0x819eac8 | 4 | 081a1990 | medium | relocated-bytes | review | W5 Tracks: pg_tracks_tick |
| 0x819eafc | 4 | 081a19c4 | medium | relocated-bytes | review | W5 Tracks: pg_tracks_event |
| 0x81b2328 | 4 | 081b51f0 | medium | relocated-bytes | review | P29 macro strip catcher where another widget wins the strip area |
| 0x81b2838 | 12 | 081b5718 | medium | relocated-bytes | review | P27 macro strip touches: macro_bar_tick; W5 Tracks: pg_bar_paint; W5 Tracks: pg_bar_walk |
| 0x81b2848 | 16 | 081b5728 | medium | relocated-bytes | review | P27 macro strip touches: macro_bar_down; P27 macro strip touches: macro_bar_move; P27 macr |
| 0x81b2948 | 8 | 081b5828 | medium | relocated-bytes | review | Launch native layout, markers and group selection: pg_launch_cell_paint; Launch native lay |
| 0x81b29bc | 4 | 081b589c | low | relocated-bytes | review | Launch native layout, markers and group selection: pg_launch_column_event |
| 0x81b2bcc | 8 | 081b5aac | medium | relocated-bytes | review | CH6 piano-grid lanes: lane_tick; CH6 piano-grid lanes: lane_paint |
| 0x81b2bdc | 12 | 081b5abc | medium | relocated-bytes | review | CH6 piano-grid lanes: grid_touch_down; CH6 piano-grid lanes: grid_touch_move; CH6 piano-gr |
| 0x81b2e8c | 8 | 081b5d6c | medium | relocated-bytes | review | Minimalist INST bank: inst_bank_paint; Minimalist INST bank: inst_bank_walk |
| 0x81b35c4 | 4 | 081b64a4 | medium | relocated-bytes | review | Launch native layout, markers and group selection: pg_launch_header_paint |
| 0x81b36b4 | 4 | 081b6594 | medium | relocated-bytes | review | P20 keep native tab lights off while Chord/Euclid is shown |
| 0x81b38d4 | 8 | 081b67cc | medium | relocated-bytes | review | W5 Tracks: pg_column_paint; W5 Tracks: pg_column_walk |
| 0x81b38e0 | 12 | 081b67d8 | medium | relocated-bytes | review | W5 Tracks: pg_column_down; W5 Tracks: pg_column_move; W5 Tracks: pg_column_up |
| 0x81b38f8 | 4 | 081b67f0 | medium | relocated-bytes | review | W5 Tracks: pg_column_hit |
| 0x81b3a18 | 8 | 081b6910 | medium | relocated-bytes | review | P4 Analog wave pane: analog_wave_tick; P4 Analog wave pane: analog_wave_paint |
| 0x81b3e40 | 3 | 081b6d38 | medium | same-bytes | review | Playground palette / release identity |
| 0x81b3e44 | 3 | 081b6d3c | medium | same-bytes | review | Playground palette / release identity |
| 0x81b3e48 | 3 | 081b6d40 | medium | same-bytes | review | Playground palette / release identity |
| 0x81b3e50 | 3 | 081b6d48 | medium | same-bytes | review | Playground palette / release identity |
| 0x81b3e54 | 3 | 081b6d4c | medium | same-bytes | review | Playground palette / release identity |
| 0x81b3e58 | 3 | 081b6d50 | medium | same-bytes | review | Playground palette / release identity |
| 0x81b3e5c | 3 | 081b6d54 | medium | same-bytes | review | Playground palette / release identity |
| 0x81b3e60 | 3 | 081b6d58 | medium | same-bytes | review | Playground palette / release identity |
| 0x81b3e64 | 3 | 081b6d5c | medium | same-bytes | review | Playground palette / release identity |
| 0x81b3e68 | 3 | 081b6d60 | medium | same-bytes | review | Playground palette / release identity |
| 0x81b3e6c | 3 | 081b6d64 | medium | same-bytes | review | Playground palette / release identity |
| 0x81b3e70 | 3 | 081b6d68 | medium | same-bytes | review | Playground palette / release identity |
