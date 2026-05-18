# Asset authoring templates

DevKit ships rc10 asset authoring templates from:

    templates/asset_authoring_templates/

These templates are packaged by freven-boot and are intended as copyable,
content-first starting points for visual/content authoring.

## Templates

| Template | Use when |
| --- | --- |
| `texture_override_pack` | You want to replace or test texture/material source. |
| `new_textured_block_pack` | A gameplay mod registers a block and content owns its material/texture. |
| `material_pack` | You want reusable material keys shared by blocks/models. |
| `block_model_pack` | You want forward-compatible model/visual source files next to validated materials. |
| `standalone_visual_style_pack` | A standalone product owns its style instead of inheriting Vanilla visuals. |
| `rock_family_pack` | You want compact generated rock variants with material/visual/tag entries. |
| `soil_grass_family_pack` | You want generated soil x grass coverage variants with skipped combinations. |
| `colored_glass_family_pack` | You want generated transparent colored-glass variants. |

## What each template contains

Each template includes:

- `experience.toml` as a minimal selected experience wrapper;
- `content.manifest` with active manifest v1 declarations, or a small root manifest with explicit `includes = [...]` for modular packs;
- `content/textures/example_16.png`;
- `content/textures/example_32.png`;
- material declarations;
- family declarations for generated variant templates where applicable;
- README notes with `content-assets check`, `content-assets explain`, and `content-assets inspect --kind generated`.

`block_model_pack` also includes forward-compatible model and block-visual
source skeleton files. They are tracked through `content.manifest` entries while
the full model/block visual authoring schema matures.

## Validate templates

From the freven-boot repository:

    bash scripts/check_asset_authoring_templates.sh

The script verifies:

- expected template directories exist;
- each template has README, experience, manifest, and content notes;
- example textures are valid 16x16 and 32x32 PNGs;
- manifest sha256 values match source files;
- each template passes `freven_boot content-assets check`.

## Modular template layout

For larger templates and future Vanilla-style visual packs, use this layout:

    content.manifest
    content/
      textures/terrain.toml
      materials/terrain.toml
      models/blocks.toml
      visuals/blocks.toml
      families/rocks.toml
      tags/terrain.toml

The root manifest is an explicit index:

    schema = 1
    includes = [
      "content/textures/terrain.toml",
      "content/materials/terrain.toml",
      "content/models/blocks.toml",
      "content/visuals/blocks.toml",
      "content/families/rocks.toml",
      "content/tags/terrain.toml",
    ]

Do not use directory scanning, generated cache, or renderer/internal ids as
authoring template source. See [Modular content authoring](MODULAR_CONTENT_AUTHORING.md).

When a selected game provides a higher-level authoring profile, template docs should name the profile id and explain the `freven_boot content compile` step. For Vanilla-style block authoring, the intended profile is `freven.vanilla:blocktypes_v1`; low-level visual templates may continue to use `freven.core:canonical_manifest_v1` directly. See [Content authoring profiles](CONTENT_AUTHORING_PROFILES.md).

## Use a template

Copy a template directory into an instance `experiences/` directory, then run:

    freven_boot content-assets update-sha --instance <instance> --experience <template_id>
    freven_boot content-assets update-sha --instance <instance> --experience <template_id> --write
    freven_boot content-assets check --instance <instance> --experience <template_id>
    freven_boot content-assets explain --instance <instance> --experience <template_id>
    freven_boot content-assets inspect --instance <instance> --experience <template_id>
    freven_boot content-assets watch --instance <instance> --experience <template_id> --once

Before shipping:

- replace `example.asset.*` ids and keys with your namespace;
- replace example PNGs with real texture bytes;
- run `freven_boot content-assets update-sha --instance <instance> --experience <template_id> --write` after replacing texture bytes;
- keep generated cache outside authored source;
- keep active runtime config for tuning, not visual definitions.

## Relationship to diagnostics

These templates are designed to avoid common diagnostics from
[Asset pipeline diagnostics](ASSET_PIPELINE_DIAGNOSTICS.md):

- missing include files or include cycles;
- duplicate semantic keys across included files;
- stale or missing texture sha256 fields;
- missing texture files;
- bad texture paths;
- missing material texture references;
- bad namespace/path keys;
- generated cache used as source.

Use [Data/content asset workflow](DATA_CONTENT_ASSET_WORKFLOW.md) for ownership
boundaries and [Troubleshooting](TROUBLESHOOTING.md) when a copied template does
not load.

## Related docs

- [Engine vs Vanilla ownership](ENGINE_VANILLA_OWNERSHIP.md)

- [Project templates](PROJECT_TEMPLATES.md)
- [Modular content authoring](MODULAR_CONTENT_AUTHORING.md)

## Family templates

Family templates demonstrate compact generated variant authoring:

    freven_boot content-assets check --instance <instance> --experience <template_id>
    freven_boot content-assets inspect --instance <instance> --experience <template_id> --kind generated

Use:

- `rock_family_pack` for rock variants;
- `soil_grass_family_pack` for cartesian-product variants with skipped
  combinations;
- `colored_glass_family_pack` for transparent generated variants with structured
  axis metadata.

The framed-glass workflow is currently an advanced composition of
`block_model_pack` and `colored_glass_family_pack`; a dedicated framed-glass
family template should wait until rich per-part model-family templates are
stable.

See [Content family authoring](CONTENT_FAMILY_AUTHORING.md).
