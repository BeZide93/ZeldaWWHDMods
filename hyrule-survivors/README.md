# Hyrule Survivors — Savage Labyrinth prototype

An experimental, standalone **ZeldaWWHDRecomp guest mod**. It starts a six-wave
Survivors expedition in a separate copy of **Savage Labyrinth floor 1
(`Cave09/Room1`)**. The original room's geometry, collision and native enemy
assets come from the player's installed game. This repository contains only
source code and configuration; no game models, textures or map archives.

The **0.1.0 room/run prototype has been reported working in-game by the user**.
Version **0.2.0 replaces its drawn spirit attacks with native abilities and Link
animations**. Version 0.2.1 fixes the reported native-combat issues: automatic
boomerang cleanup after catching, third-person automatic bow shots, and bombs
that explode on ground contact without hitting Link. These fixes still need
an in-game acceptance pass. Version 0.2.2 additionally addresses the reported
Moblin-kill teardown path: Moblins finish their native death before their pool
slot is freed. The crash log identifies a failure during Moblin graphics
teardown; this lifecycle correction is not proof that the original heap fault
is eliminated. Automated tests execute the translated adapter against synthetic game
calls; they cannot verify animation appearance, actual physics or balancing.

**0.2.3 is a mitigation/diagnostic build for the repeated Android crash.**
The second log fails during Moblin model construction, whereas the first failed
during destruction. Moblins now have a separate limit of **four resident actors**,
including pending, dying and retiring actors. Remaining wave enemies wait for a
slot; the 113-enemy total is unchanged. This lowers resource pressure but does
not prove the cause or a fix for either crash. Native construction logs include
the actor ID and current heap's free bytes before/after model creation. The menu
and entry log show 0.2.3 so the installed version can be checked. If it crashes
again, include the full runtime log alongside the crash log: the latter alone
does not include these measurements.

## Install and start

1. Use a current ZeldaWWHDRecomp build with guest mods, custom stages, live
   touch/input services and the vector HUD from this repository.
2. After catalogue publication, install **Hyrule Survivors** from **Mods → Community**
   with **Show experimental mods** enabled. For local tests, install the generated package.
3. In **Mods → Installed packages**, choose the package, install it and enable
   it. Allow the normal setup/rebuild and restart the game when requested.
4. Load a Quest Log and stand in an ordinary playable area. Tap/click
   **HYRULE SURVIVORS**, or press **L + R + Minus**. Alternatively, switch on
   **Open Survivors** in this mod's options, then close the menu.
5. Choose **Begin expedition**. After the floor check, press **A** to start.

Only the recompilation is supported: this package does not install into a Wii U
or Cemu. No Dawnlight dependency or direct Dawnlight binary port is included.
Keep a Quest Log backup while testing the new native-combat integration.

## Included gameplay

| Wave | Enemies | Count |
| --- | --- | ---: |
| Wings in the dark | Keese | 12 |
| Emerald tide | ChuChus | 16 |
| Little devils | Miniblins | 20 |
| Bokoblin assault | Bokoblins | 24 |
| Moblin guard | Moblins | 28 |
| The labyrinth champion | Mixed enemies, then an elite Moblin | 13 |

The run ends in victory after all **113 enemies** are defeated. At most **24
native enemies**, including pending creations and removals, can occupy the
pool, with a separate limit of **four Moblins**. The champion is an ordinary native Moblin with **160 Survivors HP**;
it has a boss health bar, but no new model or AI. Every cleared intermediate
wave heals one heart and collects remaining EXP.

- **Spin Attack:** Link performs his native spin, with sword animation, effects
  and attack collision. Replaces Spirit Sword.
- **Hero's Bow:** Link loads and fires a native arrow with the bow animations;
  the launch direction aims at the selected enemy. Native walls and impacts apply.
- **Boomerang:** native throwing animation, target lock, flight and return/catch;
  the returned item is automatically put away after its catch animation.
- **Bombs:** Link takes out and throws a native bomb. Ground contact triggers
  the native explosion before a bounce; the normal fuse remains a fallback.
  Run bombs cannot damage or knock back Link; enemy damage remains active.
- **EXP and levels:** collect green orbs; choose one of three upgrades while
  combat is paused. Weapon ranks, damage, cooldown, reach, pickup radius,
  heart containers and healing are available. The first three level-ups each
  offer one new weapon; choosing it remains optional.
- **HUD:** health, EXP, level, wave, timer, enemy count, kills, weapon ranks,
  champion HP, pause, results and per-weapon damage totals.
- **Results:** victory/defeat, retry, return to the starting location, persistent
  best kill count and victory count in the mod's own `Data/records-v1.bin`.

Attacks start one at a time through a fair rotation of ready abilities. They
wait while Link rolls, falls, takes damage or completes another attack. Bow and
boomerang use native upper-body animations without forcing first-person aiming;
spin and bomb throws use their normal full-body actions. Native projectile
motion and bomb fuses pause during upgrades/pause, and owned items are cleaned
up on results, retry and return. Up to 16 native weapon actors are tracked.

Survivors HP, damage upgrades and the damage breakdown remain separate from
vanilla enemy health. Damage is credited only when the native enemy damage path
accepts a tracked attack's collision, once per cast/projectile and enemy. Native
defenses, stun-only paths and wall collision can prevent damage; there is no
fallback radius hit or invisible projectile simulation. Ranks increase damage;
Haste reduces scheduling cooldowns, while animation duration remains native.
Reach enlarges the spin collision and bomb blast. The old custom arrow piercing
and guaranteed boomerang return-leg double hit are removed in favor of native
projectile behavior. EXP pickups and the UI remain vector HUD elements.

## Controls

| Action | Input |
| --- | --- |
| Move / camera / roll / lock-on | Normal game controls |
| Open expedition menu | L + R + Minus, HUD button, or mod option |
| Start / resume / confirm upgrade / retry | A |
| Choose upgrade | D-pad or left stick left/right; tap a card |
| Pause | Plus or HUD Pause button |
| Resume from pause | A or B |
| Return from ready/results | B or Return button |
| Return while paused | Return button |

The HUD labels are English in this initial version. Health is displayed in
quarter-heart units: **24 = six hearts**. Touch, mouse and controller selection
share the same action path. The HUD layout uses the central 1280×720 canvas.

## Run isolation

On entry, the adapter snapshots the story save payload, Quest Log slot and
return position. The run has its own health, weapons, EXP and upgrades. On a
normal return, it restores the payload and returns Link to the entry location.
During a run, the native serializer is wrapped so that a save request writes
the pre-run story payload. Loading a Quest Log cancels the old snapshot,
including when reloading the same slot. Native attacks temporarily select the
Hero's Sword and refill run arrow/bomb capacity and ammunition. Those bytes are
covered by the same snapshot; pending run-ammo deltas are cleared on exit.

This is not a full snapshot of every transient scene/event object. Enter from
ordinary gameplay; avoid active quests/cutscenes or other mods that alter the
same actors, save hooks or scene transitions. Full recomp save states also
rewind the mod's memory; the separate record file is not rewound, so it is a
casual local record rather than a replay-proof leaderboard.

The arena clones the room through the existing stage builder. The adapter
checks connected floor samples before starting and uses them as spawn points.
Creation, disappearance and cleanup failures lead to an interruption screen
instead of silently granting kills or leaving an unwinnable wave. Geometry
and native floor queries still need the in-game check below.

## Build and catalogue validation

Use a clean SDK checkout at the commit in the repository's `sdk.json`.
The catalogue builds the single entrypoint `src/mod.c`; source files, licence
and original artwork are included by the existing repository packager.

```sh
export WWHD_SDK=/absolute/path/to/pinned/ZeldaWWHDRecomp
python3 hyrule-survivors/tests/build_and_test.py
```

The local regression runner requires Python 3, PowerPC-enabled Clang/LLD and a
native Clang compiler. Generated files stay under the repository's ignored
`build/` directory. It does not require game files. For catalogue publication,
commit the sources first and run the repository's documented `tools/guard.py`,
`tools/package.py` and `tools/validate.py` against the pinned SDK. The local
runner's package is a test artifact, not the published catalogue download.

The maintained catalogue generates the download URL, size and checksum.
Contributors edit `catalogue.json`; do not hand-edit `index.json` or commit ELF/ZIP
files. Publication is separate from opening or merging a source PR.
