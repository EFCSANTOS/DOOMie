# Vision

**DOOMie** (working title) is a round-based co-op zombie shooter with the look and speed of
1993 DOOM, built on the GPL DOOM engine source. It's made for a small group of friends to
play together.

> In one line: **DOOM's look and speed + round-based zombie survival, played co-op with friends.**

## What it should feel like

- **Fast.** Movement and combat run at DOOM speed: instant acceleration, no sprint stamina, and
  constant movement. If a change slows the game down, that counts against it.
- **DOOM-esque look.** Low-resolution, palette-based software rendering, sprite enemies, chunky
  pixels, and light that fades with distance. The style is DOOM's, but the art is our own (see
  [Content and IP](#content-and-ip)).
- **Modern controls.** Free mouse look and aiming like a modern shooter: you hit what the crosshair
  is on, with no auto-aim locked to one axis. Levels have real height variation.
- **Round-based survival.** Zombies come in waves that get harder each round. You earn points for
  hits and kills, spend them on weapons and on opening up the map, and survive as long as
  possible with your friends.

## Goals

1. **The game mode:** endless rounds of zombies, with a points economy, wall-bought weapons, doors
   and areas you buy to open, rebuildable barricades, and downs and revives in co-op.
2. **One hand-made map** based on a real abandoned location, built from reference photos.
3. **Co-op multiplayer:** LAN co-op first (2–4 players, host and join). Later, hosting over the
   internet and inviting friends.
4. **Windows first, portable always:** it builds and runs on Windows now. All OS-specific code
   stays in the `i_*` platform layer so that Linux and Android ports stay possible.

## Non-goals

- **Compatibility with the original DOOM.** Original IWADs, PWADs, demos and save games don't
  need to work. Gameplay, data formats and limits can all change freely.
- **Recreating DOOM or any zombie mode from another game.** We borrow genre ideas (rounds, wall-buys,
  barricades) and invent our own content.
- **Deathmatch/competitive play, single-player campaigns, a level editor of our own.** Not planned.
- **A public release or community.** For now this is for me and my friends.
- **Keeping the DOS drivers going.** `ipx/` and `sersrc/` are frozen as reference.

## Rules that stay

- **Determinism.** Multiplayer uses DOOM's lockstep model, so the game simulation must still be
  deterministic: fixed tics, `P_Random` for anything in gameplay, and no wall clock or
  uninitialised data in game logic.
- **GPL-2.0.** The code stays GPL, and the copyright headers stay.
- **Platform separation.** No Windows (or SDL) calls outside the `i_*` files.

## Content and IP

- **No id Software or Activision assets.** No DOOM sprites, textures, sounds or music from the
  commercial or shareware WADs, and no names, characters, maps, perk names or music from
  *Call of Duty* zombies. We use our own art, or art with a licence that allows it
  (e.g. CC0/CC-BY, with credit in `CREDITS.md`).
- **Reference photos** of the real location live in the local `e/` folder, which is ignored by
  `.gitignore`. They're never committed, and the repo doesn't name or locate the place.
- The name "DOOMie" is fine for private use. Rename the project before any public release.

## How to judge a feature

A feature fits if it answers **yes** to most of these:

1. Does it keep the game **fast**?
2. Does it serve **round-based co-op zombie survival**, or the **look** of the game?
3. Does it keep the simulation **deterministic** (so netplay still works)?
4. Can it be built **without platform code outside `i_*`**?
5. Is all the content in it **ours or properly licensed**?

The decisions themselves are recorded in [roadmap.md](roadmap.md).
