# Reporting Bugs

## Before you file
1) Reproduce on a clean instance if possible:
   - create a fresh instance folder and retry
2) Collect version info:
   - `freven_boot --version`

## What to include (required)
Attach or paste:

1) `freven_boot --version` output  
2) `manifest/devkit_manifest.toml` (from the DevKit folder)  
3) Logs:
   - client: `<instance>/logs/`
   - server: `<instance>/logs/`

Optional but extremely helpful:
- `<instance>/run/load_plan_*.json`
- Steps to reproduce (exact commands)
- GPU/OS info (especially for graphics issues)

## Where to file
Open an issue in this repository:
- Bug report
- Feature request

Security issues: see `SECURITY.md`.
