# Troubleshooting

## DevKit can't find core experiences
Symptom: `experience 'freven.vanilla' was not found` or list is empty.

Fix:
- Run `freven_boot` from the extracted DevKit folder (where `core_experiences/` exists).
- If needed, set an override:
  - `FREVEN_CORE_EXPERIENCES_DIR=/path/to/core_experiences`

## `freven_boot --version` says manifest not found
Make sure you run from the DevKit folder that contains:
`manifest/devkit_manifest.toml`

Or set:
- `FREVEN_DEVKIT_MANIFEST=/path/to/devkit_manifest.toml`

## Client opens then exits quickly
Collect logs:
- `<instance>/logs/`
- `<instance>/run/` (load_plan JSON helps reproduce)

Try with:
- `RUST_LOG=info` (or `debug`)

## Vulkan / graphics issues (Linux)
Common causes:
- missing/old GPU drivers
- running in a remote session without GPU acceleration

Verify your Vulkan works in general (system-level). Then retry.

## Windows firewall / networking
If connecting to a dedicated server fails:
- allow `freven_server.exe` through firewall
- confirm the server is listening on the expected port (default 12806)
- confirm `--connect host:port` matches the server bind address

## QUIC trust issues (client)
If the client refuses to connect due to certificate/trust changes:
- safest: `--quic-forget <host:port>` then reconnect
- or reset trust store: `--quic-reset-trust`
- `--quic-accept-new-cert` should be used carefully (a changed cert can be a MITM or server reinstall)
