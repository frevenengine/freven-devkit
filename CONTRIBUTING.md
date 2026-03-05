# Contributing

Thanks for helping with Freven.

## Where contributions go

This repository (`freven-devkit`) is the **entry point**:
- documentation improvements
- bug reports / feature requests for the DevKit distribution and tooling

Actual code contributions typically go to:
- `freven-sdk` (public): SDK contracts, examples, docs
- `freven-vanilla` (public): Vanilla experience content & gameplay via the SDK

The engine and boot source repositories are not currently open for public PRs.

## How to help

- Improve docs: clarify setup, troubleshooting, mod/external runtime guides.
- File actionable bug reports (see `docs/REPORTING_BUGS.md`).
- Submit PRs to `freven-sdk` / `freven-vanilla` for fixes and enhancements.

## Release model (high level)

- DevKit binaries are built from internal/private sources.
- DevKit archives are published in this repository via **GitHub Releases**.
- Each DevKit release is pinned to specific `freven-engine`, `freven-sdk`, and `freven-vanilla` tags/commits (see `freven_boot --version` / manifest).
