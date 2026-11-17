# Architecture

## Startup

`main` ([i_main.c](../linuxdoom-1.10/i_main.c)) calls `D_DoomMain` ([d_main.c](../linuxdoom-1.10/d_main.c)), which runs these steps in order:

1. `IdentifyVersion` searches `$DOOMWADDIR` (or `.`) for an IWAD and sets `gamemode`
   (`shareware`/`registered`/`commercial`/`retail`) and `gamemission`
   (`doom`/`doom2`/`pack_tnt`/`pack_plut`).
2. It parses command-line flags (`-warp`, `-skill`, `-file`, `-playdemo`, `-net`, ...)
   with `M_CheckParm` ([m_argv.c](../linuxdoom-1.10/m_argv.c)).
3. It initialises the subsystems: `V_Init`, `M_LoadDefaults`, `Z_Init`, `W_InitMultipleFiles`,
   `M_Init`, `R_Init`, `P_Init`, `I_Init`, `D_CheckNetGame`, `S_Init`, `HU_Init`, `ST_Init`.
4. It starts a demo, a game, or a loaded game, and then enters `D_DoomLoop`, which never returns.

## Main loop and the tic model

The simulation runs in fixed **tics** at `TICRATE` = 35 Hz. Rendering runs as fast as the
host allows, but the game only advances in whole tics.

```
D_DoomLoop                       (d_main.c)
 └─ loop forever:
     I_StartFrame
     TryRunTics                  (d_net.c)  ── decides how many tics to run
     │   NetUpdate               build local ticcmds, exchange them with peers
     │   for each ready tic:
     │       M_Ticker            menu
     │       G_Ticker            (g_game.c) game-state machine
     │         └─ P_Ticker       (p_tick.c) run all thinkers   ← GS_LEVEL only
     │            ST_Ticker, AM_Ticker, HU_Ticker
     │         or WI_Ticker / F_Ticker / D_PageTicker
     S_UpdateSounds
     D_Display                   R_RenderPlayerView + HUD/status bar/automap/menu, wipe
     I_UpdateSound / I_SubmitSound   (unless SNDSERV / SNDINTR is defined)
```

- **Input → `ticcmd_t`.** `I_*` turns OS input into `event_t`s. `D_ProcessEvents` passes
  them to the responders (`M_Responder`, then `G_Responder`). `G_BuildTiccmd` turns the
  held input into one `ticcmd_t` per tic ([d_ticcmd.h](../linuxdoom-1.10/d_ticcmd.h)).
- **Ticcmds drive everything.** The game state at tic N depends only on the starting state and
  every player's ticcmds up to N. Demos and lockstep netplay both rely on this.
- **`gamestate`** is one of `GS_LEVEL`, `GS_INTERMISSION`, `GS_FINALE`, or `GS_DEMOSCREEN`.
  **`gameaction`** queues transitions such as `ga_loadlevel`, `ga_completed`, and
  `ga_worlddone`. `G_Ticker` acts on them at the start of the next tic.

## Module map (file prefix → responsibility)

| Prefix | Subsystem | Key files |
|---|---|---|
| `i_` | **Platform layer**: video, input, sound, net, timers. Port by replacing these. | i_video.c, i_sound.c, i_net.c, i_system.c, i_main.c |
| `d_` | Top-level driver, net/tic orchestration, shared defs | d_main.c, d_net.c, d_player.h, d_think.h, d_items.c |
| `g_` | Game flow: new game/level, save/load, demos, ticcmd building | g_game.c |
| `p_` | **Playsim**: map objects, physics, AI, sector specials | see [playsim.md](playsim.md) |
| `r_` | **Renderer**: software BSP renderer | see [rendering.md](rendering.md) |
| `v_` | Framebuffer and patch drawing (`screens[0..4]`) | v_video.c |
| `w_` | WAD/lump I/O | w_wad.c |
| `z_` | Zone memory allocator | z_zone.c |
| `s_` | Sound logic: channels, positioning (platform-independent) | s_sound.c, sounds.c |
| `m_` | Misc: menu, args, fixed point, RNG, cheats, bbox, config | m_menu.c, m_fixed.c, m_random.c, m_misc.c |
| `st_` | Status bar | st_stuff.c, st_lib.c |
| `hu_` | Heads-up messages and chat | hu_stuff.c, hu_lib.c |
| `am_` | Automap | am_map.c |
| `wi_` | Intermission screens | wi_stuff.c |
| `f_` | Finale text/cast, screen wipe | f_finale.c, f_wipe.c |
| (none) | Global data and tables | doomdef.h, doomstat.c, doomdata.h, info.c, tables.c, dstrings.c |

## Global state

Most shared state is kept in global variables declared in [doomstat.h](../linuxdoom-1.10/doomstat.h):
`gamemode`, `gameskill`, `gameepisode`, `gamemap`, `players[MAXPLAYERS]`, `consoleplayer`,
`displayplayer`, `netgame`, `deathmatch`, `demoplayback`, `leveltime`, and so on.
`MAXPLAYERS` is 4.

## Data-driven game definition

[info.c](../linuxdoom-1.10/info.c) / [info.h](../linuxdoom-1.10/info.h) define:

- `states[]` (`statenum_t`): animation frames. Each one has a sprite, frame, duration in tics,
  an action function (`A_*`), and the next state.
- `mobjinfo[]` (`mobjtype_t`): every thing type, with its doomednum, health, speed, radius,
  height, flags, sounds, and state entry points.
- `sprnames[]`.

Action functions (`A_Chase`, `A_FireShotgun`, ...) live in [p_enemy.c](../linuxdoom-1.10/p_enemy.c)
and [p_pspr.c](../linuxdoom-1.10/p_pspr.c). To add a monster or weapon, you extend these
tables and write the action functions.
