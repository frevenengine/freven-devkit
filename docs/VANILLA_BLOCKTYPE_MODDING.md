# Vanilla blocktype modding workflow

Vanilla blocktype authoring is the friendly, game-owned path for adding or
changing Vanilla-style blocks.

Use this workflow when you want to author blocks in the Vanilla content style
instead of writing the low-level canonical manifest graph by hand.

The profile id is:

    freven.vanilla:blocktypes_v1

This profile is owned by Vanilla. It is not an engine-global schema. The engine
and runtime consume the compiled canonical output, not the high-level Vanilla
source files.

## What this profile compiles

Vanilla blocktype source compiles into the canonical content graph consumed by
Boot, Engine, and the runtime:

- textures;
- materials;
- models;
- block visuals;
- content families;
- block tags;
- block shapes.

Block shapes are runtime semantics, not renderer geometry. They compile into
canonical `[[block_shapes]]` declarations with:

- `occludes`;
- `side_solid`;
- `collision_boxes`;
- `selection_boxes`.

Visual model `elements` describe how a block looks. They do not automatically
define collision, selection, side solidity, or occlusion.

In short: visual model `elements` do not automatically define collision.

## Recommended source layout

A Vanilla-style content layer should keep high-level source separate from
generated canonical output:

    content/
      blocktypes/
        my_block.toml
      shapes/
        block/
          cube.toml
      worldproperties/
        rock.toml
      _compiled/
        vanilla_blocktypes_v1/
          README.md
          blocktypes/
          families/
      manifest.d/
        compiled.generated.manifest
      content.manifest

The generated files are rebuildable output. Keep source files in
`blocktypes/`, `shapes/`, and `worldproperties/` as the author-owned files.

## Minimal cube block source

A simple texture-backed cube blocktype looks like this:

    profile = "freven.vanilla:blocktypes_v1"
    kind = "cube_block"
    code = "my_block"
    title = "My Block"

    shape = "freven.vanilla:shapes/block/cube"
    texture = "freven.vanilla:textures/my_block"
    material = "freven.vanilla:block/my_block"

    render_layer = "opaque"
    fallback_debug_tint_rgba = 2155905279

    tags = [
      "freven.vanilla:terrain",
    ]

The important long-term rules are:

- `code` becomes the stable generated Vanilla block key suffix;
- `texture` and `material` are stable namespaced content keys;
- `shape` points at a Vanilla-owned high-level shape source;
- `fallback_debug_tint_rgba` must match the generated material fallback tint;
- do not author renderer internal slots or runtime ids.

## Shape source semantics

A full cube shape source declares both visual geometry and physical/query
semantics.

Example shape fields:

    profile = "freven.vanilla:blocktypes_v1"
    kind = "shape"
    code = "block/cube"

    material_slots = ["all"]

    [[elements]]
    from = [0.0, 0.0, 0.0]
    to = [1.0, 1.0, 1.0]

    [elements.faces]
    all = "all"

    [occludes]
    bottom = true
    top = true
    north = true
    south = true
    east = true
    west = true

    [side_solid]
    bottom = true
    top = true
    north = true
    south = true
    east = true
    west = true

    [[collision_boxes]]
    min = [0.0, 0.0, 0.0]
    max = [1.0, 1.0, 1.0]

    [[selection_boxes]]
    min = [0.0, 0.0, 0.0]
    max = [1.0, 1.0, 1.0]

Box coordinates are normalized block-space AABBs. Each axis must satisfy:

    0.0 <= min < max <= 1.0

Use explicit boxes even when the visual geometry looks like the same cube. That
keeps future partial blocks, slabs, framed glass, foliage, and custom authored
shapes honest.

## Transparent blocks

Transparent visual rendering does not mean empty collision.

For example, glass can be:

- visually transparent;
- physically solid;
- selectable as a full cube;
- non-occluding for neighbor face culling.

That means the material/render layer can be transparent while `collision_boxes`
and `selection_boxes` still describe a full block. `occludes` can be false for
all sides while `side_solid` remains true where gameplay needs a solid side.

## Compile/check loop

After editing Vanilla-style source, run the profile compiler:

    freven_boot content compile \
      --instance <instance> \
      --experience <experience_id> \
      --profile freven.vanilla:blocktypes_v1 \
      --explain

Then validate texture hashes and generated canonical content:

    freven_boot content-assets check \
      --instance <instance> \
      --experience <experience_id>

When texture bytes change, update hashes explicitly:

    freven_boot content-assets update-sha \
      --instance <instance> \
      --experience <experience_id> \
      --write

Then check again:

    freven_boot content-assets check \
      --instance <instance> \
      --experience <experience_id>

## Inspect generated output

Use the generated output as a diagnostic view, not as the primary authoring
surface.

Useful checks:

    rg -n 'generated_from_profile|block_shapes|collision_boxes|selection_boxes|occludes|side_solid' \
      <experience>/content/_compiled/vanilla_blocktypes_v1 \
      <experience>/content/manifest.d

    freven_boot content-assets inspect \
      --instance <instance> \
      --experience <experience_id> \
      --kind generated

The generated output should show canonical `[[block_shapes]]` declarations for
authored blocktypes and families.

## Launch validation

After compile and asset checks pass, launch a visual validation or local
instance:

    freven_boot run client \
      --instance <instance> \
      --experience <experience_id>

For Vanilla visual validation stacks, use the stack/experience id documented by
the selected DevKit or Boot distribution.

## When to use the low-level canonical graph

Use `freven.core:canonical_manifest_v1` only when you intentionally want the
low-level content graph:

- reusable engine-neutral content packs;
- custom non-Vanilla game content;
- tests that need exact canonical declarations;
- advanced debugging of generated output.

For normal Vanilla-style blocks, use:

    freven.vanilla:blocktypes_v1

## Troubleshooting

### Duplicate generated keys

Symptom:

    duplicate generated key

Cause:

    Two high-level source files compile to the same canonical key.

Fix:

    Rename one `code`, material key, texture key, family id, or generated target.
    The diagnostic should point at both source files.

### Unsupported field

Symptom:

    unknown field
    unsupported field

Cause:

    The file is using a field that the selected profile schema does not own.

Fix:

    Check the selected profile. Vanilla blocktype source belongs to
    `freven.vanilla:blocktypes_v1`; low-level canonical manifest fields belong
    to `freven.core:canonical_manifest_v1`.

### Missing texture hash

Symptom:

    missing sha256
    sha256 mismatch

Cause:

    Texture bytes changed or a new texture was added without updating the
    manifest hash.

Fix:

    freven_boot content-assets update-sha \
      --instance <instance> \
      --experience <experience_id> \
      --write

    freven_boot content-assets check \
      --instance <instance> \
      --experience <experience_id>

### Unsupported profile

Symptom:

    unsupported content compile profile

Cause:

    The selected DevKit/Boot build does not include the requested profile, or
    the command is being run against the wrong distribution.

Fix:

    Check the profile id and the selected DevKit/Boot version. Vanilla-style
    blocktype authoring requires:

        freven.vanilla:blocktypes_v1

### Collision does not match visuals

Symptom:

    The block looks partial but collides or selects like a full cube, or the
    reverse.

Cause:

    Visual `elements` and runtime `collision_boxes` / `selection_boxes` are
    separate declarations.

Fix:

    Edit the shape source. Do not try to fix collision by changing renderer-only
    model geometry.
