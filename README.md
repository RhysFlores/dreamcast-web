# Dreamcast Web

A Sega Dreamcast emulator that runs in the browser: the
[flycast-wasm](https://github.com/nasomers/flycast-wasm) core (Flycast compiled to
WebAssembly with an SH4→WASM JIT) running inside [EmulatorJS](https://github.com/EmulatorJS/EmulatorJS).

No games are included. Bring your own dumps. A BIOS is optional: without one, Flycast's
built-in HLE BIOS is used (Sonic Adventure 2 boots to its title screen this way).

**Play online:** https://rhysflores.github.io/dreamcast-web/ (pick your game, and optionally
BIOS files; they're loaded in your browser and never uploaded).

Works on phones and tablets too: touch devices get an on-screen Dreamcast pad (analog
stick, D-pad, A/B/X/Y, L/R triggers, Start), and Bluetooth controllers work. Big disc
images need a lot of memory, so on mobile a single `.chd` file works best.

## Run it locally

```sh
python3 serve.py
```

This opens `http://localhost:8080`. The server sends the cross-origin isolation headers
the core needs, so opening `index.html` directly won't work.

You need:

- **Optionally, a BIOS**: `dc_boot.bin` and `dc_flash.bin` from your own Dreamcast, placed in
  `bios/` (or picked on the page). Without them the HLE BIOS is used.
- **A game**: `.gdi`, `.cdi`, `.chd`, or a Redump-style `.cue`/`.bin` set (converted to
  GDI on the fly), or a `.zip`/`.7z` containing one.

### Auto-run a game

Copy `config.example.json` to `config.json` (git-ignored) and point `game` at your image,
or pass it on the command line:

```sh
python3 serve.py --game "~/Games/Sonic Adventure 2/sa2.cue"
```

The page then loads the game straight from disk and boots it. Only that game's files
and the BIOS are served, on 127.0.0.1 only.

## Controls

| Keys | Dreamcast |
|---|---|
| W A S D | Analog stick |
| Arrow keys | D-pad |
| J / K / L / I | A / B / X / Y |
| Q / E | L / R triggers |
| Enter | Start |

Gamepads work automatically. Use EmulatorJS's save states (bottom bar) to save progress.

## What's custom here

- `ejs/data/src/dc-patches.js` – glue between the page and EmulatorJS, plus fixes:
  - loads picked / auto-run files into the emulator filesystem and puts the BIOS where Flycast looks;
  - applies core options on first run (stock EmulatorJS only does once settings are saved);
  - absorbs a WebGL `INVALID_ENUM` from an unsupported `GL_EXTENSIONS` query that otherwise
    makes RetroArch shut down the video driver;
  - converts GD-ROM `.cue` sheets to `.gdi`, since this core build crashes on them.
- `ejs/data/cores/flycast-wasm.data` – the flycast-wasm v1.0 release packaged as an EmulatorJS
  core by `tools/package-core.py`. The release was linked without libzip, so the HLE BIOS
  aborted when loading its font from the zip resources embedded in the core; the script
  routes those calls to `tools/libzip-shim.js`, a small read-only libzip in JavaScript
  (inflate by [tiny-inflate](https://github.com/foliojs/tiny-inflate), MIT).
- `serve.py`, `index.html` – local server and launcher page (with touch controls on mobile).
- `coi-serviceworker.js` – adds the cross-origin isolation headers on GitHub Pages, which
  can't set headers itself ([coi-serviceworker](https://github.com/gzuidhof/coi-serviceworker), MIT).

## Licenses

Flycast / flycast-wasm: GPLv2 (`LICENSE-flycast.txt`). EmulatorJS: GPLv3 (`ejs/LICENSE`).
