# Block Mutation Pipeline v1 authoring

This document describes the DevKit-facing authoring guidance for Block Mutation
Pipeline v1.

The goal is to make simulation-style mods practical without forcing authors to
emit huge numbers of scalar block edits every tick.

Examples include:

- cellular automata;
- infection / spreading blocks;
- machines and block networks;
- redstone-like systems;
- falling-sand-style simulations;
- large scripted structure edits.

## Ownership

The public contract lives in `freven-sdk`.

DevKit releases pin a compatible engine/sdk/vanilla set. Authors should treat
the DevKit manifest as the source of truth for which SDK contract is available
in a given DevKit build.

Runtime block mutations are authoritative server output. They are not client
visual effects and should not be used as a client-only prediction surface.

## Mutation shapes

Use the smallest shape that honestly represents the edit.

### `SetBlock`

Use scalar `SetBlock` for one-off player actions or very small edits.

Good examples:

- placing one block;
- breaking one block;
- toggling one machine output block;
- applying a tiny number of sparse edits.

### `SetBlocks`

Use `SetBlocks` for compact sparse batches.

Good examples:

- a cellular automata tick where only a small dirty set changed;
- infection spreading to scattered nearby cells;
- a machine network changing a bounded list of endpoints.

Keep ordering deterministic. Prefer stable world-position order when building
the edit list.

### `FillBox`

Use runtime `FillBox` for contiguous rectangular regions.

`FillBox` uses half-open world-cell bounds, matching the worldgen
`WorldTerrainWrite::FillBox` convention:

~~~text
min is inclusive
max is exclusive
~~~

For example, `min=(0, 64, 0)` and `max=(16, 65, 16)` covers one 16x16 layer.

Prefer `FillBox` over many individual edits for:

- vertical runs;
- rectangular clears/fills;
- generated rooms/platforms;
- block replacement passes over contiguous regions.

## Replace policy

Bulk mutations should be explicit about what they are allowed to replace.

Common patterns:

- only fill air when growing/spreading into empty space;
- replace only a known previous block id for deterministic simulation updates;
- use scalar `SetBlock.expected_old` for single-cell compare-and-set style edits.

Avoid broad unconditional replacement unless the mod truly owns that region.

## Budget model

The host validates runtime block mutation output before applying it.

Authors should design against these cost dimensions:

- mutation command count;
- expanded cell count;
- touched section count;
- touched column count;
- serialized callback result size;
- runtime call budget / timeout policy.

A small number of commands can still be expensive. For example, one large
`FillBox` may expand to many cells and many touched sections.

A large number of commands can also be expensive even if each command touches
one cell.

## Rejection behavior

Over-budget or invalid mutation output must be treated as a recoverable authoring
/ runtime-output rejection.

The intended contract is:

- no partial apply for a rejected batch;
- future ticks/lifecycle calls should continue when the rejection is recoverable;
- diagnostics should identify the budget or validation reason;
- authors should lower per-tick work, coalesce edits, or split work across ticks.

Do not depend on an over-budget callback disabling the mod. Use explicit mod
state to track unfinished work and continue later.

## Replication behavior

The server owns replication shape.

Authors should not depend on whether the server sends scalar `BlockUpdate`,
batched `BlockUpdateBatch`, or a future section/column delta internally.

The stable behavior authors can rely on is:

- applied authoritative block changes become visible to clients;
- prediction reconciliation is gated by causal terrain revision;
- large batches are coalesced by the host instead of requiring authors to manage
  network packet shape.

## Recommended simulation pattern

For simulation-style mods:

1. Keep simulation state in the mod.
2. Compute a bounded dirty set each tick.
3. Sort edits deterministically.
4. Emit `SetBlocks` for sparse dirty edits.
5. Emit `FillBox` for contiguous runs or regions.
6. Cap per-tick work in mod config.
7. Carry remaining work into future ticks instead of emitting an unbounded batch.
8. Log or expose diagnostics when work is throttled.

Prefer this shape:

~~~text
simulate -> collect dirty cells -> coalesce runs/boxes -> emit bounded mutations
~~~

Avoid this shape:

~~~text
scan whole world every tick -> emit one SetBlock per cell -> hope the host accepts it
~~~

## Example: sparse dirty cells

Conceptual low-level output shape:

~~~rust
use freven_block_guest::{BlockEdit, BlockMutation, BlockMutationBatch};
use freven_block_sdk_types::BlockRuntimeId;
use freven_world_guest::RuntimeOutput;

let edits = vec![
    BlockEdit {
        pos: (10, 64, 10),
        block_id: BlockRuntimeId(7),
        expected_old: None,
    },
    BlockEdit {
        pos: (11, 64, 10),
        block_id: BlockRuntimeId(7),
        expected_old: None,
    },
];

let output = RuntimeOutput {
    blocks: BlockMutationBatch {
        mutations: vec![BlockMutation::SetBlocks { edits }],
    },
    ..Default::default()
};
~~~

Prefer higher-level helpers from `freven_world_guest_sdk` when available for the
same contract shape.

## Example: contiguous fill

Conceptual low-level output shape:

~~~rust
use freven_block_guest::{BlockMutation, BlockMutationBatch, BlockReplacePolicy};
use freven_block_sdk_types::BlockRuntimeId;
use freven_world_guest::RuntimeOutput;

let output = RuntimeOutput {
    blocks: BlockMutationBatch {
        mutations: vec![BlockMutation::FillBox {
            min: (0, 64, 0),
            max: (16, 65, 16),
            block_id: BlockRuntimeId(7),
            replace: BlockReplacePolicy::OnlyAir,
        }],
    },
    ..Default::default()
};
~~~

This fills a 16x16 one-block-high layer, but only where the target cells are air.

## Checklist before reporting a bug

When a simulation mod behaves unexpectedly, include:

- DevKit version and manifest commits;
- mod runtime type: Wasm, external, native, or compile-time;
- mutation shapes used: `SetBlock`, `SetBlocks`, `FillBox`;
- approximate command count and expanded cell count per tick;
- whether the edits are sparse or contiguous;
- relevant runtime logs / rejection diagnostics;
- whether lowering the per-tick budget makes the issue disappear.
