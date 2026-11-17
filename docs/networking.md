# Networking and Demos

DOOM uses **peer-to-peer lockstep** networking. Players send each other `ticcmd_t`s and never
send game state. Every node runs the same simulation, so any nondeterminism causes a desync.

## Engine side ([d_net.c](../linuxdoom-1.10/d_net.c), [d_net.h](../linuxdoom-1.10/d_net.h))

- `doomcom_t` is the shared communication block between the engine and the network driver.
  It holds `id == DOOMCOM_ID (0x12345678)`, `command` (`CMD_SEND`/`CMD_GET`), `remotenode`,
  `numnodes`, `ticdup`, `extratics`, `consoleplayer`, `numplayers`, and `data` (a `doomdata_t` packet).
- `doomdata_t` holds a checksum with flags (`NCMD_EXIT`, `NCMD_RETRANSMIT`, `NCMD_SETUP`,
  `NCMD_KILL`), the start tic, the number of tics, and up to `BACKUPTICS` (12) ticcmds.
- `NetUpdate` builds local ticcmds (`G_BuildTiccmd`) and sends them to every node. It also
  receives packets, and handles retransmit requests and player quits.
- `TryRunTics` waits until every player's ticcmd for a tic has arrived, then runs it.
  `-dup` sets ticdup (send each command for several tics) and `-extratic` adds redundancy.
- `consistancy[][]` holds a per-tic checksum of player positions, sent with each ticcmd. If they
  don't match, the game reports "consistency failure", which means the nodes have desynced.
- `MAXNETNODES` is 8. `MAXPLAYERS` is 4.

## Linux driver ([i_net.c](../linuxdoom-1.10/i_net.c))

- UDP sockets. The default port is `IPPORT_USERRESERVED + 0x1d` (5029). Use `-port N` to change it.
- `-net <consoleplayer> <host> <host> ...` (consoleplayer counts from 1). Without `-net`, the
  game runs as a single node.
- Implements `I_InitNetwork` and `I_NetCmd`, and byte-swaps the packet fields.

## DOS drivers ([ipx/](../ipx/), [sersrc/](../sersrc/))

These are standalone DOS programs (`ipxsetup.exe`, `sersetup.exe`). Each one:

1. Negotiates with the peers. IPX broadcasts setup packets. The serial driver handshakes over
   the COM port or modem (`PORT.C`).
2. Fills in a `doomcom_t` and hooks a free software interrupt (`doomcom.intnum`).
3. Starts `doom.exe -net <address of doomcom>`. DOOM triggers the interrupt to send and receive.

They target 16-bit DOS compilers and use `dos.h` and `setvect`. They do not build or run on
Linux and are kept for reference. `*_FRCH.H` and `*STR.H` hold the French and English strings.

## Demos ([g_game.c](../linuxdoom-1.10/g_game.c))

- `-record name` writes `name.lmp`: a header (version, skill, episode, map, deathmatch,
  respawn, fast, nomonsters, consoleplayer, playeringame[4]) followed by 4 bytes per ticcmd
  per player (`G_WriteDemoTiccmd`).
- `-playdemo name` plays a demo. `-timedemo name` plays it without the timer and prints the fps.
- Demos are the **regression tests** for the simulation. If built-in demos (`demo1`–`demo3` in
  the IWAD) desync after a change, the change altered game logic.
- Upstream removed the "Demo is from a different game version" exit. See the ChangeLog.
