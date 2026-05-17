# Getting Started

## 1) Install / extract

Download the DevKit archive for your OS from GitHub Releases and extract it.
You should see:

- `freven_boot` (or `freven_boot.exe`)
- `freven_client` / `freven_server`
- `core_experiences/`
- `templates/`
- `manifest/`
- `licenses/`

## 2) Verify the build (important for bug reports)

Linux/macOS:
```bash
./freven_boot --version
```

Windows:

```powershell
.\freven_boot.exe --version
```

This prints the DevKit manifest location and versions/refs for:

* devkit_version
* engine/sdk/vanilla refs + commits
* target triple, build time, net proto

## 3) Create instances (portable)

We recommend keeping instances next to the DevKit folder (portable).

Linux/macOS:

```bash
mkdir -p instances
./freven_boot init --instance instances/player client
./freven_boot init --instance instances/server server
```

Windows:

```powershell
mkdir instances
.\freven_boot.exe init --instance instances\player client
.\freven_boot.exe init --instance instances\server server
```

Tip:

* `init` is safe by default: it **creates missing files** and does not overwrite existing ones.
* Use `--force` only if you really want to overwrite template files.
* Dedicated server instances now carry explicit world-stack config sections in
  `server.toml`: `[world_bootstrap]` selects the world save to open/create and
  `[world_streaming]` sets join/view/budget policy for world streaming.

## 4) Run dedicated server + connect a client

Linux/macOS:

```bash
./freven_boot serve --instance instances/server
./freven_boot play --instance instances/player -- --connect 127.0.0.1:12806 --username dev --devtools
```

Windows:

```powershell
.\freven_boot.exe serve --instance instances\server
.\freven_boot.exe play --instance instances\player -- --connect 127.0.0.1:12806 --username dev --devtools
```

## 5) Customize content

### Experiences

Put custom experiences here:
`<instance>/experiences/<experience_id>/experience.toml`

Instance experiences override core experiences by ID (deterministic priority).

### Mods

Put mods here:
`<instance>/mods/<mod_id>/...`

For a first runtime-loaded Wasm mod, follow
[`FIRST_WASM_MOD.md`](FIRST_WASM_MOD.md).

The exact on-disk format depends on runtime type (wasm/external/native).
See freven-sdk documentation for the current ABI contracts and layouts.

Current authoring ownership:

* use [`DATA_CONTENT_ASSET_WORKFLOW.md`](DATA_CONTENT_ASSET_WORKFLOW.md) when
  shipping authored gameplay definitions, visual content declarations,
  data-only content, or product-owned content; do not put authored content
  data in active config, generated cache, or save/world state
* use `freven_guest_sdk` / `freven_mod_api` only for neutral platform-shaped
  declarations
* use `freven_world_guest_sdk` / `freven_world_api` for gameplay, block/content
  registration, world queries/mutations, terrain-write worldgen, providers,
  and other current world-stack behavior
* use [`BLOCK_MUTATION_AUTHORING.md`](BLOCK_MUTATION_AUTHORING.md) for
  simulation-style runtime block mutation patterns, budgets, and diagnostics
* use `freven-vanilla` as the first-party reference above that world stack

## 6) Useful flags

* `--devtools` (developer HUD / debugging tools in supported builds)
* `--connect host:port` (connect to a dedicated server)
* QUIC trust maintenance (client):

  * `--quic-forget <host:port>`
  * `--quic-reset-trust`
  * `--quic-accept-new-cert` (use with care)

## Check provider selection

Experiences can explicitly select worldgen, character controller, and client
control provider keys. Before launching an authored experience, inspect and
validate those provider defaults:

~~~bash
./freven_boot providers list --instance instances/player --experience <experience_id>
./freven_boot providers check --instance instances/player --experience <experience_id>
./freven_boot providers explain --instance instances/player --experience <experience_id>
~~~

On Windows, use `freven_boot.exe` with the same `providers` subcommands.

See [Provider selection authoring](PROVIDER_AUTHORING.md) for the full provider
key contract and side-aware validation rules.\n