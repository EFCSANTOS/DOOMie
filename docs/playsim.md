# Play Simulation (`p_*`)

Everything that happens inside a level. This code **must stay deterministic** (see
[networking.md](networking.md) and [../CLAUDE.md](../CLAUDE.md)).

## Thinkers

[d_think.h](../linuxdoom-1.10/d_think.h) defines `thinker_t`, a node in a doubly linked list
(`thinkercap`) with a function pointer. `P_Ticker` ([p_tick.c](../linuxdoom-1.10/p_tick.c))
calls `P_RunThinkers` once per tic.

- Map objects (`mobj_t`) use `P_MobjThinker`.
- Moving sectors and lights are their own thinkers: `ceiling_t`, `floormove_t`, `plat_t`,
  `vldoor_t`, `lightflash_t`, `strobe_t`, `glow_t`, `fireflicker_t`.
- Removal is deferred. `P_RemoveThinker` sets `function.acv = -1`, and the node is freed on the
  next pass. Never free a thinker directly.

## Map objects ([p_mobj.c](../linuxdoom-1.10/p_mobj.c))

- `P_SpawnMobj`, `P_RemoveMobj`, `P_SetMobjState` (runs state action functions, and can chain
  through 0-tic states).
- `P_XYMovement`/`P_ZMovement` apply momentum, friction, and gravity.
- `flags` (`MF_SOLID`, `MF_SHOOTABLE`, `MF_MISSILE`, `MF_NOCLIP`, ...) are in [p_mobj.h](../linuxdoom-1.10/p_mobj.h).
- Things are linked into **sector lists** (for rendering) and the **blockmap** (for collision)
  with `P_SetThingPosition`/`P_UnsetThingPosition`. Always unset the position before moving a
  thing and set it again afterwards.

## Collision and line checks

| File | Role |
|---|---|
| [p_map.c](../linuxdoom-1.10/p_map.c) | `P_CheckPosition`, `P_TryMove`, `P_SlideMove`, `P_LineAttack`, `P_AimLineAttack`, `P_UseLines`, `P_RadiusAttack`, `P_ChangeSector` |
| [p_maputl.c](../linuxdoom-1.10/p_maputl.c) | Blockmap iteration (`P_BlockLinesIterator`, `P_BlockThingsIterator`), `P_PathTraverse` and intercepts (`MAXINTERCEPTS 128`), line-opening calculation |
| [p_sight.c](../linuxdoom-1.10/p_sight.c) | `P_CheckSight`, which uses the REJECT table and then a BSP-based line-of-sight trace |

Collision is 2D plus height checks against floors and ceilings. Things have unlimited height
when checking against other things, a well-known quirk (see TODO).

## AI and actions

- [p_enemy.c](../linuxdoom-1.10/p_enemy.c) holds monster behaviour (`A_Look`, `A_Chase`,
  `A_FaceTarget`, attack functions, boss specials), plus `P_NoiseAlert` (sound propagation
  through sectors).
- [p_pspr.c](../linuxdoom-1.10/p_pspr.c) handles player weapon sprites and weapon actions.
- [p_user.c](../linuxdoom-1.10/p_user.c) applies a player's `ticcmd_t` (movement, view bob, death).
- [p_inter.c](../linuxdoom-1.10/p_inter.c) handles pickups (`P_TouchSpecialThing`) and damage
  (`P_DamageMobj`, `P_KillMobj`).

## World specials

- [p_spec.c](../linuxdoom-1.10/p_spec.c) handles line specials (`P_CrossSpecialLine`,
  `P_ShootSpecialLine`), sector specials (damage, secrets), animated textures and flats,
  and scrolling walls.
- [p_doors.c](../linuxdoom-1.10/p_doors.c), [p_floor.c](../linuxdoom-1.10/p_floor.c),
  [p_ceilng.c](../linuxdoom-1.10/p_ceilng.c), [p_plats.c](../linuxdoom-1.10/p_plats.c),
  [p_lights.c](../linuxdoom-1.10/p_lights.c), [p_switch.c](../linuxdoom-1.10/p_switch.c), and
  [p_telept.c](../linuxdoom-1.10/p_telept.c) each implement one kind of moving sector or effect.
- Limits: `MAXPLATS 30`, `MAXCEILINGS 30`, `MAXSWITCHES 50`, `MAXBUTTONS 16` ([p_spec.h](../linuxdoom-1.10/p_spec.h)).

## Level load and save games

- [p_setup.c](../linuxdoom-1.10/p_setup.c): `P_SetupLevel` loads the map lumps (THINGS,
  LINEDEFS, SIDEDEFS, VERTEXES, SEGS, SSECTORS, NODES, SECTORS, REJECT, BLOCKMAP) into
  `PU_LEVEL` zone memory, and then spawns the things.
- [p_saveg.c](../linuxdoom-1.10/p_saveg.c) writes and reads raw struct dumps (with pointers
  fixed up). The save format depends on struct layout, so changing a saved struct breaks old
  save games and differs between 32-bit and 64-bit builds.

## Randomness

`P_Random()` ([m_random.c](../linuxdoom-1.10/m_random.c)) is a 256-entry table walked by an
index. Each call advances the shared game index. Calling it a different number of times
desyncs demos and netgames. Use `M_Random()` for anything that doesn't affect the simulation.
