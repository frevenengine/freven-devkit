# Provider selection authoring

Freven provider selection has three separate steps:

1. a mod registers one or more provider keys;
2. an experience selects provider keys in its defaults;
3. the runtime validates that selected keys exist for the side that hosts them.

This page documents the author-facing contract for worldgen, character
controller, and client control providers.

## Provider kinds

Freven currently exposes these provider families:

| Provider kind | Experience default field | Hosted by |
| --- | --- | --- |
| Worldgen | `defaults.worldgen` | server |
| Character controller | `defaults.character_controller` | server and client |
| Client control provider | `defaults.client_control_provider` | client |

The same experience may be resolved for both server and client. Validation is
side-aware:

- server validates `worldgen` and `character_controller`;
- client validates `character_controller` and `client_control_provider`.

This lets a client-only provider default exist without making dedicated server
startup fail, and lets a server-only worldgen default exist without making a
network client fail before it connects.

## Selecting explicit provider defaults

Provider defaults live in `experience.toml` or in an
`experience.stack.toml` layer defaults table.

Example standalone experience:

~~~toml
schema = 1
id = "example.experience"
version = "1.0.0"
title = "Example Experience"

[[mods]]
id = "example.providers"
version = "^1.0"

[defaults]
worldgen = "example.providers:flat"
character_controller = "example.providers:humanoid"
client_control_provider = "example.providers:controls"
~~~

Example stack layer override:

~~~toml
schema = 1
id = "example.stack"
version = "1.0.0"
title = "Example Stack"
base = "freven.vanilla"

[[layers]]
id = "example.stack.providers"
title = "Provider Override"

[layers.defaults]
worldgen = "example.providers:flat"
character_controller = "example.providers:humanoid"
client_control_provider = "example.providers:controls"

[[layers.mods]]
id = "example.providers"
version = "^1.0"
~~~

If the selected provider comes from a mod already inherited from the base
experience, the stack layer only needs `[layers.defaults]`. If the selected
provider comes from a runtime-loaded mod added by the stack, add that mod under
`[[layers.mods]]` in the same layer.

Do not put top-level `[defaults]`, top-level `[[mods]]`, top-level `[config]`,
top-level `[content]`, or top-level `[requirements]` in
`experience.stack.toml`. Those are standalone `experience.toml` fields. Stack
overlays use `[[layers]]` with `[layers.defaults]`, `[[layers.mods]]`,
`[layers.config."<mod_id>"]`, and optional `[layers.content]`.

Use explicit defaults for production experiences. Implicit fallback still selects
the first registered provider in that family, but explicit keys are the stable
long-term contract: they survive provider registration order changes and fail
honestly when a provider was removed, renamed, or not loaded.

## Combining multiple provider mods in a stack

A stack can combine providers from the base experience and from one or more
runtime-loaded mods added by the overlay layer.

Example: keep `freven.vanilla` as the base, add one Wasm worldgen mod, and add a
second Wasm mod that owns movement/control providers:

~~~toml
schema = 1
id = "example.vanilla.terrain_flight"
version = "0.1.0"
title = "Vanilla + Custom Terrain + Flight"
base = "freven.vanilla"

[[layers]]
id = "example.vanilla.terrain_flight.layer"
version = "0.1.0"
title = "Terrain + Flight Providers"

[layers.defaults]
worldgen = "example.terrain:terrain"
character_controller = "example.flight:controller"
client_control_provider = "example.flight:controls"

[[layers.mods]]
id = "example.terrain"
version = "^0.1"

[[layers.mods]]
id = "example.flight"
version = "^0.1"
~~~

In this shape:

- `example.terrain` and `example.flight` are active because they are listed under
  `[[layers.mods]]`;
- the selected provider keys are active because they are selected under
  `[layers.defaults]`;
- the base experience remains the owner of inherited content, requirements, and
  any inherited mod refs;
- a root-level `[[mods]]` entry in `experience.stack.toml` is not the stack
  overlay contract.

After editing the stack, run:

~~~bash
./freven_boot providers explain --instance <instance> --experience example.vanilla.terrain_flight
./freven_boot providers check --instance <instance> --experience example.vanilla.terrain_flight
~~~

`providers explain` shows which defaults are selected and which loaded mod owns
each provider key. `providers check` fails before launch when a selected provider
default does not resolve for the hosted side.


## Inspecting provider catalogs

Use the DevKit provider inspection commands before launching an authored
experience.

List known provider keys:

~~~bash
./freven_boot providers list \
  --instance instances/player \
  --experience example.experience
~~~

Validate selected defaults:

~~~bash
./freven_boot providers check \
  --instance instances/player \
  --experience example.experience
~~~

Explain defaults, hosted sides, and provider ownership:

~~~bash
./freven_boot providers explain \
  --instance instances/player \
  --experience example.experience
~~~

On Windows:

~~~powershell
.\freven_boot.exe providers check `
  --instance instances\player `
  --experience example.experience
~~~

## Runtime guest providers

Runtime-loaded guest provider declarations may require launch-time guest
negotiation before their full provider catalog is known.

In that case `providers check` may report a partial/deferred status instead of a
full static answer. Launch-time validation still rejects an explicit selected
default if the hosted runtime side cannot resolve it.

## Failure model

If an explicit provider default names a missing key, Freven fails instead of
silently falling back to another provider.

For example, this is invalid when no loaded server-side mod registers
`example.missing:worldgen`:

~~~toml
[defaults]
worldgen = "example.missing:worldgen"
~~~

Run:

~~~bash
./freven_boot providers check \
  --instance instances/player \
  --experience example.experience
~~~

Expected result: the check reports that the selected worldgen provider default
did not resolve for the server runtime.

## Recommended authoring loop

1. Add or update provider registration in the mod.
2. Select the provider key explicitly in the experience defaults.
3. Run `freven_boot providers list` to confirm the key appears in the catalog.
4. Run `freven_boot providers check` to validate hosted-side defaults.
5. Run `freven_boot providers explain` when debugging ownership, side, or
   deferred runtime-guest behavior.
6. Launch with `freven_boot play`, `freven_boot serve`, or `freven_boot run`.

Provider selection is part of the public DevKit authoring contract: a provider
key is not just an implementation detail, it is the stable name that experiences
and stacks select.
