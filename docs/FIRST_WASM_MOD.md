# First Wasm mod

This guide walks through a first runtime-loaded Freven Wasm world mod from scratch:

1. create a Rust Wasm guest project;
2. build the `.wasm` artifact;
3. install `mod.wasm` and `mod.toml` into a DevKit instance;
4. add the mod to an `experience.stack.toml`;
5. select the mod's worldgen provider;
6. validate the resolved provider catalog;
7. launch and inspect logs.

For a checked-in buildable example, see:

- `freven-sdk/examples/first_wasm_mod`

## Prerequisites

Install Rust and the Wasm target:

```bash
rustup target add wasm32-unknown-unknown
```

Extract a Freven DevKit release and verify it:

```bash
./freven_boot --version
```

For released DevKits, prefer pinning SDK dependencies to the `sdk_commit`
printed by `freven_boot --version`. That keeps the mod source aligned with the
DevKit runtime contract.

## Create the Rust project

Example layout:

```text
first_wasm_mod/
  Cargo.toml
  src/lib.rs
```

Example `Cargo.toml`:

```toml
[package]
name = "first_wasm_mod"
version = "0.1.0"
edition = "2024"
publish = false

[lib]
crate-type = ["cdylib"]

[dependencies]
# Replace <sdk_commit_from_freven_boot_version> with the sdk_commit printed by:
#   ./freven_boot --version
freven_world_guest_sdk = { git = "https://github.com/frevenengine/freven-sdk.git", rev = "<sdk_commit_from_freven_boot_version>", package = "freven_world_guest_sdk" }
```

During local SDK development, the checked-in SDK example uses a path dependency
instead:

```toml
freven_world_guest_sdk = { path = "../../crates/freven_world_guest_sdk" }
```

## Write the mod

Example `src/lib.rs`:

```rust
use freven_world_guest_sdk::{
    BlockDescriptor, InitialWorldSpawnHint, LifecycleResult, SectionY, StartContext, TickContext,
    WorldGenCallResult, WorldGenColumnBuilder, WorldGenContext,
};

const GUEST_ID: &str = "example.first_wasm";
const BLOCK_KEY: &str = "example.first_wasm:ground";
const WORLDGEN_KEY: &str = "example.first_wasm:flat";

fn ground_block() -> BlockDescriptor {
    BlockDescriptor::solid_colored_cube(0x3C7A_52FF)
}

fn start_server(ctx: StartContext<'_>) -> LifecycleResult {
    freven_world_guest_sdk::log_info!(
        "first wasm mod started experience={} mod={}",
        ctx.experience_id(),
        ctx.mod_id()
    );
    LifecycleResult::default()
}

fn tick_server(ctx: TickContext<'_>) -> LifecycleResult {
    if ctx.tick().is_multiple_of(120) {
        freven_world_guest_sdk::log_info!("first wasm mod heartbeat tick={}", ctx.tick());
    }
    LifecycleResult::default()
}

fn generate_worldgen(ctx: WorldGenContext<'_>) -> WorldGenCallResult {
    let ground = ctx
        .init()
        .block_id_by_key(BLOCK_KEY)
        .expect("first wasm worldgen requires its registered ground block");

    let mut column = WorldGenColumnBuilder::for_request(ctx.request());
    column.fill_section(SectionY::new(0), ground);

    let mut output = column.finish();

    if ctx.request().column.cx == 0 && ctx.request().column.cz == 0 {
        output.bootstrap.initial_world_spawn_hint = Some(InitialWorldSpawnHint {
            feet_position: [16.5, 32.0, 16.5],
        });
    }

    WorldGenCallResult { output }
}

freven_world_guest_sdk::wasm_guest!(
    guest_id: GUEST_ID,
    registration: {
        block: BLOCK_KEY => ground_block(),
        worldgen: WORLDGEN_KEY => generate_worldgen,
    },
    lifecycle: {
        start_server: start_server,
        tick_server: tick_server,
    },
);
```

This uses the public `freven_world_guest_sdk::wasm_guest!` path. New world-stack
mods should not hand-wire the low-level Wasm ABI unless they are testing the ABI
itself.

## Build the Wasm artifact

From the mod project root:

```bash
cargo +stable build --release --target wasm32-unknown-unknown
```

The artifact will be under:

```text
target/wasm32-unknown-unknown/release/first_wasm_mod.wasm
```

## Create a DevKit instance

From the extracted DevKit directory:

```bash
mkdir -p instances
./freven_boot init --instance instances/first_wasm client
```

## Install the mod files

Create the instance-local mod directory:

```bash
mkdir -p instances/first_wasm/mods/example.first_wasm
cp /path/to/first_wasm_mod/target/wasm32-unknown-unknown/release/first_wasm_mod.wasm \
  instances/first_wasm/mods/example.first_wasm/mod.wasm
```

Create `instances/first_wasm/mods/example.first_wasm/mod.toml`:

```toml
schema = 3
id = "example.first_wasm"
version = "0.1.0"
artifact = "wasm_module"
execution = "wasm_guest"
trust = "sandboxed"
policy = "safe_guest"
surfaces = "server"
entry = "mod.wasm"

[capabilities]
worldgen_max_linear_memory_bytes = 67108864
worldgen_max_call_millis = 100
worldgen_max_result_bytes = 1048576
allow_unstable = false
```

`mod.toml` is the package manifest and capability-request surface. Active
runtime config does not belong in `mod.toml`.

Use `surfaces = "server"` for this first worldgen-only mod. Use
`surfaces = "both"` only when the guest also needs to attach on the client side.

## Add a stack experience

Create:

```text
instances/first_wasm/experiences/example.first_wasm.stack/experience.stack.toml
```

with:

```toml
schema = 1
id = "example.first_wasm.stack"
version = "0.1.0"
title = "Vanilla + First Wasm Mod"
base = "freven.vanilla"

[[layers]]
id = "example.first_wasm.layer"
version = "0.1.0"
title = "First Wasm Worldgen"

[layers.defaults]
worldgen = "example.first_wasm:flat"

[[layers.mods]]
id = "example.first_wasm"
version = "^0.1"
```

This is the recommended first-mod composition path:

- `freven.vanilla` remains the base experience;
- the stack adds one runtime-loaded Wasm mod through `[[layers.mods]]`;
- `[layers.defaults]` selects the worldgen provider registered by the mod;
- no local `content/` or `content.manifest` copy is required.

Do not put root-level `[[mods]]` or root-level `[defaults]` in
`experience.stack.toml`. Stack overlays use `[[layers.mods]]` and
`[layers.defaults]`.

## Validate before launch

Inspect provider ownership:

```bash
./freven_boot providers explain \
  --instance instances/first_wasm \
  --experience example.first_wasm.stack
```

Validate selected defaults:

```bash
./freven_boot providers check \
  --instance instances/first_wasm \
  --experience example.first_wasm.stack
```

Expected result: `example.first_wasm:flat` resolves as the selected worldgen
provider for the server side.

## Launch singleplayer

```bash
./freven_boot play \
  --instance instances/first_wasm \
  --experience example.first_wasm.stack \
  -- --username dev --devtools
```

Look for log lines like:

```text
first wasm mod started experience=example.first_wasm.stack mod=example.first_wasm
first wasm mod heartbeat tick=...
```

## Dedicated server + client

Server instance:

```bash
./freven_boot init --instance instances/first_wasm_server server
```

Copy the same `mods/example.first_wasm/` directory and
`experiences/example.first_wasm.stack/` directory into the server instance.

Run the server:

```bash
./freven_boot serve \
  --instance instances/first_wasm_server \
  --experience example.first_wasm.stack
```

Client:

```bash
./freven_boot init --instance instances/first_wasm_player client

./freven_boot play \
  --instance instances/first_wasm_player \
  -- --connect 127.0.0.1:12806 --username dev --devtools
```

Because this first mod is server-only worldgen, the client does not need to
install the mod locally for this dedicated-server flow.

## Troubleshooting

### `mod.toml` schema error

Use current runtime-loaded Wasm manifest schema:

```toml
schema = 3
artifact = "wasm_module"
execution = "wasm_guest"
trust = "sandboxed"
policy = "safe_guest"
```

### Wasm artifact not found

Check that `entry = "mod.wasm"` matches the actual file beside `mod.toml`:

```text
instances/first_wasm/mods/example.first_wasm/mod.toml
instances/first_wasm/mods/example.first_wasm/mod.wasm
```

### Provider default does not resolve

Run:

```bash
./freven_boot providers explain --instance instances/first_wasm --experience example.first_wasm.stack
```

Then check:

- the mod is listed under `[[layers.mods]]`;
- the selected key is under `[layers.defaults]`;
- the key exactly matches the string registered in `src/lib.rs`;
- the mod id in `mod.toml` matches the `[[layers.mods]].id`.

### World did not change after editing worldgen

Existing worlds keep persisted bootstrap/world metadata. For a first test
instance, remove the old world and bootstrap:

```bash
rm -rf instances/first_wasm/worlds
rm -f instances/first_wasm/world_bootstrap.toml
```

Then launch again.

### Config does not appear in `StartInput.config`

Do not put active runtime config under `mod.toml [config]`.

Use:

- `[config."<mod_id>"]` in standalone `experience.toml`;
- `[layers.config."<mod_id>"]` in `experience.stack.toml`.

Use:

```bash
./freven_boot config check --instance <instance> --experience <experience_id>
./freven_boot config explain --instance <instance> --experience <experience_id>
```

### Which SDK crate should I use?

Use `freven_world_guest_sdk` for current gameplay/world-stack mods: blocks,
worldgen, world queries/mutations, character controllers, client-control
providers, and other world-facing runtime services.

Use `freven_guest_sdk` only for neutral platform-shaped guests that do not need
world-stack features.
