# CLAUDE.md

Guidance for Claude Code (and humans) working in **DOOMie**, a fork of id Software's
DOOM source release (Linux DOOM 1.10, 1997-12-23, GPL-2.0 — see [LICENSE.TXT](LICENSE.TXT)).

The deeper knowledge base lives in [docs/](docs/README.md). Read the relevant topic doc
before changing a subsystem.

## Repository layout

| Path | What it is | Builds on |
|---|---|---|
| [linuxdoom-1.10/](linuxdoom-1.10/) | The game engine (X11/Linux). ~45k lines of C. | Linux + X11 (gcc) |
| [sndserv/](sndserv/) | Standalone `sndserver` process for Linux audio (OSS `/dev/dsp`). | Linux |
| [ipx/](ipx/) | DOS IPX network driver (`ipxsetup`). | DOS (Borland/Watcom era) |
| [sersrc/](sersrc/) | DOS serial/modem network driver (`sersetup`). | DOS |
| [README.TXT](README.TXT) | Carmack's release notes and suggested projects. | — |

`ipx/` and `sersrc/` are separate DOS programs. They talk to DOOM through the shared
`doomcom_t` struct and a software interrupt. They are not linked into the Linux binary.

## Build

```sh
cd linuxdoom-1.10 && mkdir -p linux && make      # -> linux/linuxxdoom
cd sndserv && make                               # -> sndserver (optional)
```

- Flags: `-DNORMALUNIX -DLINUX`, libs `-lXext -lX11 -lnsl -lm`.
- You need an IWAD at runtime (`doom1.wad`, `doom.wad`, `doomu.wad`, `doom2.wad`,
  `plutonia.wad`, `tnt.wad`). DOOM looks for it in `$DOOMWADDIR`, or in `.` if that is unset.
  WADs are **not** in the repo and must never be committed.
- The X server must provide an **8-bit PseudoColor** visual. On modern systems, run under
  `Xephyr -screen 320x200x8` or similar.
- The code is 1997, 32-bit C. It does not build cleanly on modern 64-bit toolchains without
  porting work. See [docs/building-and-porting.md](docs/building-and-porting.md).

There are no tests. To verify a change, play a demo (`-playdemo demo1`) or run
`-timedemo demo1` and compare the result. Demo playback only stays in sync if the simulation
is still bit-for-bit deterministic.

## Coding conventions (match the existing code)

- C89, no C++. Keep the K&R style: 4-space indent with tabs mixed in, `void Foo (void)`
  with a space before the paren, and `//` comments (the original style).
- **Module prefixes** tell you where a symbol lives (`R_` render, `P_` playsim, `G_` game
  flow, etc.). See [docs/architecture.md](docs/architecture.md). New code follows the same
  prefix-per-file scheme.
- Fixed-point math: `fixed_t` is 16.16 (`FRACBITS 16`), using `FixedMul`/`FixedDiv`. Angles are
  BAM (`angle_t`, `ANG90 = 0x40000000`). Trig uses `finesine`/`finecosine` LUTs indexed by
  `angle >> ANGLETOFINESHIFT`.
- Allocate memory with `Z_Malloc(size, tag, user)` from the zone, not `malloc`. Choose the
  purge tag carefully. See [docs/data-and-memory.md](docs/data-and-memory.md).
- Randomness in game logic **must** use `P_Random()`. `M_Random()` is for things that don't
  affect the simulation, like menus and visuals. Mixing these up breaks demos and netgames.
- `info.c`/`info.h` and `sounds.c`/`sounds.h` are generated tables, originally produced by
  id's `multigen` tool. Edit them by hand, carefully, and keep the enums and arrays in sync.
- Files use CRLF line endings in `ipx/`, `sersrc/`, and much of `linuxdoom-1.10/`. Keep each
  file's existing line endings.

## Invariants — do not break

1. **Determinism.** Game logic runs at `TICRATE` (35 Hz) from `ticcmd_t` inputs only. No wall
   clock, floats, uninitialised reads, or `M_Random` inside `P_*`/`G_Ticker` paths.
2. **Static limits** (`MAXVISPLANES 128`, `MAXDRAWSEGS 256`, `MAXVISSPRITES 128`,
   `MAXINTERCEPTS 128`, `MAXPLATS 30`, ...). Overflowing them crashes the game or corrupts
   memory. If you raise one, document it in [docs/fork-notes.md](docs/fork-notes.md).
3. **Shareware/commercial gating** in `IdentifyVersion` and `W_*` must stay (see `README.b`).
4. **License.** Everything here is GPL-2.0. Keep the copyright headers.

## Git workflow

Conventional Commits 1.0.0 always. Flow: `<type>/<name>` from `main` → merge to `testing` →
once it works, merge to `release` (with a CHANGELOG entry) → `release` into `main`, tag `vX.Y.Z`,
GitHub release. All git actions go through the `git-workflow` agent (`/flow <step>`).

- **Release assets:** none yet. The GitHub release ships the auto-generated source archives.
  If a build artifact is ever added, put the command that produces it here.
- `master` mirrors the upstream id Software default branch and is not part of the flow.

## Documentation

- Every feature gets an entry in [CHANGELOG.md](CHANGELOG.md) when it's promoted to `release`.
- Fork-specific technical decisions (raised limits, porting fixes, demo/save compatibility
  breaks) also go in [docs/fork-notes.md](docs/fork-notes.md).
