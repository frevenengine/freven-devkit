# Troubleshooting

## DevKit can't find core experiences
Symptom: `experience 'freven.vanilla' was not found` or list is empty.

Fix:
- Run `freven_boot` from the extracted DevKit folder (where `core_experiences/` exists).
- If needed, set an override:
  - `FREVEN_CORE_EXPERIENCES_DIR=/path/to/core_experiences`

## `freven_boot --version` says manifest not found
Make sure you run from the DevKit folder that contains:
`manifest/devkit_manifest.toml`

Or set:
- `FREVEN_DEVKIT_MANIFEST=/path/to/devkit_manifest.toml`

## Client opens then exits quickly
Collect logs:
- `<instance>/logs/`
- `<instance>/run/` (load_plan JSON helps reproduce)

Try with:
- `RUST_LOG=info` (or `debug`)

## Vulkan / graphics issues (Linux)
Common causes:
- missing/old GPU drivers
- running in a remote session without GPU acceleration

Verify your Vulkan works in general (system-level). Then retry.

## Windows firewall / networking
If connecting to a dedicated server fails:
- allow `freven_server.exe` through firewall
- confirm the server is listening on the expected port (default 12806)
- confirm `--connect host:port` matches the server bind address

## QUIC trust issues (client)
If the client refuses to connect due to certificate/trust changes:
- safest: `--quic-forget <host:port>` then reconnect
- or reset trust store: `--quic-reset-trust`
- `--quic-accept-new-cert` should be used carefully (a changed cert can be a MITM or server reinstall)

## Provider default does not resolve

If launch or validation reports a missing worldgen, character controller, or
client control provider, inspect the resolved provider catalog:

~~~bash
./freven_boot providers explain --instance <instance> --experience <experience_id>
~~~

Then check that the selected key in `[defaults]` or `[layers.defaults]` exactly
matches a provider registered by an active mod. For stack experiences, mods added
by the overlay must be listed under `[[layers.mods]]`; top-level `[[mods]]` is
the standalone `experience.toml` shape, not the stack overlay shape. Provider
default validation is side-aware: server validates `worldgen` and
`character_controller`, while client validates `character_controller` and
`client_control_provider`.

See [Provider selection authoring](PROVIDER_AUTHORING.md).

## Content or asset file is not loading

First identify the boundary that owns the fix:

- `mod.toml` is manifest/capability metadata.
- `config.schema.toml` and active `[config."<mod_id>"]` values are runtime
  configuration.
- `content/` and `content.manifest` are authored gameplay/visual content source.
- asset files are resource bytes referenced by content declarations.
- generated cache is rebuildable output.
- `worlds/` is save/world state, not shipped defaults.

See [Data/content asset workflow](DATA_CONTENT_ASSET_WORKFLOW.md) before moving
authored data into config or rebuilding Wasm just to change data files.
