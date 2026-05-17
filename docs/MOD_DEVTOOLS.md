# Mod DevTools v1

Freven DevKit rc10 provides a first mod-devtools workflow for authoring,
validating, inspecting, and iterating on mods without relying only on launch
logs.

This is a v1 surface. It intentionally starts with stable CLI/devtools
contracts and debug HUD/export surfaces before promising live in-game runtime
editing for every subsystem.

## Quick command map

| Need | Tool |
| --- | --- |
| Check selected experience/stack config | `freven_boot config check` |
| Inspect selected effective config | `freven_boot config explain` |
| Check selected provider defaults | `freven_boot providers check` |
| List registered providers by side | `freven_boot providers list` |
| Explain provider ownership/default selection | `freven_boot providers explain` |
| Check texture/material/model/content graph | `freven_boot content-assets check` |
| Print full resolved content/assets load plan | `freven_boot content-assets explain` |
| Inspect one asset kind/key | `freven_boot content-assets inspect` |
| Watch content/assets during visual iteration | `freven_boot content-assets watch` |
| Inspect runtime/debug metrics in client | Debug HUD |
| Export/debug HUD metrics for CI/log review | Debug HUD dump/log consumers |

## Provider viewer

Use provider commands when debugging worldgen, character controller, or
client-control provider ownership.

    freven_boot providers check --instance <instance> --experience <experience_id>
    freven_boot providers list --instance <instance> --experience <experience_id>
    freven_boot providers explain --instance <instance> --experience <experience_id>

`providers explain` is the author-facing provider viewer v1. It shows selected
defaults, hosted sides, loaded provider declarations, and deferred runtime-guest
validation boundaries.

## Active experience / layers / mods

Use:

    freven_boot config explain --instance <instance> --experience <experience_id>
    freven_boot content-assets explain --instance <instance> --experience <experience_id>

Together these show the selected experience/stack, effective config, content
layers, resolved asset graph, dependency edges, shadowed declarations, and load
plan fingerprints.

## Visual asset inspector

Use:

    freven_boot content-assets inspect --instance <instance> --experience <experience_id>
    freven_boot content-assets inspect --instance <instance> --experience <experience_id> --kind material
    freven_boot content-assets inspect --instance <instance> --experience <experience_id> --kind material --key example.assets:materials/stone

Stable author-facing identity is always a `namespace:path` key.

Renderer/internal slots can be shown only as debug output:

    freven_boot content-assets inspect --instance <instance> --experience <experience_id> --kind material --key example.assets:materials/stone --show-internal

Do not use internal slots as authored content identity.

## Hot reload / edit loop

Use:

    freven_boot content-assets watch --instance <instance> --experience <experience_id>

This is a conservative development workflow. It watches resolved content roots
and manifests, reruns visual asset validation, prints changed files and
fingerprints, and keeps running after broken reload attempts.

It does not yet promise live Wasm module replacement, GPU texture replacement,
material-table swaps, or chunk mesh invalidation in a running game session.
When those runtime hooks are absent, restart the running client/server after
the watcher validates the change.

## Block inspector v1

Current stable block/visual inspection is split across:

- content/assets inspection for visual keys, materials, models, render layers,
  dependencies, and internal debug slots;
- provider/config explanations for selected gameplay/provider ownership;
- debug HUD world/rendering sections for runtime world, streaming, meshing,
  prediction, and worldgen state.

Full in-game “block under cursor” inspection is a future runtime UI layer. It
should reuse the same stable identity model:

- block key;
- runtime/storage id as debug-only output;
- tags;
- collision/solid policy;
- material/visual key;
- render layer;
- authoritative vs predicted/overlay source.

## Mutation and rejected-output diagnostics

Runtime mutation behavior is visible today through:

- runtime rejection diagnostics;
- block mutation authoring docs;
- debug HUD/runtime metrics and dumps;
- server/worldgen/world-stream counters;
- tests that keep rejected runtime output and over-budget mutation behavior
  explicit.

A full interactive mutation log UI is future runtime/devtools work. It should
build on the existing runtime-output vocabulary and include:

- lifecycle/action source;
- mutation family;
- command count;
- budget result;
- applied/rejected disposition;
- affected regions where available.

## Worldgen and transient overlay diagnostics

Current debug surfaces include worldgen runtime metrics, world-stream metrics,
meshing counters, prediction/overlay counters, and debug HUD dump/log consumers.

These are developer diagnostics. They are not save state and not authored
content identity.

## V1 scope

This issue is considered complete for rc10 when a mod author has one place to
discover the available DevKit tools for:

- provider viewing;
- active experience/config/content inspection;
- content/assets validation;
- visual asset inspection;
- conservative asset/content watch workflows;
- debug HUD/dump/log surfaces;
- current limits and future runtime-devtools hooks.

## Future runtime-devtools hooks

Future work should add deeper live/in-game tooling without changing the v1
authoring contracts:

- live Wasm module reload where safe;
- block-under-cursor inspector panel;
- mutation log UI;
- rejected runtime-output panel;
- worldgen column debug panel;
- transient overlay/debug field views;
- runtime hot reload hooks for GPU assets, material tables, and chunk meshes.

## Related docs

- [Getting started](GETTING_STARTED.md)
- [Provider selection authoring](PROVIDER_AUTHORING.md)
- [Block mutation authoring](BLOCK_MUTATION_AUTHORING.md)
- [Data/content asset workflow](DATA_CONTENT_ASSET_WORKFLOW.md)
- [Asset pipeline diagnostics](ASSET_PIPELINE_DIAGNOSTICS.md)
- [Asset inspector / devtools](ASSET_INSPECTOR_DEVTOOLS.md)
- [Asset/content hot reload](ASSET_CONTENT_HOT_RELOAD.md)
- [Friendly authoring diagnostics](FRIENDLY_DIAGNOSTICS.md)
- [Troubleshooting](TROUBLESHOOTING.md)
