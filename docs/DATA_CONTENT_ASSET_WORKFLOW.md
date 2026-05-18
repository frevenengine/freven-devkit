# Data/content asset workflow

This document defines the DevKit-facing workflow for authored gameplay/content
data and referenced asset files.

It is the author-facing bridge between the SDK rc10 content contracts and the
current DevKit package/layout guidance.

## Core rule

Do not mix these categories:

- manifest: package identity, dependency, trust, execution, surfaces,
  entrypoint, capability requests, schema references;
- config schema: supported runtime settings and defaults;
- active config: selected runtime values for one experience or stack;
- content data: authored gameplay and visual definitions;
- asset files: resource bytes referenced by content declarations;
- generated cache: derived tooling/runtime output;
- save/world state: runtime-persistent world/player/session data.

A file may reference another category, but it should not become that category.
For example, `mod.toml` may reference `config.schema.toml`, but it is not active
runtime config. A material entry may reference a texture, but the texture file is
not the material definition. A generated atlas/load plan may be built from source
assets, but it is not authored source.

## What belongs where

### `mod.toml`

Use `mod.toml` for package metadata:

```toml
schema = 3
id = "example.weather"
version = "0.1.0"
artifact = "wasm_module"
execution = "wasm_guest"
trust = "sandboxed"
policy = "safe_guest"
surfaces = "server"
entry = "mod.wasm"
config_schema = "config.schema.toml"
```

Do not put active runtime values or authored gameplay definitions in `mod.toml`.

### `config.schema.toml`

Use `config.schema.toml` for settings that users/admins tune:

```toml
schema = 1

[[settings]]
key = "enabled"
type = "bool"
default = true
scope = "server_world"
reload = "runtime"
authority = "server"
```

Active values belong in the selected experience or stack:

```toml
[config."example.weather"]
enabled = true
```

or:

```toml
[layers.config."example.weather"]
enabled = true
```

### `content/` and `content.manifest`

Use content files for authored definitions:

```text
experiences/example.game/
  experience.toml
  content.manifest
  content/
    blocks/
    items/
    recipes/
    tags/
    materials/
    models/
    visuals/
    data/
```

Examples of content data:

- block/item/recipe/entity definitions;
- material/model/visual bindings;
- semantic tags and families;
- loot tables;
- dialogue data;
- NPC archetypes;
- schedules;
- location registries;
- encounter tables;
- product-authored gameplay definitions.

Current DevKit/Boot experience content is selected through:

    [content]
    root = "content"
    manifest = "content.manifest"

A stack should inherit base content unless a layer explicitly declares a content
overlay. A standalone/product-owned experience should declare its own content
root when it owns content.

### Modular content manifests

Small packs can keep all declarations in one `content.manifest`. Larger packs
should use the root manifest as an explicit deterministic source index:

    schema = 1
    includes = [
      "content/textures/terrain.toml",
      "content/materials/terrain.toml",
      "content/models/blocks.toml",
      "content/visuals/blocks.toml",
      "content/families/rocks.toml",
      "content/tags/terrain.toml",
    ]

Included files are authored source content. They are not generated cache,
runtime ids, renderer slots, or hidden imports.

Include paths are resolved relative to the declaring file. They are ordered
explicitly by the author; DevKit does not scan directories, expand globs, or use
filesystem order as content order.

Use [Modular content authoring](MODULAR_CONTENT_AUTHORING.md) for the full
layout, include rules, troubleshooting shape, and hash-update workflow.

### Asset files

Asset files are bytes referenced by content declarations.

Examples:

```text
content/
  textures/
    block/example_block.png
  models/
    block/example_model.toml
  shaders/
```

Long-term SDK contracts distinguish authored content definitions from asset
resource bytes. The exact root name may evolve with the asset/content resolver,
but the boundary stays the same: authored declarations are source, generated
atlases/load plans are cache, and renderer-internal ids are not author-facing
asset identities.

### Generated cache

Generated cache is derived output. It may contain load plans, validation indexes,
atlases, texture arrays, fingerprints, or compiled/transcoded assets.

Authors should not edit generated cache as source.

### Save/world state

Save/world state belongs under runtime world storage. It is not shipped content
defaults and should not be edited to define a mod's default content.

Authoritative content changes may require compatibility handling or migration.
That is separate from editing content source files.

## Current author workflow

1. Put executable behavior in a Wasm/native/external mod only when behavior is
   needed.
2. Put user/admin-tunable settings in schema-backed config.
3. Put authored gameplay/visual data in content source.
4. Reference resource files from content declarations.
5. Select the experience or stack that owns the content root.
6. Validate config and provider selection before launch.
7. Use launch/load-plan diagnostics for current content errors until the
   dedicated content/assets check command lands.

Commands available today:

```bash
./freven_boot config check --instance <instance> --experience <experience_id>
./freven_boot config explain --instance <instance> --experience <experience_id>
./freven_boot providers check --instance <instance> --experience <experience_id>
./freven_boot providers explain --instance <instance> --experience <experience_id>
./freven_boot content compile --instance <instance> --experience <experience_id> --profile freven.core:canonical_manifest_v1 --explain
./freven_boot content-assets check --instance <instance> --experience <experience_id>
./freven_boot content-assets explain --instance <instance> --experience <experience_id>
./freven_boot content-assets inspect --instance <instance> --experience <experience_id>
./freven_boot content-assets watch --instance <instance> --experience <experience_id>
./freven_boot content-assets update-sha --instance <instance> --experience <experience_id>
./freven_boot content-assets update-sha --instance <instance> --experience <experience_id> --write
```

Use `content compile` when a selected game/mode authoring profile needs to be validated or compiled into the canonical content graph. The default `freven.core:canonical_manifest_v1` profile is a passthrough for current canonical manifests; game-owned profiles such as `freven.vanilla:blocktypes_v1` are opt-in and owned by their game/profile.

Use `content-assets check` before launch when a texture, material, model, block
visual, or content patch does not load. It resolves the selected experience or
stack and validates the rc10 visual asset load plan without starting the full
client/server runtime.

Use `content-assets inspect --kind generated` when you need to inspect content
family expansion output before launch.

Use `content-assets explain` when you need to inspect resolved content layers,
effective assets, dependency edges, shadowed overrides, internal slots, and the
load-plan fingerprint.

Use `content-assets update-sha` after adding or editing texture bytes. The
default dry-run reports missing or stale hashes. Add `--write` to update the
authored manifest or included source file that owns the texture declaration.

Use `content-assets watch` during development when you want a conservative
edit/check loop for texture, material, model, visual, or content source changes.
The watcher validates changes and reports diagnostics/fingerprints, but runtime
live swap may still require restart until engine hot reload hooks are attached.

Friendly asset diagnostic output is defined in
[Asset pipeline diagnostics](ASSET_PIPELINE_DIAGNOSTICS.md).

For shader/material/visual-style ownership, use
[Engine vs Vanilla ownership](ENGINE_VANILLA_OWNERSHIP.md). The short rule is:
Engine resolves and renders, SDK defines stable contracts, Vanilla owns
first-party style, standalone products may replace Vanilla, and mods/content
packs override through declared selected layers.

## Example: data-backed Wasm mod

A server-side Wasm mod can provide behavior while authored content remains data:

```text
mods/example.weather/
  mod.toml
  config.schema.toml
  mod.wasm

experiences/example.weather_stack/
  experience.stack.toml
```

The stack selects the mod and its runtime config:

```toml
schema = 1
id = "example.weather_stack"
version = "0.1.0"
title = "Base + Weather"
base = "freven.vanilla"

[[layers]]
id = "example.weather_layer"
version = "0.1.0"

[[layers.mods]]
id = "example.weather"
version = "^0.1"

[layers.config."example.weather"]
enabled = true
```

If the same package also needs authored content data, do not hide it inside
`include_str!()` unless it is truly code-owned. Prefer a content source path that
the DevKit can inspect and validate.

## Example: product-owned standalone content

A standalone game that owns its content should keep it under its product-owned
experience:

```text
core_experiences/example.game/
  experience.toml
  content.manifest
  content/
    blocks/
    tags/
    materials/
    textures/
    data/
  mods/example.game.core/
    mod.toml
    mod.wasm
```

The experience declares:

```toml
[content]
root = "content"
manifest = "content.manifest"
```

The bundled core mod can provide behavior, providers, or runtime services. The
content root owns authored definitions and referenced resources.

## Canonical modular layout

For larger visual/content packs, prefer:

    experiences/example.visual_pack/
      experience.toml
      content.manifest
      content/
        textures/terrain.toml
        materials/terrain.toml
        models/blocks.toml
        visuals/blocks.toml
        families/rocks.toml
        tags/terrain.toml

The root `content.manifest` should stay small and list only `schema` plus
explicit `includes = [...]`. Vanilla's rc10 split should become the first-party
reference for this layout, but it should not be a special engine-only path.

## Common mistakes

### Putting gameplay definitions in config

Config should tune behavior. Content should define authored game data.

Bad fit for config:

- 200 NPC archetypes;
- quest graph definitions;
- loot table entries;
- block/material/model declarations;
- large schedule/location registries.

Good fit for config:

- enable/disable;
- small numeric limits;
- selected mode;
- debug options;
- reload policy controlled settings.

### Rebuilding Wasm for every data edit

Embedding data with `include_str!()` is acceptable for small prototypes, tests,
or code-owned constants. It should not become the default workflow for authored
content that modders, products, or future tools should inspect.

### Editing generated cache

Generated cache can be deleted and rebuilt. It is not source.

### Using save/world state as defaults

World saves persist runtime state. Shipped content defaults belong in package or
experience source files.

### Leaking renderer internals

Content/asset authoring should use stable namespaced keys and source paths, not
renderer slots, atlas coordinates, Bevy handles, or GPU ids.

## Relationship to SDK docs

The SDK defines the long-term contracts:

- architecture ownership;
- package boundaries;
- visual asset model;
- layered asset overrides;
- content patch/merge semantics;
- creator-friendly content schema;
- data-driven authoring layer;
- material, texture, model, variant, tint, lighting, and shader/effect contracts.

This DevKit document explains how authors should think about those contracts from
the packaged DevKit workflow.

## Done criteria for this workflow layer

This workflow layer is considered present when DevKit users can answer:

- where do I put package identity?
- where do I put runtime settings?
- where do I put authored gameplay/visual data?
- where do I put referenced resource files?
- what should never be edited as source?
- which current commands can I run before launch?
- which follow-up commands will provide deeper resolved content/asset checks?

## Content family expansion

Content families are compact source declarations in `content.manifest`. They
expand during the load/build stage into ordinary semantic entries before runtime
consumers build material registries, visual load plans, block visual mesh tables,
or tag registries.

Expansion order:

1. resolve selected experience/stack layers;
2. load authored content manifests;
3. validate family declarations;
4. expand axes, allowed combinations, skipped combinations, and templates;
5. validate generated materials/models/visuals/tags;
6. consume the expanded semantic entries through the normal content/assets
   pipeline.

Generated family output participates in the same stable-key override and
patch/merge rules as hand-authored content. Authors should override generated
semantic keys intentionally, not renderer/runtime internals.

See [Content family authoring](CONTENT_FAMILY_AUTHORING.md).
