# Building and Porting

## Stock build (as released)

```sh
cd linuxdoom-1.10
mkdir -p linux            # Makefile writes objects to $(O)=linux, which doesn't exist yet
make                      # -> linux/linuxxdoom
```

[Makefile](../linuxdoom-1.10/Makefile): `gcc -g -Wall -DNORMALUNIX -DLINUX`, linked with
`-L/usr/X11R6/lib -lXext -lX11 -lnsl -lm`. `-DUSEASM` is commented out, and the assembler
sources were never released (see README.asm).

### Sound

- The Makefile defines neither `SNDSERV` nor `SNDINTR`, so the stock build mixes sound
  **synchronously** in the main loop (`I_UpdateSound` + `I_SubmitSound` writing to `/dev/dsp`).
  This contradicts the ChangeLog, which says SNDSERV is the default. Trust the code.
- `-DSNDSERV` makes the game send commands to an external `./sndserver` process instead.
  Build it from [sndserv/](../sndserv/). If the game crashes, the sndserver can be left running
  and keep `/dev/dsp` open.
- There is no music. The MUS playback came from the proprietary DMX library and was removed.

## Problems on modern systems

These are the problems you're likely to hit with a current gcc/clang, a 64-bit OS, and a
modern X server:

| Problem | Where | Typical fix |
|---|---|---|
| `#include <errnos.h>` doesn't exist | i_video.c | Use `<errno.h>` |
| `extern int errno;` conflicts with glibc's thread-local errno | i_sound.c | Delete the line and include `<errno.h>` |
| Pointer ↔ `int` casts, and structs that assume 4-byte pointers | z_zone.c (`memblock_t`), r_data.c, p_saveg.c, w_wad.c, info/action pointers | Use `intptr_t`, fix struct padding, and pack on-disk structs explicitly |
| `-lnsl` missing | Makefile | Remove it (not needed on Linux) |
| X11 needs an 8-bit PseudoColor visual | i_video.c | Run under `Xephyr :1 -screen 640x400x8`, or convert to a 32-bit framebuffer with a palette lookup in `I_FinishUpdate` |
| `/dev/dsp` (OSS) is missing | i_sound.c, sndserv | Use `padsp`/`aoss`, or replace the `i_sound` backend (SDL, ALSA, ...) |
| `-m32` as a shortcut | — | Build 32-bit (`CFLAGS += -m32`, needs 32-bit X11 libs) to avoid most pointer-size bugs |

Record every fix that DOOMie applies in [fork-notes.md](fork-notes.md).

## Porting to a new platform

Only the `i_*` files talk to the OS. A port implements:

| File | Must provide |
|---|---|
| i_system.c | `I_ZoneBase`, `I_GetTime` (tics since start, at 35 Hz), `I_Quit`, `I_Error`, `I_BaseTiccmd`, `I_AllocLow` |
| i_video.c | `I_InitGraphics`, `I_SetPalette`, `I_FinishUpdate` (blit `screens[0]`), `I_StartTic` (pump events → `D_PostEvent`), `I_ReadScreen`, `I_ShutdownGraphics` |
| i_sound.c | `I_InitSound`, `I_StartSound`, `I_StopSound`, `I_SoundIsPlaying`, `I_UpdateSoundParams`, `I_UpdateSound`, `I_SubmitSound`, music stubs |
| i_net.c | `I_InitNetwork` (fill in `doomcom`), `I_NetCmd` |

Keep the framebuffer as 8-bit indexed (`screens[0]`, 320×200) and convert it to the host format
in `I_FinishUpdate`. Then the renderer doesn't need to change.

## Running

```sh
export DOOMWADDIR=/path/to/wads
./linux/linuxxdoom [-warp 1 1] [-skill 4] [-file pwad.wad] [-playdemo demo1] [-timedemo demo1]
```

Other useful flags: `-devparm`, `-nomonsters`, `-respawn`, `-fast`, `-deathmatch`/`-altdeath`,
`-record`, `-loadgame N`, `-config file`, `-turbo N`, `-net ...`, `-port N`. Video flags:
`-disp`, `-geom`, `-grabmouse`, and `-2`/`-3`/`-4` for scaling.
Config is stored in `$HOME/.doomrc`. The path is set in `IdentifyVersion` ([d_main.c](../linuxdoom-1.10/d_main.c)), and the file is read and written by `M_LoadDefaults`/`M_SaveDefaults` ([m_misc.c](../linuxdoom-1.10/m_misc.c)).
