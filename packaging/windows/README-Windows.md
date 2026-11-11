# NHL 21 native PC port — Windows

This archive runs **EA SPORTS NHL 21** (PlayStation 4, `CUSA18182`, Europe, update **1.10**) as a
native Windows program. It's built from the [NHL 21 port](https://github.com/thegh0sts/NHL-21-PS4-PORT)
and **contains no game files**: you need your own PS4 copy of the game, dumped from your own console.

This is an early test build: expect bugs, and back up your saves.

## Requirements

- Windows 10 or 11, 64-bit
- A CPU from Intel Haswell (4th generation Core) or newer, or any AMD Ryzen
- A GPU with Vulkan 1.3 support and an up-to-date driver
- The game: the base folder `CUSA18182` and the 1.10 update next to it, e.g.
  `PS4\CUSA18182` and `PS4\CUSA18182-patch` (also `-PATCH`, `-update`, `_patch`). They are
  merged automatically into `CUSA18182-merged` with hard links (no extra space on NTFS).
  A folder with the update already merged in works too.

## Starting

1. Unzip the archive into a folder of its own, outside OneDrive (e.g. `C:\Games\NHL21Native`).
2. Run **`NHL21.exe`**, pick the `CUSA18182` folder with **Browse...**, press **Play**.
3. The first start prepares the game (a minute or so), and the first matches build the shader
   cache, so the first load into a match can sit on a black screen for a minute or two.

Settings, saves and the shader cache live in `%LOCALAPPDATA%\NHL21Native`.

## In game

- **Insert**, or **L3+R3** on a controller: the overlay, to change settings while you play
- **Hold ○ (B on Xbox) for 2 seconds** on any screen: "Quit NHL 21?" — ✕ / A (or Enter) quits,
  ○ / B (or Esc) goes back to the game. A short press of ○ still goes back a screen as usual

## Online

The port doesn't connect to EA's servers; the game plays offline.

## Problems

Logs go to the `logs` folder beside `NHL21.exe` (or to `%LOCALAPPDATA%\NHL21Native\logs` if that
folder can't be written). If the game crashes, report it on the project's GitHub issues with the
newest `.log` from there and your saves (zip `%LOCALAPPDATA%\NHL21Native\user`, leaving out
`cache`). Please don't attach game files.

If Windows blocks the launcher, unblock the zip (Properties > Unblock) before extracting.

## Legal

A fan project, not affiliated with or endorsed by Sony Interactive Entertainment, Electronic Arts,
the NHL or the NHLPA. GPL-2.0-or-later; third-party licences are in `licenses\`.
