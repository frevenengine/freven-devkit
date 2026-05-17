# Asset pipeline diagnostics

This document defines the DevKit-facing diagnostic contract for rc10 visual
assets and data-driven content.

It is the user-facing error model for texture, material, model, visual,
shader/effect, generated load-plan, and asset-file validation.

## Scope

This document answers:

- what information an asset/content diagnostic must include;
- how errors should be grouped by ownership boundary;
- how authoring-mode warnings differ from release-mode failures;
- how current engine/runtime errors should be translated into modder-friendly
  output;
- how future commands, inspectors, hot reload, and packaging checks should
  speak the same diagnostic language.

## Non-goals

This document does not itself implement the command that walks a resolved
content/assets graph. The current command surface is documented below and is
implemented by frevenengine/freven-boot#86.

This document does not fix the current `providers check` Material Registry v1
runtime-path failure. That concrete bug is tracked by frevenengine/freven-devkit#85.

This document does not implement the visual inspector, hot reload, or authoring
templates. Those are separate rc10 DevKit issues.

## Core rule

A useful DevKit asset diagnostic must tell the author:

- what failed;
- where the authored source is;
- which stable key is involved;
- which selected package/layer owns the failing declaration;
- what the validator expected;
- what value or file was actually found;
- which boundary owns the fix;
- one concrete example of a valid fix when possible.

Diagnostics must prefer stable author-facing identities over renderer internals.

Good diagnostic vocabulary:

```text
freven.vanilla:textures/block/stone
example.gems:materials/block/ruby_ore
content.manifest
content/materials/ruby_ore.toml
assets/textures/block/ruby_ore.png
layer "example.gems.visuals"
field material.base_color_texture
```

Bad diagnostic vocabulary for normal authors:

```text
atlas page 3 rect 12
texture array layer 41
Bevy Handle<Image>
wgpu bind group
renderer material slot 7
generated cache path as source truth
```

Renderer-internal ids may appear only under an explicit debug/devtools section
after the author-facing source and key are already shown.

## Diagnostic shape

Every asset/content diagnostic should have this conceptual shape:

```text
severity: error | warning | info | hint
code: stable diagnostic code
category: manifest | config | content | asset | dependency | override | backend | cache | packaging | runtime-bridge
summary: short human-readable problem
source_file: package-relative or absolute authoring file path
source_field: field/path inside the source file, when known
key: stable namespace:path key, when known
referenced_key: dependency key, when relevant
owner: product/experience/mod/content-pack id and selected layer
expected: accepted schema/value/policy
actual: value/file/metadata found
fix: concrete next step
docs: relevant guide
```

Future machine-readable output may use JSON, but the text output should stay
readable in a terminal and in CI logs.

## Severity

### Error

The selected experience/stack cannot be considered valid.

Examples:

- missing required texture file;
- invalid namespaced key;
- duplicate effective key without an explicit override rule;
- material references a missing texture;
- model references a missing material slot;
- unsupported source format in release mode;
- invalid alpha/render-layer combination when strict policy requires failure.

### Warning

The content can still run, but the author should fix it.

Examples:

- oversized but allowed texture in authoring mode;
- opaque material has unexpected alpha;
- optional PBR field is preserved but not supported by the current renderer;
- fallback/debug path was used instead of full texture/material rendering.

### Info

Useful resolved-state output.

Examples:

- inferred texture profile;
- selected override winner;
- generated cache was invalidated and rebuilt;
- fallback policy was selected.

### Hint

A follow-up suggestion that is not a validation failure.

Examples:

- “run content/assets check before launch”;
- “delete generated cache if source files changed outside DevKit”;
- “this looks like config, but should probably be authored content”.

## Strictness modes

DevKit tooling should support at least:

| Mode | Purpose |
| --- | --- |
| `authoring` | Fast iteration, warnings allowed when deterministic fallback exists. |
| `release` | Packaging/CI, stricter failures for incomplete or unsupported assets. |

A diagnostic should explain whether the result depends on strictness mode.

Example:

```text
warning FVK-TEXTURE-SIZE-PERF
texture example.gems:textures/block/ruby_ore is 256x256 for profile voxel_block
accepted in authoring mode, but may fail product release policy if max voxel block texture size is 128x128
```

## Categories

### Manifest diagnostics

The fix belongs in `mod.toml`, `experience.toml`, `experience.stack.toml`, or
`product.toml`.

Examples:

- missing declared content root;
- invalid package id;
- invalid dependency or layer reference;
- unsafe path outside the package boundary.

### Config diagnostics

The fix belongs in `config.schema.toml` or active selected config.

Asset/content definitions should not be moved into config just to silence an
asset diagnostic.

### Content diagnostics

The fix belongs in authored content data.

Examples:

- invalid material key;
- duplicate model key;
- block visual references missing material;
- generated family creates conflicting keys;
- invalid patch/merge operation.

### Asset file diagnostics

The fix belongs in resource bytes or package-local asset paths.

Examples:

- missing PNG;
- unsupported file extension;
- unreadable file;
- hash mismatch;
- invalid image dimensions;
- bad color-space metadata.

### Dependency diagnostics

The fix belongs in the declaration that references another key, or in the
package/layer that should provide the missing key.

Examples:

- material references missing texture;
- model references missing material;
- visual references missing model;
- shader/effect references unsupported capability.

### Override and patch diagnostics

The fix belongs in layering/override/patch declarations.

Examples:

- duplicate key without explicit override;
- patch target does not exist;
- content patch tries to write renderer-internal ids;
- cosmetic override used where selected server policy forbids it.

### Backend/capability diagnostics

The content is valid authoring data, but the selected runtime/backend cannot
realize it fully.

Examples:

- unsupported optional PBR field;
- unsupported shader/effect capability;
- current renderer uses fallback debug tint for a material.

### Generated cache diagnostics

The fix is to rebuild or invalidate generated output, not to edit cache files.

Examples:

- stale atlas/load-plan fingerprint;
- generated texture-array output missing;
- cache derived from old source hash.

### Runtime bridge diagnostics

The fix may be in temporary bridge setup between current runtime systems and
the long-term asset pipeline.

Example:

- material-key block visuals require a resolved Material Registry v1 before
  runtime SDK attach.

Runtime bridge diagnostics must say whether the author can fix the content or
whether the command/tool is currently limited.

## Required rc10 visual diagnostic cases

DevKit-friendly asset diagnostics must cover at least these cases:

| Problem | Diagnostic must include |
| --- | --- |
| Missing texture key | material/visual key, field, referenced texture key, owner layer, suggested declaration |
| Missing material key | visual/model/block key, field or slot, referenced material key, owner layer |
| Missing model key | visual key, field, referenced model key, owner layer |
| Missing texture file | texture key, expected package-relative path, manifest/source file |
| Bad image format | texture key, path, accepted formats |
| Unsupported texture size | texture key, actual dimensions, active profile, accepted dimensions |
| Invalid namespace/path | file, field, bad value, expected `namespace:path` |
| Duplicate effective key | key, winning layer if any, conflicting layer/file |
| Alpha/render-layer mismatch | material key, alpha mode, render layer, expected pairing |
| Material references missing texture | material key, field, texture key |
| Model references missing material | model key, slot/field, material key |
| Invalid asset path | file, field/path, bad path, package boundary rule |
| Renderer-internal id in authored data | file, field, forbidden value, stable-key replacement |
| Generated cache used as source | file, field/path, cache path, source path expectation |

## Text output format

Human-facing text should be compact but complete.

Example missing texture file:

```text
error FVK-ASSET-MISSING-TEXTURE
texture key: example.gems:textures/block/ruby_ore
declared in: core_experiences/example.game/content.manifest
expected file: content/textures/block/ruby_ore.png
selected layer: example.game
fix: add the PNG at that path, or update the texture declaration path.
```

Example material dependency failure:

```text
error FVK-MATERIAL-MISSING-TEXTURE
material key: example.gems:materials/block/ruby_ore
field: texture
missing texture key: example.gems:textures/block/ruby_ore
declared in: core_experiences/example.game/content.manifest
fix: declare the texture key, fix the typo, or change the material texture field.
```

Example alpha/render-layer mismatch:

```text
warning FVK-MATERIAL-ALPHA-LAYER
material key: example.glass:materials/block/blue_glass
alpha mode: blend
render layer: opaque
expected: blend materials normally use render_layer = "transparent"
fix: change render_layer to "transparent", or change alpha_mode to "opaque".
```

Example bridge limitation:

```text
error FVK-RUNTIME-BRIDGE-MATERIAL-REGISTRY
command: providers check
selected experience: freven.vanilla
reason: material-key block visuals require a resolved Material Registry v1
fix: use the content/assets check command once available, or run launch-time validation; this providers check path should not present this as a content authoring error.
tracked by: frevenengine/freven-devkit#85
```

## Suggested fix snippets

When possible, diagnostics should include a minimal valid snippet.

Example:

```toml
[[textures]]
key = "example.gems:textures/block/ruby_ore"
path = "textures/block/ruby_ore.png"
sha256 = "<64 lowercase hex chars>"

[[materials]]
key = "example.gems:materials/block/ruby_ore"
texture = "example.gems:textures/block/ruby_ore"
fallback_debug_tint_rgba = 3223335167
```

Snippets should be small and should not pretend to be the full final schema
when the current bridge uses a transitional manifest shape.

## Ordering

Diagnostics should be deterministic.

Recommended order:

1. selected product/experience/stack resolution errors;
2. manifest/config boundary errors;
3. content schema errors;
4. duplicate/conflict errors;
5. dependency graph errors;
6. asset file/hash/decode errors;
7. backend/capability warnings;
8. generated cache info/hints.

Within one category, sort by stable key and then by source path.

## Relationship to current implementation

Current engine/runtime code already has low-level validation errors for:

- content manifest parsing and validation;
- material registry construction;
- visual asset load-plan construction;
- texture file existence and sha256 checks;
- material-key block visual validation;
- renderer/material bridge diagnostics.

Those errors should be preserved, but DevKit commands should translate them
into this author-facing shape instead of exposing only internal build/runtime
phrasing.

## Command surface

`freven_boot content-assets check` validates the resolved rc10 visual asset load
plan for the selected experience or stack.

```bash
./freven_boot content-assets check --instance <instance> --experience <experience_id>
```

Use it when authored texture, material, model, shader/effect, block visual, or
content patch data does not load.

`freven_boot content-assets explain` prints the resolved graph:

```bash
./freven_boot content-assets explain --instance <instance> --experience <experience_id>
```

The explain output includes selected content layers, effective assets,
dependency edges, shadowed overrides, internal slots, and the load-plan
fingerprint.

This command is the preferred pre-launch path for asset/content validation. Do
not route visual asset diagnosis through `providers check` unless you are
specifically debugging provider declarations.

## Relationship to other rc10 DevKit issues

- frevenengine/freven-boot#86 implements the current `content-assets check` and
  `content-assets explain` command surface for resolved visual load plans.
- frevenengine/freven-devkit#85 should fix the current `providers check`
  Material Registry v1 failure path.
- frevenengine/freven-devkit#88 should display the same diagnostic fields in
  the inspector/devtools UI.
- frevenengine/freven-devkit#91 should use the same diagnostic codes during
  asset/content hot reload.
- frevenengine/freven-devkit#87 should generate templates that avoid these
  diagnostics by default.

## Done criteria for this diagnostic layer

This diagnostic layer is present when DevKit users and future tooling agree on:

- stable diagnostic categories;
- severity and strictness behavior;
- required source/key/layer/fix fields;
- the boundary that owns each kind of fix;
- examples for common texture/material/model failures;
- relationship to content/assets check, inspector, hot reload, and packaging.
