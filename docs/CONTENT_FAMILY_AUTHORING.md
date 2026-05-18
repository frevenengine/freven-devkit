# Content family authoring

Content families are compact authoring declarations for generated content
variants.

Use them when many content entries share the same shape but differ by one or
more variant axes, such as rock type, soil type, grass coverage, or glass color.

Current DevKit support is backed by:

- freven-engine content family expansion;
- freven-boot `content-assets check`;
- freven-boot `content-assets inspect --kind generated`;
- asset authoring templates shipped under `templates/asset_authoring_templates/`.

## Core rule

A family is source. Generated entries are derived semantic content.

Flow:

    authored content.manifest
      [[families]]
        axes / allow / skip / templates
            |
            v
    load/build-stage family expansion
            |
            v
    ordinary semantic entries
      materials / models / block visuals / block tags
            |
            v
    runtime consumers
      material registry / visual load plan / block visual mesh table / tag registry

Do not author or depend on renderer/runtime internals in family source:

- no runtime block ids;
- no renderer slots;
- no atlas coordinates;
- no GPU handles;
- no generated cache paths as source truth.

Use stable `namespace:path` keys.

## Expansion order

The author-facing order is:

1. resolve the selected experience or stack;
2. collect effective content layers;
3. load each `content.manifest`, including explicit modular `includes = [...]` files;
4. validate authored declarations, including `[[families]]`;
5. expand families into normal semantic entries;
6. validate generated references after expansion;
7. apply the existing layer/override/load-plan rules used by runtime consumers.

Family expansion is not renderer-time logic and not Vanilla-specific hardcoding.

Patch/merge semantics still operate on stable semantic content keys. A generated
entry should be treated like any other material/model/visual/tag once expansion
has completed. If a selected layer wants to override generated output, it should
override the stable generated key intentionally rather than referring to runtime
internals.

## Starter templates

DevKit packages include family-focused asset templates:

| Template | Purpose |
| --- | --- |
| `rock_family_pack` | Generates rock materials, block visuals, and a rock tag from a `rock` axis. |
| `soil_grass_family_pack` | Generates soil x grass-coverage variants and demonstrates skipped combinations. |
| `colored_glass_family_pack` | Generates transparent colored-glass materials and visuals from structured color metadata. |

Copy one into an instance:

    cp -R templates/asset_authoring_templates/rock_family_pack \
      instances/dev/experiences/example.asset.rock_family_pack

Then validate it:

    freven_boot content-assets check \
      --instance instances/dev \
      --experience example.asset.rock_family_pack

Inspect generated output:

    freven_boot content-assets inspect \
      --instance instances/dev \
      --experience example.asset.rock_family_pack \
      --kind generated

Inspect one family:

    freven_boot content-assets inspect \
      --instance instances/dev \
      --experience example.asset.rock_family_pack \
      --kind generated \
      --key example.asset.rock_family_pack:families/rock

## What `check` catches

`content-assets check` validates both authored source and generated output.

It should catch:

- invalid family keys;
- invalid axis names or values;
- unknown allow/skip axis references;
- contradictory allow/skip combinations;
- generated key conflicts;
- missing generated texture/material/model/visual dependencies;
- bad render-layer or tint metadata after expansion.

Family diagnostics should include the family key and, where relevant, axis
values such as `color=red` or `grass=covered, soil=sand`.

## What `inspect --kind generated` shows

The generated inspector is author-facing. It lists:

- authored family key;
- owning content layer;
- source manifest path;
- axes and values;
- allow/skip rule counts;
- generated material keys;
- generated model keys;
- generated block visual keys;
- generated block tag keys.

It intentionally does not expose renderer slots, runtime ids, atlas positions,
or GPU handles as authoring API.

## Skipped and allowed combinations

Use `[[families.skip]]` to omit invalid combinations.

Example:

    [[families.skip]]
    grass = "covered"
    soil = "sand"
    reason = "covered sand intentionally omitted"

Use `[[families.allow]]` when the authored family should include only a small
subset of the cartesian product.

If a combination is both allowed and skipped, `content-assets check` should fail
with the family key, axis values, and skip reason.

## Framed glass and advanced model families

Current rc10 family templates focus on entries that are already represented in
the stable manifest path: materials, models, block visuals, and block tags.

For framed-glass-style authoring today:

- start from `block_model_pack` for model/visual structure;
- use `colored_glass_family_pack` for transparent color-family material/visual
  generation;
- keep custom model source files as authored content until richer per-part model
  family templates are stabilized.

A dedicated framed-glass family template should be added only when model-family
templates can express the required per-part/cuboid structure without hiding
renderer or mesh internals in author source.

## Related docs

- [Data/content asset workflow](DATA_CONTENT_ASSET_WORKFLOW.md)
- [Modular content authoring](MODULAR_CONTENT_AUTHORING.md)
- [Asset authoring templates](ASSET_AUTHORING_TEMPLATES.md)
- [Asset pipeline diagnostics](ASSET_PIPELINE_DIAGNOSTICS.md)
- [Asset inspector / devtools](ASSET_INSPECTOR_DEVTOOLS.md)
- [Mod DevTools v1](MOD_DEVTOOLS.md)
- [Troubleshooting](TROUBLESHOOTING.md)
