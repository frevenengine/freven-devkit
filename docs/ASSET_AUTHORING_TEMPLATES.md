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

## What each template contains

Each template includes:

- `experience.toml` as a minimal selected experience wrapper;
- `content.manifest` with active manifest v1 declarations;
- `content/textures/example_16.png`;
- `content/textures/example_32.png`;
- material declarations;
- README notes with `content-assets check` and `content-assets explain`.

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

## Use a template

Copy a template directory into an instance `experiences/` directory, then run:

    freven_boot content-assets check --instance <instance> --experience <template_id>
    freven_boot content-assets explain --instance <instance> --experience <template_id>
    freven_boot content-assets inspect --instance <instance> --experience <template_id>
    freven_boot content-assets watch --instance <instance> --experience <template_id> --once

Before shipping:

- replace `example.asset.*` ids and keys with your namespace;
- replace example PNGs with real texture bytes;
- update sha256 values in `content.manifest`;
- keep generated cache outside authored source;
- keep active runtime config for tuning, not visual definitions.

## Relationship to diagnostics

These templates are designed to avoid common diagnostics from
[Asset pipeline diagnostics](ASSET_PIPELINE_DIAGNOSTICS.md):

- missing texture files;
- bad texture paths;
- missing material texture references;
- bad namespace/path keys;
- generated cache used as source.

Use [Data/content asset workflow](DATA_CONTENT_ASSET_WORKFLOW.md) for ownership
boundaries and [Troubleshooting](TROUBLESHOOTING.md) when a copied template does
not load.
