# Modular content authoring

Modular content manifests let large content packs stay readable without changing
the semantic content model.

The root `content.manifest` is an explicit deterministic source index. Included
files are authored source files. They are not generated cache, not runtime ids,
and not renderer-internal data.

Use this workflow when one manifest becomes hard to review, when multiple
authors are editing different content areas, or when a content pack naturally
splits into textures, materials, models, visuals, families, and tags.

## Recommended layout

For a standalone experience or content-pack style project:

    experiences/example.visual_pack/
      experience.toml
      content.manifest
      content/
        textures/
          terrain.toml
          stone.png
          dirt.png
        materials/
          terrain.toml
        models/
          blocks.toml
        visuals/
          blocks.toml
        families/
          rocks.toml
          soil_grass.toml
        tags/
          terrain.toml

The experience still selects the same content boundary:

    [content]
    root = "content"
    manifest = "content.manifest"

The root manifest should stay small and explicit:

    schema = 1
    includes = [
      "content/textures/terrain.toml",
      "content/materials/terrain.toml",
      "content/models/blocks.toml",
      "content/visuals/blocks.toml",
      "content/families/rocks.toml",
      "content/tags/terrain.toml",
    ]

Include paths are resolved relative to the file that declares them. In the
current DevKit layout above, `content.manifest` sits next to `content/`, so the
root includes usually start with `content/...`.

If a manifest file inside `content/families/rocks.toml` includes another nearby
file, that include is relative to `content/families/rocks.toml`.

## Single-file vs modular

Keep a single `content.manifest` when:

- the pack is tiny;
- it has one or two declarations;
- the content is a throwaway prototype;
- splitting would make review harder, not easier.

Split into modular files when:

- the manifest mixes textures, materials, models, visuals, families, and tags;
- texture/material churn causes large unrelated diffs;
- generated family declarations make the root file hard to scan;
- Vanilla or a standalone product needs a stable long-term reference layout;
- different people own different content areas.

Both forms expand into the same semantic content model. The runtime should not
expose include files as ids, renderer slots, atlas pages, or cache paths.

For game-specific high-level source formats, use [Content authoring profiles](CONTENT_AUTHORING_PROFILES.md). A profile may compile friendlier source such as Vanilla blocktypes/worldproperties into the same canonical graph, while modular manifests remain the low-level canonical source format.

## Include rules

Includes are explicit ordered paths, not filesystem traversal.

Allowed:

    includes = [
      "content/textures/terrain.toml",
      "content/materials/terrain.toml",
    ]

Not allowed:

    includes = ["../other_pack/content.manifest"]
    includes = ["/absolute/path/content.manifest"]
    includes = ["content/generated/cache.toml"]
    includes = ["content/textures/"] # directory include

Rules:

- no hidden directory scanning;
- no glob imports;
- no absolute paths;
- no `..` traversal;
- no directory includes;
- no content-root escapes;
- no include cycles;
- no generated cache as authored source.

The order in `includes = [...]` is the authoring order. Do not rely on filesystem
listing order.

## Texture sha256 workflow

Texture hashes remain strict because they support reproducible validation,
fingerprints, packaging, and stale-asset diagnostics. Authors should not compute
or paste hashes by hand.

After adding or editing texture bytes, run:

    freven_boot content-assets update-sha \
      --instance <instance> \
      --experience <experience_id>

That dry-run prints what would change. To update authored source files:

    freven_boot content-assets update-sha \
      --instance <instance> \
      --experience <experience_id> \
      --write

Useful filters:

    freven_boot content-assets update-sha \
      --instance <instance> \
      --experience <experience_id> \
      --missing-only \
      --write

    freven_boot content-assets update-sha \
      --instance <instance> \
      --experience <experience_id> \
      --stale-only \
      --write

The tool uses declared texture entries only. It does not import random files
from the filesystem. For modular manifests, it updates the included source file
that owns the texture declaration instead of rewriting the root index.

## Validation loop

Use this loop while authoring:

    freven_boot content-assets update-sha --instance <instance> --experience <experience_id>
    freven_boot content-assets update-sha --instance <instance> --experience <experience_id> --write
    freven_boot content-assets check --instance <instance> --experience <experience_id>
    freven_boot content-assets explain --instance <instance> --experience <experience_id>
    freven_boot content-assets inspect --instance <instance> --experience <experience_id>
    freven_boot content-assets watch --instance <instance> --experience <experience_id> --once

Use `inspect --kind generated` for content family output:

    freven_boot content-assets inspect \
      --instance <instance> \
      --experience <experience_id> \
      --kind generated

## Diagnostics authors should expect

Good diagnostics should identify:

- selected experience or stack;
- selected content layer;
- root manifest path;
- included source file path;
- include stack when there is a cycle;
- semantic kind and key, such as texture/material/model/visual/family/tag;
- field path when available, such as `textures[].sha256`;
- old and new sha256 when a texture hash is stale;
- the next focused command.

Common examples:

    error: missing include
    root manifest: experiences/example.visual_pack/content.manifest
    include: content/textures/terrain.toml
    fix: create the file, remove the include, or correct the path.

    error: include cycle
    include stack:
      content.manifest
      content/families/rocks.toml
      content/families/common.toml
      content/families/rocks.toml
    fix: make one file own the shared declarations and include it from only one direction.

    error: duplicate semantic key
    key: example.visual:textures/block/stone
    first source: content/textures/terrain.toml
    second source: content/textures/overrides.toml
    fix: remove the duplicate, rename one key, or use the proper layer override mechanism.

    error: stale sha256
    texture key: example.visual:textures/block/stone
    declared in: content/textures/terrain.toml
    asset path: content/textures/stone.png
    fix: run content-assets update-sha --write after intentionally changing the texture bytes.

## Vanilla reference

The eventual Vanilla split should follow this same shape: a small explicit root
manifest plus stable included files for terrain textures, materials, models,
visuals, families, and tags.

Vanilla is a reference for first-party content organization, not a special engine
path. Mods and standalone products should use the same author-facing rules.

## Related docs

- [Data/content asset workflow](DATA_CONTENT_ASSET_WORKFLOW.md)
- [Asset authoring templates](ASSET_AUTHORING_TEMPLATES.md)
- [Asset pipeline diagnostics](ASSET_PIPELINE_DIAGNOSTICS.md)
- [Content family authoring](CONTENT_FAMILY_AUTHORING.md)
- [Asset inspector / devtools](ASSET_INSPECTOR_DEVTOOLS.md)
- [Troubleshooting](TROUBLESHOOTING.md)
