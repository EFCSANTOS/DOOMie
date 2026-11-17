# WAD Data and Zone Memory

## WAD files (`w_*`)

[w_wad.c](../linuxdoom-1.10/w_wad.c) loads the IWAD and any PWADs (`-file`) into a single
**lump directory** (`lumpinfo[]`). Lumps in later files override earlier ones with the same name.

- File format: a header (`IWAD`/`PWAD`, number of lumps, directory offset), then a directory of
  `{filepos, size, name[8]}` entries. See [doomdata.h](../linuxdoom-1.10/doomdata.h) and
  [w_wad.h](../linuxdoom-1.10/w_wad.h).
- API: `W_CheckNumForName` (-1 if missing), `W_GetNumForName` (I_Error if missing),
  `W_CacheLumpNum/Name(lump, tag)` (loads the lump into the zone with a purge tag), and `W_LumpLength`.
- Map lumps follow a marker lump (`ExMy` / `MAPxx`) in a fixed order. Sprite and flat namespaces
  sit between `S_START`/`S_END` and `F_START`/`F_END`.
- Map lumps are stored on disk in little-endian order. [m_swap.c](../linuxdoom-1.10/m_swap.c)
  provides `SHORT()`/`LONG()` for big-endian hosts.

### IWAD detection

`IdentifyVersion` ([d_main.c](../linuxdoom-1.10/d_main.c)) looks in `$DOOMWADDIR` (default `.`)
for these files, in order: `doom2f.wad`, `doom2.wad`, `plutonia.wad`, `tnt.wad`, `doomu.wad`,
`doom.wad`, `doom1.wad`. The file it finds sets `gamemode`/`gamemission`/`language`. To get
the 4th episode, rename the Ultimate DOOM IWAD to `doomu.wad`.
PWADs are refused on shareware. Keep that check (see README.b).

## Zone allocator (`z_*`)

[z_zone.c](../linuxdoom-1.10/z_zone.c) is a single block of memory obtained from
`I_ZoneBase` (`mb_used = 6` MB in [i_system.c](../linuxdoom-1.10/i_system.c)). Inside it,
blocks are kept in a circular doubly linked list and allocated first-fit with a roving pointer.

`Z_Malloc(size, tag, user)`:

| Tag | Value | Lifetime |
|---|---|---|
| `PU_STATIC` | 1 | The whole program |
| `PU_SOUND` / `PU_MUSIC` | 2 / 3 | While the sound or music is playing |
| `PU_LEVEL` | 50 | Until the level ends (`Z_FreeTags(PU_LEVEL, PU_PURGELEVEL-1)` in `P_SetupLevel`) |
| `PU_LEVSPEC` | 51 | Level thinkers (doors, floors, ...) |
| `PU_PURGELEVEL` | 100 | Can be purged when space is needed |
| `PU_CACHE` | 101 | Can be purged at any time. Cached lumps usually use this tag |

- **`user` pointer.** If it isn't NULL, the zone writes the block address into `*user`. When the
  block is purged, the zone sets `*user = NULL`. Purgeable blocks (tag ≥ 100) *must* have an
  owner pointer.
- `Z_ChangeTag(ptr, tag)` is used to "lock" a cached lump while you use it, by temporarily
  raising its tag to `PU_STATIC`, and then to release it by setting it back to `PU_CACHE`.
- `Z_CheckHeap` validates the heap. Call it when you suspect memory corruption.
- Each block has a `memblock_t` header with `id == ZONEID`. The header assumes 32-bit pointers,
  which is one of the main problems when building for 64-bit.
- Running out of zone memory produces `Z_Malloc: failed on allocation of N bytes`. Raise
  `mb_used` if you need more.
