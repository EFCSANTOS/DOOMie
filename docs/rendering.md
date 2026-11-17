# Renderer (`r_*`)

DOOM uses a software renderer at 320×200 with 8-bit paletted output (`SCREENWIDTH`/`SCREENHEIGHT`
in [doomdef.h](../linuxdoom-1.10/doomdef.h)). Walls are drawn as vertical columns of constant
depth. Floors and ceilings are drawn as horizontal spans of constant depth. Lighting is a
colormap lookup chosen by light level and distance.

## Frame pipeline

`R_RenderPlayerView` ([r_main.c](../linuxdoom-1.10/r_main.c)) runs these steps:

1. `R_SetupFrame`: sets up the view position, angle, and fixed colormap (invulnerability, light amp).
2. `R_ClearClipSegs`, `R_ClearDrawSegs`, `R_ClearPlanes`, `R_ClearSprites`.
3. **`R_RenderBSPNode`** ([r_bsp.c](../linuxdoom-1.10/r_bsp.c)) walks the BSP front to back.
   - `R_CheckBBox` rejects subtrees that are behind the view or already fully covered (`solidsegs`).
   - `R_Subsector` → `R_AddLine` for each seg → `R_StoreWallRange` ([r_segs.c](../linuxdoom-1.10/r_segs.c))
     draws the wall columns straight away and records a `drawseg`. It also marks the floor and
     ceiling ranges in `visplane_t`s.
   - `R_AddSprites` queues the subsector's things as `vissprite_t`s.
4. **`R_DrawPlanes`** ([r_plane.c](../linuxdoom-1.10/r_plane.c)) turns each visplane into
   horizontal spans (`R_MakeSpans` → `R_MapPlane` → `R_DrawSpan`). The sky is drawn as columns.
5. **`R_DrawMasked`** ([r_things.c](../linuxdoom-1.10/r_things.c)) sorts the sprites back to front,
   clips them against drawsegs, and draws masked mid-textures and player weapon sprites (psprites).

The low-level inner loops are in [r_draw.c](../linuxdoom-1.10/r_draw.c): `R_DrawColumn`,
`R_DrawSpan`, `R_DrawFuzzColumn` (spectre), and `R_DrawTranslatedColumn` (player colours).
`detailshift` selects low detail (double-wide pixels).

## Data

- [r_defs.h](../linuxdoom-1.10/r_defs.h) defines the runtime map structures (`vertex_t`,
  `sector_t`, `side_t`, `line_t`, `subsector_t`, `seg_t`, `node_t`), plus `drawseg_t` and `vissprite_t`.
- [r_data.c](../linuxdoom-1.10/r_data.c) builds textures from patches at startup (`R_InitTextures`,
  `R_GenerateComposite`), and loads flats, sprites, and colormaps. It also handles texture
  and flat animation translation (`texturetranslation`, `flattranslation`).
- [r_sky.c](../linuxdoom-1.10/r_sky.c) handles the sky texture, which is chosen by game mode and episode.
- [tables.c](../linuxdoom-1.10/tables.c) holds the `finesine`, `finetangent`, and `tantoangle` LUTs.

## Hard limits (overflowing them crashes the game or corrupts memory)

| Constant | Value | Where |
|---|---|---|
| `MAXVISPLANES` | 128 | r_plane.c ("no more visplanes" I_Error) |
| `MAXOPENINGS` | `SCREENWIDTH*64` | r_plane.c |
| `MAXDRAWSEGS` | 256 | r_defs.h (overflow is silently ignored) |
| `MAXSEGS` (solidsegs) | 32 | r_bsp.c |
| `MAXVISSPRITES` | 128 | r_things.h |

## Known quirks and ideas

- Raising the resolution means changing more than `SCREENWIDTH`/`SCREENHEIGHT`. The status bar,
  HUD, menus, and `V_DrawPatch` all assume 320×200 graphics.
- Carmack's suggested redesign ([README.TXT](../README.TXT)): do one front-to-back BSP walk
  that collects data, then draw each subsector's contents on the way back up. This needs
  floors and ceilings treated as polygons.
- Tutti-frutti, medusa, and slime trails are well-known artefacts of the original renderer.
  Some demos and maps depend on renderer behaviour only visually, never in the simulation.
