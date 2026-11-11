# NHL 21 PS4 Port for Windows

This project runs the **PS4 version** of *EA SPORTS NHL 21* natively on Windows. It isn't an emulator
running a whole console. It loads the game's own x86-64 code directly, gives it Windows versions of
the PS4 system libraries it calls, and translates its graphics to Vulkan.

NHL 21 was the last NHL on EA's Ignite engine, the same engine as NHL 19 and UFC 3, so this port
starts from the [NHL 19 port](https://github.com/thegh0sts/NHL-19-PS4-PORT-WINDOWS) (itself a fork of
[GNSB's UFC 3 port](https://github.com/GNSB/UFC-3-PS4-PORT-LINUX-WINDOWS)) and inherits its fixes.

> [!IMPORTANT]
> This project contains **no game files**. You need your own PS4 copy of the game (**CUSA18182**,
> with update 1.10), dumped from your own console. Game files, and files made from them such as
> `boot-linked.bin`, must not be shared.

> [!WARNING]
> **v0.1.0 is the first test release.** The game boots, plays Play Now matches and creates its mode
> saves, but most of it hasn't been tested yet. Expect bugs and crashes, and back up your saves. If
> you hit a bug, please report it **with your log and saves**: see
> [Reporting problems](#reporting-problems).

<!-- Add a screenshot here, e.g. ![NHL 21 in game](docs/screenshot.png) -->

## Status

The game boots through its menus and team select into Play Now matches, with the arena, players and
crowd drawn at about 58-60 FPS on a modern PC. Game modes create their saves without a storage
warning. Everything else is still to be checked.

| Area | State |
|---|---|
| Boot, menus | Working |
| Team select | Working (3D players drawn) |
| Play Now matches | Loads into the match and plays; not tested at length |
| Game-mode saves (Be a GM, Tournaments...) | Created as on a PS4, without the low-storage warning |
| Other modes (Franchise, Be a Pro, HUT, World of Chel offline...) | Not tested yet |
| Graphics details (jerseys, beards, ice, menus' hub tiles) | Not checked yet: NHL 19's settings are used |
| Frame rate | About 58-60 FPS in menus, team select and the match (the port keeps 60) |
| Controller | Detected (Xbox controller tested) |
| Local multiplayer (several controllers) | Not yet |
| Online play | Not available (see [Online](#online)) |
| Launcher (`NHL21.exe`) | Working |
| Intel CPUs | Working: the game's AMD-only SSE4a instructions are emulated |

## Features

- **Launcher** (`NHL21.exe`) that sets everything up and starts the game:
  - checks your game folder, and merges a base game and update dumped separately
  - game language, controller for player 1, player name
  - video: output 720p or 1080p, window or screen size, FSR 1.0 upscaling with sharpening, FPS
    limit, presentation mode, fullscreen, FPS counter, PS4 Pro mode
  - developer switches (frame statistics, Vulkan validation, shader cache, logs)
  - a Steam shortcut, and `--play` to start without the window
- **In-game overlay** (Insert, or L3+R3 on a controller) to change settings while you play
- **Hold ○ for 2 seconds** on any screen to quit: "Quit NHL 21?", ✕ yes, ○ no
- The Ignite graphics settings that make NHL 19 draw correctly, applied to NHL 21

### What carries over from the NHL 19 port

- **Ignite graphics settings** (precise readbacks, linear readbacks, write windows off, background
  pre-upload on)
- **60 FPS throughout**: the game's requests for 30 FPS (team select, matches) are kept at 60
- **Red-zone protection**: the game's short-lived stack data is kept out of reach of Windows'
  exception handling (see [How it works](#how-it-works))
- **Save sizes that add up**, so the game doesn't think its storage is full
- **Network mode without a server**, which the game needs to get past its start-up offline
- **System functions added for NHL 24** that NHL 21's newer SDK (7.50) also calls: install-chunk
  and add-on queries, offline PSN answers, asynchronous file reads, POSIX read/write locks and a
  few kernel calls

## Requirements

- Windows 10 or 11, 64-bit
- A CPU from Intel Haswell (4th generation Core) or newer, or any AMD Ryzen
- A GPU with Vulkan 1.3 support and an up-to-date driver
- Your PS4 dump of *NHL 21* with update **1.10**

### Supported version

| Serial | Region | Update | Status |
|---|---|---|---|
| **CUSA18182** | Europe (EP0006) | 01.10 | Plays |

The port recognises the game by a fingerprint of its executable, so other regions or updates are
refused until they're added. Reports with a dump's serial and version are welcome: you can find
them in its `sce_sys\param.sfo` (`TITLE_ID`, `APP_VER`).

## Getting started

1. Download `NHL21Native-windows-x86_64.zip` from the
   [releases](https://github.com/thegh0sts/NHL-21-PS4-PORT/releases) and unzip it into a folder of its
   own, outside OneDrive (e.g. `C:\Games\NHL21Native`).
2. Run **`NHL21.exe`**, pick your game folder with **Browse...** (`CUSA18182`; a `CUSA18182-patch`
   folder beside it is merged in automatically, with hard links that take no extra space), and
   press **Play**.
3. The first start prepares the game, which takes a minute or so. The first matches build the
   shader cache, so the first load into a match can sit on a black screen for a minute or two.

### Where your files are

Logs go to the `logs` folder beside `NHL21.exe` (or to `%LOCALAPPDATA%\NHL21Native\logs` if that
folder can't be written, for example under Program Files; the launcher's Log folder setting picks
another). Settings, saves and the shader cache live in `%LOCALAPPDATA%\NHL21Native` (paste that
into the Explorer address bar). Set `BB_DATA_DIR=.` before starting the launcher to keep everything
beside the program instead.

### Updating

Unzip the new release over the old one, or into a new folder. Your settings and saves stay in
`%LOCALAPPDATA%\NHL21Native`.

## Online

The port doesn't connect to EA's servers. As in the NHL 19 port, the game starts in a network mode
with no server behind it (`BB_ONLINE_SERVER=127.0.0.1`), so it behaves like a PS4 that can't reach
EA, and plays offline.

## Building from source

The port builds natively on Windows with [MSYS2](https://www.msys2.org/)'s **CLANG64** environment.

1. Install MSYS2, open the **MSYS2 CLANG64** shell and install the tools and libraries (press Enter
   at each group prompt to take all of it):
   ```sh
   pacman -S --needed git mingw-w64-clang-x86_64-{toolchain,cmake,ninja,pkgconf,python,glslang,vulkan-devel,vulkan-memory-allocator,fmt,boost,robin-map,xxhash,sdl3,zydis}
   ```
2. Clone and build:
   ```sh
   git clone --recursive https://github.com/thegh0sts/NHL-21-PS4-PORT.git
   cd NHL-21-PS4-PORT
   bash scripts/build-msys2.sh
   ```
   The first build also builds a few pinned libraries MSYS2 doesn't package (magic_enum, miniz,
   VulkanMemoryAllocator, robin-map, xbyak). Programs and their DLLs go to `out/windows/bin`.
   `BB_LTO=ON` gives a slower build and a faster game.
3. Either package it (`bash scripts/package-msys2.sh` makes `dist/NHL21Native-windows-x86_64.zip`
   with `NHL21.exe`), or run it straight from the repo: put your game dump in `nhl21\` inside the
   repo (it's in `.gitignore`, so it's never committed), then
   ```sh
   bash scripts/run-msys2.sh nhl21
   BB_SKIP_PREPARE=1 bash scripts/run-msys2.sh nhl21   # later runs: skip the preparation
   ```
   The whole log goes to `out/run.txt` as well as the terminal.

If a build stops with `error: patch failed: src/ffx_vk_fsr4_v07.c`, reset that library and build
again: `git -C gpu/third_party/fsr-vulkan reset --hard`.

### Versions and releases

The version comes from git: `git describe --tags`, so a build of the commit tagged `v0.1.0` is
version `v0.1.0`, and later commits read like `v0.1.0-3-g2abb064`. A `-dirty` ending means the build
had uncommitted changes. To make a release, tag the commit, build and package, then upload. With the
[GitHub CLI](https://cli.github.com/):

```sh
git tag v0.1.0 && git push origin v0.1.0
bash scripts/build-msys2.sh && bash scripts/package-msys2.sh
gh release create v0.1.0 dist/NHL21Native-windows-x86_64.zip --title "NHL 21 port v0.1.0" --notes-file docs/release-notes/v0.1.0.md
```

### Settings for testing

`run-msys2.sh` and the launcher set NHL 21's defaults (for now the same as NHL 19's). Any of them
can be changed for one run by putting it in front of the command, for example
`BB_FLIP_RATE=-1 BB_SKIP_PREPARE=1 bash scripts/run-msys2.sh nhl21`.

| Variable | Default | Effect |
|---|---|---|
| `BB_READBACKS` / `BB_READBACK_LINEAR` | 2 / 1 | Reads of GPU-written memory. In NHL 19, 1 leaves a streak from bearded players' chins, and without the linear ones the players and rink render black |
| `BB_WRITE_WINDOW` / `BB_WRITE_STEP` | 0 / 0 | UFC 3's write handling. In NHL 19, on makes the jerseys shake |
| `BB_PREUPLOAD` | 1 | Background upload of rewritten memory to the GPU |
| `BB_FLIP_RATE` | 0 | 0: 60 FPS throughout. `-1`: the game's own choice (30 FPS in matches, as on a PS4) |
| `BB_ONLINE_SERVER` | 127.0.0.1 | The network mode without a server. Empty (`BB_ONLINE_SERVER=`) turns it off |
| `BB_REDZONE_SHIFT` | on | Red-zone protection. `0` turns it off, for comparison |
| `BB_HOLD_CIRCLE_QUIT` | 1 | Holding ○ for 2 seconds asks "Quit NHL 21?". `0` turns it off |
| `BB_SSE4A_LOG=1` | off | Logs each emulated AMD-only instruction |
| `BB_LOG_UNRESOLVED=1` | off | Logs every unhandled system call (very large logs) |
| `BB_FRAME_STATS=1` | off | Frame timing in the log |

`python3 tools/module_imports.py <module.prx>` lists what a game module needs from the system and
how much of it the port already provides.

## How it works

1. **Preparation** (`scripts/prepare.py`, `link_libc.py`, `link_modules.py`) reads the game's
   `eboot.bin` and links it with the PS4 libraries the game ships (`libc.prx`, `libSceFios2.prx`)
   into one image, `boot-linked.bin`. `game_check.py` identifies the game by a fingerprint of its
   executable (profile `nhl21`), `gen_import_names.py` names the system functions it imports, and
   `content_profile.py` records its content details. The launcher runs these on each start.
2. **The loader** (`sacredfire-game.exe`, `src/`) maps that image into memory and runs the game's
   code natively. Each call into a PS4 system library (kernel, threads, files, saves, pads, audio,
   video output, network and so on) goes to a Windows implementation in `src/runtime_*.c`.
3. **Red-zone protection:** PS4 code may keep data in the 128 bytes just below its stack pointer.
   Windows writes its exception information there whenever an instruction faults, and the port's
   GPU memory tracking makes the game fault often, on purpose. At start-up the loader moves the
   game's red-zone accesses 64 KiB lower, out of reach (about 11,600 in NHL 21), leaving
   stack-switching code alone.
4. **AMD-only instructions:** the PS4's CPU has AMD's SSE4a extension, which Intel CPUs lack. When
   the game uses one, the loader catches the error and carries out the instruction itself
   (`src/platform/sse4a.inc`).
5. **Graphics** go through a Vulkan backend based on [shadPS4](https://github.com/shadps4-emu/shadPS4)'s
   `video_core` (`gpu/`), which translates the game's GNM command buffers and shaders to Vulkan.

## Reporting problems

The port is at a very early stage, and bug reports are the fastest way to get problems fixed. Open
an issue with:

- your CPU, GPU and driver version, and Windows version
- what happened, in which mode and where (menu, face-off, intermission...), and what you were doing
- the log of that session: the newest file in the `logs` folder beside `NHL21.exe` (or
  `out/run.txt` when running from the repo)
- **your saves**, so the bug can be reproduced: zip the `user` folder in
  `%LOCALAPPDATA%\NHL21Native` (or in the repo); the `cache` folder inside can be left out, it's
  only the shader cache

Please don't attach game files or anything from the `out` folder other than `run.txt`: they're made
from the game and can't be shared. Saves are fine to share: they hold only your progress and settings.

## Version history

### v0.1.0 (2026-11-11)

First test release of the NHL 21 port.

**Game**
- Boots through the menus and team select into Play Now matches at about 58-60 FPS
- Game modes create their saves as on a PS4, without a low-storage warning (Be a GM tested)
- **Hold ○ for 2 seconds to quit:** a "Quit NHL 21?" box appears on any screen; ✕ (or Enter) quits,
  ○ (or Esc) goes back to the game. A short press of ○ still goes back a screen as usual.
  `BB_HOLD_CIRCLE_QUIT=0` turns it off

**Launcher**
- `NHL21.exe`: checks for NHL 21 CUSA18182 with the 1.10 update (merging a separate update folder
  automatically), and starts it with NHL 21's settings
- Settings and saves in `%LOCALAPPDATA%\NHL21Native`; logs in a `logs` folder beside `NHL21.exe`

**Under the hood**
- Started from the NHL 19 port (v0.2.1): red-zone protection, precise readbacks, 60 FPS throughout,
  realistic save sizes, the depth/stencil crash fix from NHL 19's Be a GM
- `nhl21` profile: NHL 21 CUSA18182 (EU) 1.10 recognised by a fingerprint of its executable
- System functions from the NHL 24 port for NHL 21's SDK 7.50

**Known issues**
- Most modes, and matches played to the end, are untested
- Graphics details (jerseys, beards, hub tiles) not checked yet
- One local player only; no online play; only the European 1.10 version is recognised

## Credits

This port is built on the work of:

- The **[NHL 19 port](https://github.com/thegh0sts/NHL-19-PS4-PORT-WINDOWS)**, which this project
  starts from
- **[GNSB](https://github.com/GNSB/UFC-3-PS4-PORT-LINUX-WINDOWS)**'s UFC 3 port, which the NHL 19
  port is a fork of
- **SacredFire** (T3CHNOLOG1C): the loader and launcher the UFC 3 port grew from
- **bbport** (deadinside28): the original native PS4 loader approach
- **[shadPS4](https://github.com/shadps4-emu/shadPS4)**: the Vulkan graphics backend and much PS4 knowledge
- SDL3, Dear ImGui, Vulkan, Zydis, xxHash, fmt, sirit, LibAtrac9, miniz, AMD FidelityFX FSR and
  others: see [`licenses/`](licenses/)

## Licence

This project is free software, licensed under the **GNU General Public License v2.0 or later**: see
[`LICENSE`](LICENSE). Third-party components keep their own licences, listed in [`licenses/`](licenses/).

## Legal

This is a fan project and is not affiliated with, endorsed by or connected to Electronic Arts, EA
SPORTS, the National Hockey League, the NHL Players' Association or Sony Interactive Entertainment.
*NHL*, *EA SPORTS* and *PlayStation* are trademarks of their respective owners. The project contains
no copyrighted game code or assets; it only runs a copy of the game you dumped yourself from a
console you own.
