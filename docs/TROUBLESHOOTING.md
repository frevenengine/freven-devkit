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
- `content/` and `content.manifest` are authored gameplay/visual content source. For large packs, `content.manifest` may be a small explicit `includes = [...]` index over modular source files.
- asset files are resource bytes referenced by content declarations.
- generated cache is rebuildable output.
- `worlds/` is save/world state, not shipped defaults.

See [Data/content asset workflow](DATA_CONTENT_ASSET_WORKFLOW.md) before moving
authored data into config or rebuilding Wasm just to change data files.

## Asset pipeline diagnostic points at the wrong kind of file

Use [Asset pipeline diagnostics](ASSET_PIPELINE_DIAGNOSTICS.md) to identify the
ownership boundary before editing files.

Common examples:

- missing `namespace:path` key: fix the authored content declaration or the
  package/layer that should provide the key;
- missing PNG/model/shader file: fix the asset path or add the resource file;
- alpha/render-layer mismatch: fix the material or visual declaration;
- generated atlas/load-plan/cache problem: rebuild generated cache, do not edit
  generated output as source;
- Material Registry v1 errors during `providers check`: current DevKit builds
  the resolved Material Registry before provider validation. If this still
  fails, run `freven_boot content-assets check` first because the problem is
  now in the content/assets graph, not provider declaration resolution.

## Content authoring profile compile fails

If `freven_boot content compile` reports an unknown or unsupported profile, check
which authoring layer you intended to use.

Use the canonical passthrough profile for current low-level manifests:

    ./freven_boot content compile --instance <instance> --experience <experience_id> --profile freven.core:canonical_manifest_v1 --explain

Use a game-owned profile only when the selected game or template documents it.
For future Vanilla blocktype/worldproperty authoring, the intended profile id is:

    freven.vanilla:blocktypes_v1

If the Vanilla profile is reported as unsupported, that means the schema is
documented but the compiler is not available in the current DevKit build. Use
the current canonical manifest workflow until the profile compiler lands.

After compile/explain, validate the generated or existing canonical graph:

    ./freven_boot content-assets check --instance <instance> --experience <experience_id>

See [Content authoring profiles](CONTENT_AUTHORING_PROFILES.md).

## Modular content manifest include fails

If a content/assets command reports a missing include, invalid include path,
include cycle, or duplicate semantic key, start from the root manifest selected
by the experience:

    [content]
    root = "content"
    manifest = "content.manifest"

Check that:

- `includes = [...]` paths are explicit and ordered;
- include paths are relative to the file that declares them;
- the root `content.manifest` sits next to `content/`, so root includes usually
  look like `content/textures/terrain.toml`;
- included files stay inside the selected content root;
- no include uses an absolute path, `..`, a directory, or generated cache;
- no two included files declare the same texture/material/model/visual/family/tag
  key unless that is expressed through a proper selected layer override;
- family declarations and generated outputs still validate after expansion.

Run:

    ./freven_boot content-assets check --instance <instance> --experience <experience_id>
    ./freven_boot content-assets explain --instance <instance> --experience <experience_id>

See [Modular content authoring](MODULAR_CONTENT_AUTHORING.md).

## Texture sha256 is missing or stale

If `content-assets check` reports a missing or stale texture hash, do not compute
and paste hashes by hand. First dry-run:

    ./freven_boot content-assets update-sha --instance <instance> --experience <experience_id>

Then write the exact authored manifest or included source file that owns the
texture declaration:

    ./freven_boot content-assets update-sha --instance <instance> --experience <experience_id> --write

The command uses declared texture entries only. It does not scan random files
into content.

## Texture, material, model, or content patch does not load

Run the resolved content/assets check first:

```bash
./freven_boot content-assets check --instance <instance> --experience <experience_id>
```

To inspect the selected graph:

```bash
./freven_boot content-assets explain --instance <instance> --experience <experience_id>
```

Use this before launching the full client/server runtime. The command reports
manifest/load-plan/texture asset failures with FVK diagnostics and distinguishes
authored content/schema problems from runtime/provider bridge problems.

## Friendly authoring diagnostics

DevKit diagnostics should point at the owning file/field and the next focused
command.

Use [Friendly authoring diagnostics](FRIENDLY_DIAGNOSTICS.md) when an error
mentions:

- `mod.toml`;
- `experience.toml` or `experience.stack.toml`;
- provider defaults;
- worldgen selection;
- load-plan construction;
- world/save compatibility;
- vertical contract mismatch;
- `expected_old` mutation mismatch.

Start with:

    freven_boot config check --instance <instance> --experience <experience_id>
    freven_boot providers explain --instance <instance> --experience <experience_id>
    freven_boot content-assets check --instance <instance> --experience <experience_id>

## Related docs

- [Engine vs Vanilla ownership](ENGINE_VANILLA_OWNERSHIP.md)
- [Modular content authoring](MODULAR_CONTENT_AUTHORING.md)

## Content family expansion fails

Run:

    ./freven_boot content-assets check --instance <instance> --experience <experience_id>

Then inspect generated output after the manifest validates:

    ./freven_boot content-assets inspect --instance <instance> --experience <experience_id> --kind generated

Check:

- the family key is namespaced;
- every axis has valid values;
- `allow` and `skip` rules reference existing axes and values;
- no combination is both allowed and skipped;
- generated material/model/visual/tag keys do not conflict unintentionally;
- generated materials reference existing textures;
- generated visuals reference existing models and materials.

See [Content family authoring](CONTENT_FAMILY_AUTHORING.md).
