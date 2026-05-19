# Content authoring profiles

Freven has two layers of content authoring:

1. the canonical Freven content graph;
2. game-owned authoring profiles that compile or expand into that graph.

The canonical graph is the stable backend boundary used by engine/runtime
systems. A game-owned profile is a friendlier source format selected by a game,
mode, experience, or modding ecosystem.

This keeps the engine flexible:

- Vanilla can provide a Vintage-Story-style blocktypes/worldproperties/shapes
  workflow;
- another game can provide a Factorio-like prototype workflow;
- another game can provide a Minecraft-like resource-pack convention;
- a standalone product can use the canonical graph directly;
- a custom game can define its own profile without changing engine runtime code.

## Canonical graph vs authoring profile

The canonical graph contains low-level semantic declarations such as:

- textures;
- materials;
- models;
- block visual bindings;
- content families;
- block tags;
- future canonical gameplay/content declarations.

A game-owned authoring profile may contain friendlier source files such as:

- blocktypes;
- worldproperties;
- shapes;
- prototypes;
- recipes;
- loot tables;
- texture aliases;
- by-type rules;
- game-specific behavior references.

The profile compiler maps the friendly source into canonical declarations.

The engine consumes the canonical graph. It must not hardcode Vanilla's
blocktype schema, folder layout, or modding conventions.

## Command workflow

Boot exposes the profile bridge through:

    freven_boot content compile \
      --instance <instance> \
      --experience <experience_id> \
      --profile <profile_id> \
      --explain

The default profile is:

    freven.core:canonical_manifest_v1

That profile is a passthrough for low-level canonical manifests. It validates the
resolved canonical graph and explains the compile boundary, but it does not need
to generate new source files because the authored files are already canonical.

For profile compilers that support materialized output, write mode is explicit:

    freven_boot content compile \
      --instance <instance> \
      --experience <experience_id> \
      --profile <profile_id> \
      --output <output_dir> \
      --write

Generated output is not the source of truth unless the selected profile says so.
For normal game-owned profiles, source files remain under the game/profile-owned
authoring layout, and generated canonical files are rebuildable output.

After compiling or validating the profile boundary, use the existing graph tools:

    freven_boot content-assets check --instance <instance> --experience <experience_id>
    freven_boot content-assets explain --instance <instance> --experience <experience_id>
    freven_boot content-assets inspect --instance <instance> --experience <experience_id>
    freven_boot content-assets inspect --instance <instance> --experience <experience_id> --kind generated

Texture hash maintenance remains:

    freven_boot content-assets update-sha --instance <instance> --experience <experience_id>
    freven_boot content-assets update-sha --instance <instance> --experience <experience_id> --write

## Vanilla profile

The Vanilla profile id is:

    freven.vanilla:blocktypes_v1

The target Vanilla layout is:

    content/
      blocktypes/
        rock.toml
        soil.toml
        grass.toml
        glass.toml
      worldproperties/
        rock.toml
        fertility.toml
        grass_coverage.toml
        color.toml
      shapes/
        block/
          cube.toml
          cube_faces.toml
          topsoil.toml
          glass_framed.toml
      textures/
        terrain.toml
        tint.toml
      tags/
        terrain.toml

This profile is Vanilla-owned. It is not an engine-global convention.

A Vanilla mod should usually target the Vanilla profile when it wants to edit or
add Vanilla-style blocks. A low-level content pack may target the canonical graph
directly when it intentionally wants exact material/model/visual declarations.

## Current rc10 state

The current rc10 Vanilla content has already moved away from one giant manifest
into semantic modular source files. Those files are still canonical manifest
source organized by meaning, not the full high-level Vanilla profile compiler.

That means:

- `content/blocktypes/*.toml` in current Vanilla are semantic canonical source
  files;
- `freven.vanilla:blocktypes_v1` source is the high-level Vanilla blocktype path;
- `freven_boot content compile --profile freven.vanilla:blocktypes_v1` compiles it into canonical generated output;
- the engine should still only see the canonical content graph.

## Choosing the right layer

Use a game-owned profile when:

- the selected game documents one;
- the source files are easier for that game's modders;
- you want by-type rules, variant groups, or profile-local conventions;
- diagnostics should point at game concepts like blocktype/worldproperty/shape.

Use the canonical graph directly when:

- you are building a tiny content pack;
- you need exact texture/material/model/visual control;
- the game has no higher-level profile;
- you are authoring reusable low-level visual assets;
- you are testing engine/runtime content contracts.

Do not put renderer/runtime ids, atlas coordinates, GPU handles, generated cache
paths, or engine slots in author-facing profile source.

## How mods know what profile they target

The selected experience, stack, or modding guide should document the profile id.

For Vanilla-style mods, use:

    freven.vanilla:blocktypes_v1

For low-level canonical content, use:

    freven.core:canonical_manifest_v1

Future templates may include profile metadata in their README or experience
source. Until then, the profile id should be passed explicitly to
`freven_boot content compile`.

## Diagnostics authors should expect

Good profile diagnostics should report:

- selected experience or stack;
- selected profile id;
- source file;
- source kind, such as blocktype, worldproperty, shape, or prototype;
- source field path;
- generated canonical declaration kind;
- generated canonical key;
- first and second source files for duplicate generated keys;
- the next focused command.

Example:

    error: duplicate generated material key
    profile: freven.vanilla:blocktypes_v1
    source: content/blocktypes/soil.toml
    field: textures.grass_top
    generated declaration: materials
    key: freven.vanilla:block/grass_normal_top
    first source: content/blocktypes/grass.toml
    second source: content/blocktypes/soil.toml
    fix: move the shared material to a common source file or rename the generated key

Example unsupported profile:

    error: duplicate generated key 'freven.vanilla:stone'
    fix: rename one source `code` or generated target so each canonical key has one owner

## Relationship to templates

This repository currently documents template contracts. Packaged DevKit templates
may live in the Boot distribution.

Template docs should distinguish:

- project templates for experience/mod/stack shape;
- asset templates for low-level canonical content examples;
- game-owned profile templates for friendly game-specific authoring.

Do not add fake template files here just to satisfy a docs issue. If the repo
does not contain a checked-in template catalog, document the intended template
contract and point to the packaged DevKit/Boot template location.

## Related docs

- [Data/content asset workflow](DATA_CONTENT_ASSET_WORKFLOW.md)
- [Modular content authoring](MODULAR_CONTENT_AUTHORING.md)
- [Asset authoring templates](ASSET_AUTHORING_TEMPLATES.md)
- [Project templates](PROJECT_TEMPLATES.md)
- [Engine vs Vanilla ownership](ENGINE_VANILLA_OWNERSHIP.md)
- [Troubleshooting](TROUBLESHOOTING.md)
- [Vanilla blocktype modding](VANILLA_BLOCKTYPE_MODDING.md)
