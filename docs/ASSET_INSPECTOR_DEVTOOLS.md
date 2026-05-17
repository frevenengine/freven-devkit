# Asset inspector / devtools

DevKit ships a first asset inspector surface through:

    freven_boot content-assets inspect

This is the rc10 CLI inspector for resolved visual asset load plans. It is not
the final in-game debug UI, but it establishes the stable inspection contract
that later UI/devtools surfaces should reuse.

## Commands

Inspect all resolved assets:

    freven_boot content-assets inspect --instance <instance> --experience <experience_id>

Inspect one asset kind:

    freven_boot content-assets inspect --instance <instance> --experience <experience_id> --kind material

Inspect one stable asset key:

    freven_boot content-assets inspect --instance <instance> --experience <experience_id> --kind material --key example.assets:materials/stone

Show renderer/runtime internal slots:

    freven_boot content-assets inspect --instance <instance> --experience <experience_id> --kind material --key example.assets:materials/stone --show-internal

## What it shows

The inspector prints author-facing resolved asset information:

- selected experience/stack content layers;
- load-plan fingerprint;
- texture/material/model/effect/generated counts;
- owner layer for each matching asset;
- source path when available;
- dependencies;
- shadowed declarations;
- render-layer summary;
- missing key guidance;
- internal slots only when `--show-internal` is explicitly used.

## Stable identity vs debug internals

Stable author-facing identity is the asset key:

    namespace:path

Internal slots are renderer/runtime debug output. They are useful for diagnostics,
performance debugging, and future UI inspector panels, but they are not an author
API and should not be referenced from content, config, mods, saves, or templates.

## Relationship to other content/assets commands

Use:

- `content-assets check` when something fails to load;
- `content-assets explain` when you need the full resolved load plan;
- `content-assets inspect` when you want a focused view of one kind/key or a
  devtools-friendly summary.

## Current scope

This is the DevKit CLI inspector v1.

It covers:

- listing resolved textures/materials/models/effects/generated assets;
- inspecting a specific stable key;
- showing dependencies and shadowed declarations;
- summarizing render layers;
- exposing internal slot ids as debug-only output.

Future runtime/in-game devtools can build on the same contract for:

- block-under-cursor resolved asset inspection;
- live visual overlays;
- atlas/texture-array visualization;
- hot reload event display;
- richer material/shader diagnostics panels.

## Related docs

- [Data/content asset workflow](DATA_CONTENT_ASSET_WORKFLOW.md)
- [Asset pipeline diagnostics](ASSET_PIPELINE_DIAGNOSTICS.md)
- [Asset authoring templates](ASSET_AUTHORING_TEMPLATES.md)
- [Troubleshooting](TROUBLESHOOTING.md)
