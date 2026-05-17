# Friendly authoring diagnostics

DevKit rc10 diagnostics should help authors fix the source problem instead of
exposing only internal resolver/runtime wording.

A good author-facing diagnostic answers:

- what happened;
- why it probably happened;
- which file and field to check;
- which focused command to run next;
- a small fixed snippet when useful;
- the relevant docs/template path.

## Implemented command surface

Current Boot diagnostics include friendly wrappers for:

| Family | Typical command | What to check |
| --- | --- | --- |
| Missing/malformed experience | `freven_boot config check` / `config explain` | `<instance>/experiences/<id>/experience.toml`, installed core experience ids |
| Mod resolution failure | `config check` / `config explain` | `experience.toml [[mods]]`, `experience.stack.toml [[layers.mods]]`, `<instance>/mods/<mod_id>/mod.toml` |
| Load-plan construction failure | provider/content commands | selected experience manifest, active mods, provider defaults, content roots |
| Provider default mismatch | `providers check/list/explain` | `[defaults]` / `[layers.defaults]` provider keys |
| Material Registry bridge/content graph | `providers check`, then `content-assets check` | visual content/material declarations |
| Content/assets graph failure | `content-assets check/explain/inspect/watch` | `content.manifest`, texture/material/model/effect declarations and files |

The Boot-side implementation landed in frevenengine/freven-boot#91.

## Experience or stack cannot resolve

Run:

    freven_boot config check --instance <instance> --experience <experience_id>

If the selected experience cannot be found or parsed, check:

- `<instance>/experiences/<id>/experience.toml`;
- `<instance>/experiences/<id>/experience.stack.toml`;
- packaged/core experience ids;
- the manifest `id` field.

Example standalone experience shape:

    schema = 1
    id = "example.experience"
    version = "1.0.0"
    title = "Example"

## Mod listed by an experience cannot resolve

Run:

    freven_boot config explain --instance <instance> --experience <experience_id>

Then check:

- `experience.toml [[mods]].id`;
- `experience.stack.toml [[layers.mods]].id`;
- `<instance>/mods/<mod_id>/mod.toml`;
- mod `id`;
- mod `version`;
- side/surface policy;
- unsafe/native/external policy flags when relevant.

Example stack-layer mod entry:

    [[layers.mods]]
    id = "example.mod"
    version = "^1.0"

The id must match the loaded `mod.toml` id exactly.

## Selected mod filter does not match

For config commands with `--mod <mod_id>`, the mod must be active in the selected
experience or stack.

Run:

    freven_boot config explain --instance <instance> --experience <experience_id>
    freven_boot providers explain --instance <instance> --experience <experience_id>

Then either remove `--mod`, choose an active mod id, or add the mod to the
selected experience/stack.

## Provider default does not resolve

Run:

    freven_boot providers list --instance <instance> --experience <experience_id>
    freven_boot providers explain --instance <instance> --experience <experience_id>

Check:

- `experience.toml [defaults].worldgen`;
- `experience.toml [defaults].character_controller`;
- `experience.stack.toml [layers.defaults].worldgen`;
- `experience.stack.toml [layers.defaults].character_controller`;
- `experience.stack.toml [layers.defaults].client_control_provider`;
- mod side/surface policy.

Example fixed snippet:

    [defaults]
    worldgen = "<listed_provider_key>"

Use a key printed by `providers list` / `providers explain`.

## Material Registry or visual-content bridge error

Provider diagnostics should not be used as the main visual-content validator.

Run:

    freven_boot content-assets check --instance <instance> --experience <experience_id>
    freven_boot content-assets inspect --instance <instance> --experience <experience_id>

Then fix the content/assets graph before debugging provider declarations.

## Content/assets failure

Run:

    freven_boot content-assets check --instance <instance> --experience <experience_id>

Then inspect details:

    freven_boot content-assets explain --instance <instance> --experience <experience_id>
    freven_boot content-assets inspect --instance <instance> --experience <experience_id> --kind material
    freven_boot content-assets watch --instance <instance> --experience <experience_id> --once

Content/assets diagnostics use the FVK vocabulary documented in
[Asset pipeline diagnostics](ASSET_PIPELINE_DIAGNOSTICS.md).

## World save, vertical contract, and runtime mutation families

Some diagnostics can only happen once a runtime/server/world path is involved.
They should still follow the same shape:

- world/save mismatch: identify the world directory, selected experience id, and
  whether the save belongs to a different experience/profile;
- vertical contract mismatch: identify the world/runtime height contract and the
  selected experience/worldgen contract;
- `expected_old` mutation mismatch: identify the mutation family, position or
  region when available, expected value, observed value, and whether the command
  was rejected for deterministic state safety.

For these families, do not move fixes into `mod.toml` or active config unless the
diagnostic explicitly points there. First check the selected experience/stack,
provider defaults, worldgen provider, and world/save compatibility.

## Authoring rule

Use diagnostics to fix the owning source:

- `mod.toml`: identity, dependency, runtime type, trust/surface/capability
  metadata;
- `experience.toml` / `experience.stack.toml`: selected mods, defaults, content
  root, stack overlays;
- `config.schema.toml` and active config: tunable runtime settings;
- `content/` and `content.manifest`: authored gameplay/visual content source;
- generated cache: rebuildable output;
- `worlds/`: runtime save state.

## Related docs

- [Getting started](GETTING_STARTED.md)
- [First Wasm mod](FIRST_WASM_MOD.md)
- [Provider selection authoring](PROVIDER_AUTHORING.md)
- [Data/content asset workflow](DATA_CONTENT_ASSET_WORKFLOW.md)
- [Asset pipeline diagnostics](ASSET_PIPELINE_DIAGNOSTICS.md)
- [Mod DevTools v1](MOD_DEVTOOLS.md)
- [Troubleshooting](TROUBLESHOOTING.md)
