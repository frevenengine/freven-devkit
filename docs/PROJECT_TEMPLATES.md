# Project templates

Freven DevKit rc10 ships project templates for common authoring starts.

Project templates are different from asset authoring templates:

- project templates create a starter project/stack/mod layout;
- asset authoring templates create starter visual/content declarations;
- standalone product templates create a product-owned game scaffold.

Use project templates when you want the correct Freven directory shape,
manifest boundaries, SDK pinning, and first validation commands before writing
custom code.

The Boot-side packaged templates landed in frevenengine/freven-boot#92.

## Where they live

In a packaged DevKit distribution:

    templates/project_templates/

In the Boot source tree:

    templates/project_templates/
    scripts/check_project_templates.sh

Validate the template catalog from Boot source:

    bash scripts/check_project_templates.sh

## Template catalog

| Template | Use when | Main surface |
| --- | --- | --- |
| `standalone_game` | You are creating a zero-base product-owned game. | standalone product generator |
| `vanilla_mod` | You are extending an existing base experience with a runtime-loaded Wasm mod. | stack + Wasm mod |
| `content_pack` | You are shipping authored data/assets without Wasm. | experience + content root |
| `game_mode` | You are changing provider defaults/config around a base experience. | stack layer |
| `worldgen_mod` | You are creating a server-side Wasm worldgen provider. | stack + server Wasm mod |
| `block_mod` | You are registering gameplay blocks and pairing them with content/assets. | stack + Wasm mod + content root |
| `simulation_mod` | You are creating server-side tick/action simulation behavior. | stack + server Wasm mod |

## Important ownership rules

Project templates keep source ownership explicit:

- engine/runtime behavior stays generic;
- Vanilla is a reference/base experience, not a required dependency;
- standalone products may be zero-Vanilla and product-owned;
- mods/content packs extend selected bases through declared layers;
- visual style belongs to Vanilla, a standalone product, or selected content packs,
  not engine code.

See [Engine vs Vanilla ownership](ENGINE_VANILLA_OWNERSHIP.md) for the full
shader/material/visual-style boundary.

- `mod.toml` is package identity, runtime type, trust/surface/capability metadata;
- `experience.toml` and `experience.stack.toml` select mods, defaults, content roots, and stack overlays;
- `config.schema.toml` defines tunable runtime settings;
- active config stores user/admin-selected values;
- `content/` and `content.manifest` store authored gameplay/visual content source;
- generated cache is rebuildable output;
- `worlds/` is runtime save state.

Do not move authored content into `mod.toml`, active config, generated cache, or
world save data just to make a template work.

## Base experience placeholder

Stack-based templates use:

    base = "<base_experience_id>"

This is intentional. The templates must work in DevKit profiles that may or may
not bundle first-party Vanilla content.

Replace `<base_experience_id>` with the experience you are targeting, then run:

    freven_boot config check --instance <instance> --experience <experience_id>
    freven_boot providers explain --instance <instance> --experience <experience_id>
    freven_boot content-assets check --instance <instance> --experience <experience_id>

## Standalone game

Use `standalone_game` when the game owns its product identity and does not depend
on a base experience.

The template delegates to the checked standalone product generator:

    tools/new_standalone_product.sh --template shell --product-id my.game --product-name "My Game" --out ./my-game
    tools/new_standalone_product.sh --template playable --product-id my.game --product-name "My Game" --out ./my-game

Then validate the generated project:

    ./scripts/check.sh

Use this for product-owned games, experiments, and future standalone releases.

## Vanilla-style runtime mod

Use `vanilla_mod` when you want a runtime-loaded Wasm mod layered onto an
existing base experience.

The template includes:

- `experience.stack.toml`;
- `mod.toml`;
- `Cargo.toml` pinned to the SDK tag;
- `src/lib.rs` with `freven_world_guest_sdk::wasm_guest!`;
- copy/check helper scripts.

Replace the base experience placeholder, build the Wasm guest, copy it to the
mod directory as `mod.wasm`, then run:

    freven_boot config check --instance <instance> --experience example.project.vanilla_mod.stack
    freven_boot providers explain --instance <instance> --experience example.project.vanilla_mod.stack

## Content pack

Use `content_pack` when no Wasm code is needed.

The template includes:

- selected experience wrapper;
- content root;
- `content.manifest`;
- validation helpers.

Validate with:

    freven_boot content-assets check --instance <instance> --experience example.project.content_pack

For texture/material/model starters, combine this with
[Asset authoring templates](ASSET_AUTHORING_TEMPLATES.md).

## Game mode

Use `game_mode` for stack-level changes around an existing base experience.

Typical changes:

- provider defaults;
- config overrides;
- selected runtime-loaded mods;
- content layers.

Start by listing provider keys:

    freven_boot providers list --instance <instance> --experience <base_experience_id>

Then replace placeholders in `[layers.defaults]` with listed provider keys.

## Worldgen mod

Use `worldgen_mod` for server-side terrain/world generation.

The template includes:

- server-only `mod.toml`;
- worldgen capabilities;
- SDK-pinned Wasm guest crate;
- stack default selecting the registered worldgen key.

Validate with:

    freven_boot providers explain --instance <instance> --experience example.project.worldgen_mod.stack

## Block mod

Use `block_mod` when code registers gameplay block keys and content owns visual
source.

The template includes:

- Wasm block registration skeleton;
- stack layer;
- content root;
- empty content manifest;
- links naturally to asset authoring templates for visual files.

Validate code/provider shape and visual content separately:

    freven_boot config check --instance <instance> --experience example.project.block_mod.stack
    freven_boot content-assets check --instance <instance> --experience example.project.block_mod.stack

## Simulation mod

Use `simulation_mod` for server-side lifecycle/tick/action behavior.

The template includes:

- server-only Wasm guest;
- config schema;
- stack layer;
- tick skeleton.

Runtime truth belongs in world/runtime state, not generated cache or authored
content files.

Validate with:

    freven_boot config check --instance <instance> --experience example.project.simulation_mod.stack
    freven_boot providers explain --instance <instance> --experience example.project.simulation_mod.stack

## First validation loop

After copying any project template into an instance:

    freven_boot config check --instance <instance> --experience <experience_id>
    freven_boot providers explain --instance <instance> --experience <experience_id>
    freven_boot content-assets check --instance <instance> --experience <experience_id>

For visual/content iteration:

    freven_boot content-assets inspect --instance <instance> --experience <experience_id>
    freven_boot content-assets watch --instance <instance> --experience <experience_id>

For friendly errors:

    freven_boot config explain --instance <instance> --experience <experience_id>
    freven_boot providers explain --instance <instance> --experience <experience_id>

## Relationship to other templates

Use:

- project templates for project shape;
- asset authoring templates for texture/material/model/content examples;
- standalone product generator for product-owned games;
- `FIRST_WASM_MOD.md` for a longer first Wasm walkthrough.

## Related docs

- [Getting started](GETTING_STARTED.md)
- [First Wasm mod](FIRST_WASM_MOD.md)
- [Data/content asset workflow](DATA_CONTENT_ASSET_WORKFLOW.md)
- [Asset authoring templates](ASSET_AUTHORING_TEMPLATES.md)
- [Asset pipeline diagnostics](ASSET_PIPELINE_DIAGNOSTICS.md)
- [Asset inspector / devtools](ASSET_INSPECTOR_DEVTOOLS.md)
- [Asset/content hot reload](ASSET_CONTENT_HOT_RELOAD.md)
- [Friendly authoring diagnostics](FRIENDLY_DIAGNOSTICS.md)
- [Mod DevTools v1](MOD_DEVTOOLS.md)
- [Troubleshooting](TROUBLESHOOTING.md)
