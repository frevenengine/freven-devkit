# Freven DevKit

Freven DevKit is the **developer distribution** of Freven:
- `freven_boot` launcher (resolves experiences/mods, creates load plans)
- `freven_client` and `freven_server` binaries
- `core_experiences/` (includes the Vanilla reference experience)
- templates for clean client/server instances

This repository contains **documentation and issue tracking only**.
**Binaries are distributed via GitHub Releases**.

## Download

Go to **Releases** and download the archive for your OS:

- **Linux**: `freven-devkit-vX.Y.Z-x86_64-unknown-linux-gnu.tar.gz`
- **Windows**: `freven-devkit-vX.Y.Z-x86_64-pc-windows-msvc.zip`
- **macOS**: `freven-devkit-vX.Y.Z-*-apple-darwin.(tar.gz|zip)` (when available)

After extracting, you will get a folder like:
`freven-devkit-vX.Y.Z-<target>/`

## Quickstart (Dedicated server + client)

### Linux / macOS
```bash
cd freven-devkit-vX.Y.Z-<target>

mkdir -p instances
./freven_boot init --instance instances/server server
./freven_boot init --instance instances/player client

# Terminal 1:
./freven_boot serve --instance instances/server

# Terminal 2:
./freven_boot play --instance instances/player -- --username dev --devtools --connect 127.0.0.1:12806
```

### Windows (PowerShell)

```powershell
cd freven-devkit-vX.Y.Z-<target>

mkdir instances
.\freven_boot.exe init --instance instances\server server
.\freven_boot.exe init --instance instances\player client

# Terminal 1:
.\freven_boot.exe serve --instance instances\server

# Terminal 2:
.\freven_boot.exe play --instance instances\player -- --username dev --devtools --connect 127.0.0.1:12806
```

## Quickstart (Singleplayer / inproc)

### Linux / macOS

```bash
cd freven-devkit-vX.Y.Z-<target>
mkdir -p instances
./freven_boot init --instance instances/sp client
./freven_boot play --instance instances/sp -- --username dev --devtools
```

### Windows (PowerShell)

```powershell
cd freven-devkit-vX.Y.Z-<target>
mkdir instances
.\freven_boot.exe init --instance instances\sp client
.\freven_boot.exe play --instance instances\sp -- --username dev --devtools
```

## Where things live

An **instance** is your mutable user data folder. It will contain:

* `logs/` (runtime logs)
* `run/` (generated load plans)
* `worlds/` (world storage)
* `net/` (QUIC trust/certs/TOFU pins)
* `mods/` (your mods)
* `experiences/` (your custom experiences)

Important:

* The DevKit **archive is read-only**. You can delete and re-extract it anytime.
* Your instances are safe to keep between updates.

## Writing mods / creating experiences

* Neutral platform-shaped authoring uses `freven_mod_api`, `freven_guest_sdk`,
  and `freven_sdk_types` from **freven-sdk**.
* Current gameplay/world-stack authoring uses the explicit world-owned
  `freven_world_api`, `freven_world_guest_sdk`, and `freven_world_sdk_types`
  surfaces from **freven-sdk**.
* Vanilla is the first-party reference experience for that explicit world-owned
  path.

See:

* `docs/GETTING_STARTED.md`
* freven-sdk docs (WASM ABI / External IPC / Native ABI).

## Bug reports

Before opening an issue, please read:

* `docs/REPORTING_BUGS.md`

Include:

* output of `freven_boot --version`
* `manifest/devkit_manifest.toml`
* relevant logs from your instance (`logs/`)

## License

* This repository (docs/templates) is under **MIT** (see `LICENSE`).
* DevKit binaries/content are covered by `LICENSE-DEVKIT.txt` (distributed with the release).
