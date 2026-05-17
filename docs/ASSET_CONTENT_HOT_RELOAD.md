# Asset/content hot reload

DevKit ships a conservative rc10 hot-reload workflow through:

    freven_boot content-assets watch

This is a development-only edit/check loop for visual content. It watches the
resolved content roots and manifests for the selected experience or stack,
rebuilds the visual asset load plan after edits, reports changed files, prints
reload fingerprints, and keeps running after broken reload attempts.

It does not yet perform live GPU texture replacement, material-table swapping,
chunk-mesh invalidation, or in-game block-under-cursor refresh. Running sessions
may still require restart until runtime hot reload hooks are added.

## Commands

Watch continuously:

    freven_boot content-assets watch --instance <instance> --experience <experience_id>

Use a shorter poll interval:

    freven_boot content-assets watch --instance <instance> --experience <experience_id> --interval-ms 250

Run one watch pass and exit:

    freven_boot content-assets watch --instance <instance> --experience <experience_id> --once

## What it reports

Each watch pass reports:

- selected experience or stack;
- development-only reload mode;
- watched content/manifests/source files;
- changed files;
- content/assets validation status;
- visual load-plan fingerprint;
- whether the fingerprint changed;
- reload errors without crashing the watcher;
- whether restart or future runtime hooks may still be required.

## Safe reload contract

Current rc10 behavior is conservative:

- texture/material/model/content edits are revalidated without restarting the
  watch process;
- broken reload attempts print the same FVK diagnostic vocabulary used by
  `content-assets check`;
- the watcher remains active for the next edit;
- production and release paths remain deterministic and do not depend on hot
  reload state;
- runtime live swap is explicitly not treated as available until engine/runtime
  hooks own GPU/resource/chunk invalidation.

## Recommended workflow

1. Start from an asset authoring template.
2. Run `content-assets check` once.
3. Run `content-assets inspect` for the key you are editing.
4. Start `content-assets watch`.
5. Edit textures/materials/model or visual source files.
6. Use the watch output to see changed files, diagnostics, and fingerprints.
7. Restart a running client/server when the watcher says runtime live swap is not
   attached.

## Relationship to other commands

Use:

- `content-assets check` for pre-launch validation;
- `content-assets explain` for the full resolved load plan;
- `content-assets inspect` for a focused key/kind view;
- `content-assets watch` for an edit/check loop while authoring visual content.

## Future runtime hooks

Future engine/runtime hot reload can build on this workflow for:

- live texture upload replacement;
- material table rebuilds;
- model/block visual reload;
- affected chunk mesh invalidation;
- inspector hot reload event display;
- in-game overlays showing which assets changed.

## Related docs

- [Data/content asset workflow](DATA_CONTENT_ASSET_WORKFLOW.md)
- [Asset pipeline diagnostics](ASSET_PIPELINE_DIAGNOSTICS.md)
- [Asset authoring templates](ASSET_AUTHORING_TEMPLATES.md)
- [Asset inspector / devtools](ASSET_INSPECTOR_DEVTOOLS.md)
- [Mod DevTools v1](MOD_DEVTOOLS.md)
- [Troubleshooting](TROUBLESHOOTING.md)
