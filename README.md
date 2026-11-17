# DOOMie

A round-based **co-op zombie shooter** with the look and speed of classic DOOM, built on the
GPL [DOOM engine source](https://github.com/id-Software/DOOM). Made for playing with friends.

> **Status: early.** v0.0.1 contains only documentation. The engine is still the 1997 Linux
> release, and the Windows build is the first milestone.

## The idea

- **DOOM look and feel:** pixelated software rendering, sprite enemies, fast movement.
- **Modern shooter controls:** free mouse look, aiming at the crosshair, and maps with real height.
- **Zombie survival:** endless rounds, points for kills, buyable weapons and areas, barricades,
  and reviving teammates.
- **One map** based on a real abandoned place.
- **Co-op:** LAN first, then host-and-invite over the internet.
- **Windows first**, and kept portable for Linux and Android later.

More detail: [docs/vision.md](docs/vision.md) · plan: [docs/roadmap.md](docs/roadmap.md).

## Building

Windows build: not yet available (roadmap milestone M0, SDL2 + CMake).

The original Linux/X11 build still works the way it did in 1997. See
[docs/building-and-porting.md](docs/building-and-porting.md).

## Documentation

| | |
|---|---|
| [docs/vision.md](docs/vision.md) | What the game is, its goals, what's out of scope, and the content rules |
| [docs/roadmap.md](docs/roadmap.md) | Planned, maybe, and rejected features |
| [docs/README.md](docs/README.md) | Engine knowledge base (architecture, rendering, playsim, ...) |
| [CHANGELOG.md](CHANGELOG.md) | Everything that has shipped, version by version |
| [CLAUDE.md](CLAUDE.md) | Conventions and git workflow (for contributors and AI assistants) |
| [README.TXT](README.TXT) | John Carmack's original 1997 release notes |

## Contributing

This is a private project for me and my friends. It follows
[Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/) and the branch flow
`feature → testing → release → main`, described in [CLAUDE.md](CLAUDE.md#git-workflow).

## License

GPL-2.0. See [LICENSE.TXT](LICENSE.TXT). The engine is © id Software / ZeniMax Media.
This project includes no DOOM game assets.
