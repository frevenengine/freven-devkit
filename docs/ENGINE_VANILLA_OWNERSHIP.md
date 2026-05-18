# Engine vs Vanilla ownership

Freven DevKit rc10 keeps the engine, SDK, Vanilla, standalone products, mods,
and content packs separated by ownership boundary.

This matters most for shaders, materials, textures, block visuals, models,
visual style, fallback assets, and generated renderer/cache output.

## Rule of thumb

- Engine owns generic rendering/runtime capability.
- SDK owns stable author-facing contracts and keys.
- Vanilla owns first-party gameplay/content/style.
- Standalone products may replace Vanilla completely.
- Mods and content packs add or override through declared dependencies, selected
  experience layers, and content/asset manifests.

Do not move Vanilla style into engine code just because Vanilla is currently the
bundled reference experience.

Do not move renderer internals into authored content just because they are useful
for debugging.

## Ownership map

| Layer | Owns | Must not own |
| --- | --- | --- |
| Engine/runtime | renderer implementation, asset loading, material registry resolution, fallback rendering, generated renderer/cache output, diagnostics plumbing | Vanilla block library, Vanilla visual style, mod-authored schemas, stable author namespaces |
| SDK | public schemas, stable `namespace:path` keys, guest APIs, material/model/visual/effect contract vocabulary | renderer slots, Bevy/wgpu handles, Vanilla content, product-specific style |
| DevKit / Boot | validation commands, inspectors, templates, packaging, author-facing diagnostics, resolved load-plan explanation, content compile command surface | first-party content policy, renderer implementation, hidden gameplay defaults, engine-global Vanilla schema |
| Vanilla | first-party blocks, tags, materials, textures, visual style, default providers, reference gameplay integration | engine/platform behavior, mandatory base for all games, SDK schemas |
| Standalone product | product identity, bundled default experience, product-owned content/style/providers/config | implicit dependency on Vanilla, reuse of shipped proof experience as authoring contract |
| Mod | runtime behavior, providers, config schema, package identity, declared content/assets | unrestricted engine access, silent ownership of another namespace |
| Content pack | data/assets only: textures, materials, models, visuals, tags, recipes, patches where allowed | executable behavior, renderer handles, runtime ids, hidden side effects |
| Generated cache | rebuildable atlas/load-plan/material-table/shader-cache output | authored source of truth |
| Save/world state | persisted runtime world/session/player state | shipped content defaults, generated cache, package manifests |

## Visual asset rule

Author-facing visual identity is always a stable key:

    namespace:path

Examples:

    freven.vanilla:textures/stone
    freven.vanilla:block/stone
    example.pack:materials/block/polished_stone
    example.total:effects/tonemap_default

Renderer/backend identity is never authored source:

- renderer material slot;
- atlas coordinate;
- texture-array layer;
- GPU handle;
- Bevy entity/component handle;
- shader module handle;
- pipeline id;
- generated cache path.

Inspectors may show these values only as debug/devtools output.

## Engine-owned

Engine/runtime may provide:

- renderer backends;
- asset file loading;
- texture/material/model/effect resolution;
- Material Registry v1 bridge;
- fallback/debug rendering when authored assets are missing or unsupported;
- generated load plans, atlases, texture arrays, material tables, and shader
  caches;
- runtime invalidation hooks for future live reload;
- low-level diagnostics that DevKit turns into author-facing messages.

Engine/runtime must stay generic. It should not hardcode that stone, grass,
humanoid controls, Vanilla lighting, Vanilla foliage motion, or Vanilla material
names are special engine concepts.

## SDK-owned

SDK owns the stable vocabulary that authors and guests can rely on:

- schema versions;
- `namespace:path` key shape;
- material/model/visual/effect contract fields;
- guest API helpers;
- capability and fallback vocabulary;
- validation model vocabulary.

SDK must not expose renderer-local ids as normal authoring API. If a debug or
legacy helper still accepts a low-level value, docs must label it as transitional
or debug-only.

## Vanilla-owned

Vanilla owns first-party Freven gameplay/content/style:

- `freven.vanilla:*` blocks and providers;
- default first-party textures and materials;
- first-party block tags;
- first-party movement/controller/control defaults;
- reference integration patterns;
- Vanilla-specific visual style choices.

Vanilla is allowed to be bundled by a Vanilla product/profile. It is not a
required dependency of the engine, SDK, or standalone products.

## Standalone-owned

A standalone product may ship a zero-Vanilla experience.

It should own:

- product id/name;
- default bundled experience;
- product-owned content/assets;
- product-owned provider defaults;
- product-owned visual style;
- product packaging and launch profile.

A standalone game should not copy `freven.vanilla` just to get a starting visual
style. Use project templates, asset authoring templates, and the standalone
product generator instead.

## Mods and content packs

Mods and content packs may extend or override selected content through declared
dependencies, selected stack layers, and content/assets manifests.

Valid examples:

- a Vanilla texture pack replacing `freven.vanilla:textures/grass` through an
  allowed selected layer;
- a material pack adding `example.pack:materials/block/marble`;
- a block mod registering gameplay block keys and pairing them with authored
  material keys;
- a total conversion replacing the selected base experience instead of depending
  on Vanilla.

Invalid examples:

- writing renderer slots into content files;
- depending on atlas coordinates as public API;
- putting Vanilla material libraries in engine code;
- using generated cache as authored source;
- silently taking ownership of another namespace without declared override policy.

## Shaders and effects

Shader/effect ownership follows the same split:

- Engine owns shader loading, compilation, backend resources, fallback shader
  behavior, and generated shader cache.
- SDK owns the effect contract vocabulary, capability names, and stable effect
  keys.
- Vanilla owns Vanilla-specific effect choices and style presets.
- Standalone products may choose a different style/effect library.
- Mods/content packs may request semantic effects inside allowed policy.

Raw shader source or renderer plugins are not normal content by default. They
require explicit future trust/product policy.

## Fallback assets

Fallbacks are engine/runtime behavior, not Vanilla content.

A missing material may fall back to a debug tint so authors can see the problem,
but that fallback does not mean the engine owns the final material. The owning
content layer must still declare the texture/material/model/effect source.

## Debug output

DevKit tools may show internal slots for diagnostics:

    freven_boot content-assets inspect --instance <instance> --experience <experience_id> --show-internal

Internal slots are not stable author-facing identity. Do not copy them into
authored content.

## Diagnostic flow

When a visual/content problem appears, use focused commands:

    freven_boot content-assets check --instance <instance> --experience <experience_id>
    freven_boot content-assets explain --instance <instance> --experience <experience_id>
    freven_boot content-assets inspect --instance <instance> --experience <experience_id>
    freven_boot content-assets watch --instance <instance> --experience <experience_id> --once

If provider defaults or worldgen selection are involved:

    freven_boot providers list --instance <instance> --experience <experience_id>
    freven_boot providers explain --instance <instance> --experience <experience_id>

Fix the owning source:

- `mod.toml` for package identity/capability/schema references;
- `experience.toml` or `experience.stack.toml` for selected mods/defaults/content
  roots;
- `content.manifest` and `content/` for authored content/visual declarations;
- asset files for images/models/shaders/sounds/fonts;
- generated cache only by rebuilding it;
- world/save state only through migration/runtime tools.

## Long-term contract

New visual/data features should pass these checks:

- Can a zero-Vanilla standalone game use this without Vanilla?
- Can Vanilla express its style through authored content instead of engine code?
- Can a mod/content pack add or override through declared layers?
- Are stable keys used instead of renderer ids?
- Are generated caches rebuildable and not source?
- Do diagnostics point to the owning file/field?

If any answer is no, the feature is probably crossing an ownership boundary.

## Related docs

- [Getting started](GETTING_STARTED.md)
- [Project templates](PROJECT_TEMPLATES.md)
- [Data/content asset workflow](DATA_CONTENT_ASSET_WORKFLOW.md)
- [Asset authoring templates](ASSET_AUTHORING_TEMPLATES.md)
- [Content authoring profiles](CONTENT_AUTHORING_PROFILES.md)
- [Asset pipeline diagnostics](ASSET_PIPELINE_DIAGNOSTICS.md)
- [Asset inspector / devtools](ASSET_INSPECTOR_DEVTOOLS.md)
- [Asset/content hot reload](ASSET_CONTENT_HOT_RELOAD.md)
- [Friendly authoring diagnostics](FRIENDLY_DIAGNOSTICS.md)
- [Troubleshooting](TROUBLESHOOTING.md)
