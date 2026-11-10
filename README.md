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
> **This port has only just started.** It boots and plays its first matches, but almost nothing has
> been tested yet, there's no release, and the launcher is still NHL 19's. Expect bugs and crashes,
> and back up your saves. If you hit a bug, please report it **with your saves**: see
> [Reporting problems](#reporting-problems).

<!-- Add a screenshot here, e.g. ![NHL 21 in game](docs/screenshot.png) -->

## Status

On its first runs the game booted through its menus and team select into a Play Now match, with the
arena, players and crowd drawn at about 58-60 FPS on a modern PC. Everything else is still to be
checked.

| Area | State |
|---|---|
| Boot, menus | Working |
| Team select | Working (3D players drawn) |
| Play Now matches | Loads into the match; gameplay not checked yet |
| Other modes (Franchise, Be a Pro, HUT, World of Chel offline...) | Not tested yet |
| Graphics details (jerseys, beards, ice, menus' hub tiles) | Not checked yet: NHL 19's settings are used |
| Frame rate | About 58-60 FPS in menus, team select and the match (the port keeps 60) |
| Saves | A profile is created on the first start; not tested further |
| Controller | Detected (Xbox controller tested); in-match control not checked yet |
| Local multiplayer (several controllers) | Not yet |
| Online play | Not available (see [Online](#online)) |
| Launcher (`NHL21.exe`) | Not yet: run from source (below) |
| Intel CPUs | Working: the game's AMD-only SSE4a instructions are emulated |

## What carries over from the NHL 19 port

- **Ignite graphics settings** that make NHL 19 draw correctly (precise readbacks, linear readbacks,
  write windows off, background pre-upload on), applied to NHL 21 by default
- **60 FPS throughout**: the game's requests for 30 FPS (team select, matches) are kept at 60
- **Red-zone protection**: the game's short-lived stack data is kept out of reach of Windows'
  exception handling (see [How it works](#how-it-works))
- **Network mode without a server**, which NHL 19 needs to get past its start-up offline
- **System functions added for NHL 24** that NHL 21's newer SDK (7.50) also calls: install-chunk
  and add-on queries, offline PSN answers, asynchronous file reads, POSIX read/write locks and a
  few kernel calls

## Requirements

- Windows 10 or 11, 64-bit
- A CPU from Intel Haswell (4th generation Core) or newer, or any AMD Ryzen
- A GPU with Vulkan 1.3 support and an up-to-date driver
- Your PS4 dump of *NHL 21* with update **1.10**
- For now, [MSYS2](https://www.msys2.org/) to build and run it (see [Building and running](#building-and-running))

### Supported version

| Serial | Region | Update | Status |
|---|---|---|---|
| **CUSA18182** | Europe (EP0006) | 01.10 | Boots into a match |

The port recognises the game by a fingerprint of its executable, so other regions or updates are
refused until they're added. Reports with a dump's serial and version are welcome: you can find
them in its `sce_sys\param.sfo` (`TITLE_ID`, `APP_VER`).

## Online

The port doesn't connect to EA's servers. As in the NHL 19 port, the game starts in a network mode
with no server behind it (`BB_ONLINE_SERVER=127.0.0.1`), so it behaves like a PS4 that can't reach
EA, and plays offline.

## Building and running

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
3. Put your game dump in `nhl21\` inside the repo (it's in `.gitignore`, so it's never committed),
   then run it:
   ```sh
   bash scripts/run-msys2.sh nhl21
   BB_SKIP_PREPARE=1 bash scripts/run-msys2.sh nhl21   # later runs: skip the preparation
   ```
   The whole log goes to `out/run.txt` as well as the terminal. The first matches build the shader
   cache, so the first load into a match can sit on a black screen for a minute or two.

If a build stops with `error: patch failed: src/ffx_vk_fsr4_v07.c`, reset that library and build
again: `git -C gpu/third_party/fsr-vulkan reset --hard`.

### Settings for testing

`run-msys2.sh` sets NHL 21's defaults (for now the same as NHL 19's). Any of them can be changed for
one run by putting it in front of the command, for example
`BB_FLIP_RATE=-1 BB_SKIP_PREPARE=1 bash scripts/run-msys2.sh nhl21`.

| Variable | Default | Effect |
|---|---|---|
| `BB_READBACKS` / `BB_READBACK_LINEAR` | 2 / 1 | Reads of GPU-written memory. In NHL 19, 1 leaves a streak from bearded players' chins, and without the linear ones the players and rink render black |
| `BB_WRITE_WINDOW` / `BB_WRITE_STEP` | 0 / 0 | UFC 3's write handling. In NHL 19, on makes the jerseys shake |
| `BB_PREUPLOAD` | 1 | Background upload of rewritten memory to the GPU |
| `BB_FLIP_RATE` | 0 | 0: 60 FPS throughout. `-1`: the game's own choice (30 FPS in matches, as on a PS4) |
| `BB_ONLINE_SERVER` | 127.0.0.1 | The network mode without a server. Empty (`BB_ONLINE_SERVER=`) turns it off |
| `BB_REDZONE_SHIFT` | on | Red-zone protection. `0` turns it off, for comparison |
| `BB_SSE4A_LOG=1` | off | Logs each emulated AMD-only instruction |
| `BB_LOG_UNRESOLVED=1` | off | Logs every unhandled system call (very large logs) |
| `BB_FRAME_STATS=1` | off | Frame timing in the log |

`python3 tools/module_imports.py <module.prx>` lists what a game module needs from the system and
how much of it the port already provides.

### Versions

The version comes from git: `git describe --tags`. There's no release yet; the first will be
`v0.1.0`, once the launcher is converted to NHL 21.

## How it works

1. **Preparation** (`scripts/prepare.py`, `link_libc.py`, `link_modules.py`) reads the game's
   `eboot.bin` and links it with the PS4 libraries the game ships (`libc.prx`, `libSceFios2.prx`)
   into one image, `boot-linked.bin`. `game_check.py` identifies the game by a fingerprint of its
   executable (profile `nhl21`), `gen_import_names.py` names the system functions it imports, and
   `content_profile.py` records its content details.
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
- the log of that session: `out/run.txt`
- **your saves**, so the bug can be reproduced: zip the `user` folder in the repo (the `cache`
  folder inside can be left out: it's only the shader cache) and attach it

Please don't attach game files or anything from the `out` folder other than `run.txt`: they're made
from the game and can't be shared. Saves are fine to share: they hold only your progress and settings.

## Development history

### Not released yet (2026-11-10)

- Started from the NHL 19 port (v0.2.0)
- `nhl21` profile: NHL 21 CUSA18182 (EU) 1.10 recognised, with NHL 19's settings
- System functions from the NHL 24 port added for NHL 21's SDK 7.50
- First runs: boots through the menus and team select into a Play Now match at about 58-60 FPS

**To do**
- Check gameplay, graphics (jerseys, beards, ice, hub tiles) and other modes
- Convert the launcher to NHL 21 (`NHL21.exe`, data in `%LOCALAPPDATA%\NHL21Native`)
- First release, v0.1.0

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
