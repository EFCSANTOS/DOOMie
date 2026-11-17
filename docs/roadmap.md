# Roadmap

Features sorted into **Planned**, **Maybe** and **Rejected**. Check every idea against
[vision.md](vision.md#how-to-judge-a-feature) before it goes here. When a feature ships, move it
to *Done* and add it to [CHANGELOG.md](../CHANGELOG.md).

## Planned (in order)

### M0 — Builds and runs on Windows
- [ ] Replace the X11/OSS platform layer (`i_video`, `i_sound`, `i_system`, `i_net`) with
      **SDL2**. One layer then covers Windows now, and Linux and Android later.
- [ ] Make the engine 64-bit clean (pointer/`int` casts, `memblock_t`, struct layouts). See
      [building-and-porting.md](building-and-porting.md).
- [ ] A CMake build that works with MSVC and MinGW. Keep the old Makefile until SDL works on Linux.
- [ ] A test asset WAD of our own, so the game starts without any id IWAD.

### M1 — Modern controls
- [ ] Mouse look on both axes: first y-shearing in the software renderer (vertical look of about ±30°).
- [ ] Free aiming: hitscan and projectiles follow the view pitch. Remove vertical auto-aim.
- [ ] A crosshair, WASD/mouse bindings by default, and rebindable keys.
- [ ] Jump (and maybe crouch), so the height differences can be used.

### M2 — Content pipeline
- [ ] Our own base WAD: palette, sprites, textures, sounds, and fonts for the HUD and menus.
- [ ] Map editing with existing open tools (e.g. Ultimate Doom Builder / SLADE). We don't write our own editor.
- [ ] `CREDITS.md` for any third-party assets and their licences.

### M3 — Zombie round mode
- [ ] Round manager: spawn waves, zombie count/health/speed that grow with each round, and breaks between rounds.
- [ ] Zombie AI: walk to the nearest player, break barricades, and spawn at set spawn points.
- [ ] Points economy: points for hits and kills, and a HUD showing points and the round number.
- [ ] Buyable weapons on walls, doors and areas that open for points, and rebuildable window barricades.
- [ ] Co-op: downed state, reviving teammates, and game over when everyone is down.
- [ ] Our own weapons (pistol to start, then ~6 buyable weapons), plus an upgrade machine with our own name.

### M4 — The map
- [ ] Block out the real abandoned location from the reference photos (`e/`, local only).
- [ ] Make it playable: spawn rooms, buyable areas, zombie spawns, and barricaded windows.
- [ ] Detail and lighting pass in the DOOM-esque style.

### M5 — Co-op netplay
- [ ] LAN co-op for 2–4 players: host a game, and friends join by IP or by finding it on the LAN.
- [ ] Players can join in the lobby, before the game starts.
- [ ] Later: internet hosting and invites (needs port forwarding, a relay, or NAT punch-through).
      Lockstep networking is sensitive to latency, so friends in the same region come first.

### M6 — Ports
- [ ] Linux (SDL2, which should come almost free after M0).
- [ ] Android: SDL2, touch controls, and a performance check.

## Maybe (decide later)

- Higher internal resolution (e.g. 640×400). The HUD and menu graphics would need to scale.
- Rooms above rooms / 3D floors (true multi-level buildings). This needs big renderer changes.
- A hardware (OpenGL) renderer that keeps the pixelated look and allows real pitch.
- Music (via the SDL_mixer or OGG/MOD formats).
- A random weapon box, power-up drops, character upgrades (with our own names), and map secrets.
- More maps.

## Rejected

| Idea | Why |
|---|---|
| Running the original DOOM IWADs, demos and save games | Not a goal. It would hold back changes to gameplay and formats |
| Deathmatch / competitive modes | The game is about co-op survival |
| Using DOOM or Call of Duty assets, names or maps | IP. We use our own content (see vision.md) |
| Keeping the DOS IPX/serial drivers working | Replaced by SDL/UDP netplay. Kept only as reference |

## Done

_Nothing shipped yet beyond documentation (v0.0.1)._
