# Pit-Fighter Recomp

A native recompilation of **Pit-Fighter** (USA), built on
[snesrecomp](https://github.com/RetroPortingToolKit/snesrecomp).

Pit-Fighter is a 1990 fighting game developed and published by Atari Games for arcades. The Japanese release was published by Konami.

_Description from [Wikipedia](https://en.wikipedia.org/wiki/Pit-Fighter), CC BY-SA 4.0._

> **A compatible ROM is required.** This project does not provide or download
> game ROM data. You are responsible for ensuring you have the right to use
> the ROM you supply. Recompiled C is generated locally and is never committed.

## Status

The native executable builds and runs. Initial runtime testing has confirmed
working visuals, audio, and gameplay, including the Player Two option. The
project remains under development; broader compatibility testing, code
coverage, and port refinement are ongoing.

## ROM identity

| | |
|---|---|
| File | `Pit-Fighter (USA).sfc` |
| Publisher | THQ |
| Developer | Tengen |
| Year | 1992 |
| Mapping | lorom |
| Region | USA (North America) |
| Coprocessor | none |
| CRC32 | `9fab69fc` |
| SHA-256 | `c2a1a66648a0a0bfe2f201cf4f926d138e410fbf85ecf436ccb9aac70c0df3de` |

`tools/regen.sh` refuses to run against anything else, so a mismatched
revision fails immediately instead of producing subtly wrong output.

## Build

```sh
git submodule update --init --recursive
bash tools/regen.sh --rom /path/to/Pit-Fighter (USA).sfc
cmake -S . -B build -DCMAKE_BUILD_TYPE=Release
cmake --build build -j
```

The ROM does not have to live in the repository — keeping it on your own
drive is the better habit, and `SNESRECOMP_ROM` sets the path once for a
shell. Without either, `tools/regen.sh` finds `Pit-Fighter (USA).sfc` at the repo root,
then the path the setup wizard recorded in `rom.cfg`; `.gitignore` blocks
both from ever being committed.

`tools/regen.sh` verifies the ROM, generates `src/gen/*.c`, and re-syncs
`recomp/funcs.h`. Re-run it whenever you change anything under `recomp/`.

## Run

```sh
./build/PitFighterSNESRecomp                     # launcher picks the ROM
./build/PitFighterSNESRecomp /path/to/Pit-Fighter (USA).sfc # or name it and skip the launcher
```

With no ROM on the command line the build opens the recomp-ui launcher: a ROM
picker with a verification badge, plus display / audio / input settings. Your
choice is cached in `rom.cfg` beside the executable, so the next launch opens
on it and "Skip launcher on boot" makes it immediate. If the launcher is not
built in (`--no-recomp-ui`), the host falls back to a native file picker.

Either way the ROM is checked against the digests above before anything boots
— a dump that could not have produced this build is refused at the door rather
than mis-executing ten frames in.

## Layout

| Path | What lives there |
|---|---|
| `recomp/` | Analysis input: `bank*.cfg`, `symbols.toml`, generated `funcs.h` |
| `rom_identity.txt` | ROM digests — read by the build, `tools/regen.sh` and CI |
| `src/` | Host code you own: `main.c`, `game_rtl.c` |
| `src/gen/` | Generated C. Never committed — regenerate locally |
| `snesrecomp/` | Framework submodule (owns `lib/recomp-net`, `lib/retcomm-rbengine`) |
| `tools/` | `regen.sh` — the ROM → C pipeline |
| `scripts/` | `package_release.sh` — player-facing zip |
| `framework_pins.txt` | Exact framework commits this project was scaffolded against |

## Porting from here

The initial executable is running. The next development steps are:

1. **Expand coverage.** Exercise more of the game's modes, transitions,
   characters, and gameplay scenarios to identify untested code paths.
2. **Resolve dispatch misses.** After every run, deal with unresolved
   indirect targets before anything else. They can cause execution to diverge.
3. **Name and analyze routines.** Add entries to `recomp/symbols.toml`
   as routines are identified, then re-run `tools/regen.sh`. Set
   `emit = true` to promote proven routines into ahead-of-time analysis.
4. **Improve performance carefully.** Promote only verified code paths.
   Keep uncertain or hardware-sensitive routines interpreted.
5. **Preserve correctness.** Never synthesize a result to get past uncovered
   code, and never edit `src/gen/` by hand. Fix the configuration or framework
   and regenerate.

## Multiplayer

The launcher exposes a Player Two option. Full two-player gameplay has not
yet been validated for this title.

The framework includes netplay support: delay-sync is the default, and
rollback can be selected with `SNES_NET_MODE=rollback`. These modes have
not yet been validated with Pit-Fighter. See `snesrecomp/docs/RECOMP_NET.md`
and `snesrecomp/docs/ROLLBACK.md`.

## License

This project's own source is under the license in `LICENSE`. The framework
carries its own terms — see `snesrecomp/LICENSE` and
`snesrecomp/THIRD_PARTY_ATTRIBUTION.md`. Neither covers the game data, which
is not distributed here.
