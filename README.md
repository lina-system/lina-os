# LINA OS

A Linux distribution built around [LINA](https://github.com/lina-system/lina), on top of [Omarchy](https://omarchy.org), with Hangul input set up from the start.

## Scope

- LINA OS ships no component code of its own. `manifest/lina-os.toml` pins released artifacts: the LINA core (Python, a fork of Hermes Agent), the [LINA app](https://github.com/lina-system/lina-app) for the web, OCX for models, omo for work, and the Postgres + pgvector image that LINA installs and manages in Docker. LINA OS adds only the OS layer around them.
- Three profiles in `profiles/`:
  - `main`: LINA and the app on one machine.
  - `headless`: LINA without a desktop.
  - `device`: a connected device that works for a main LINA.
- Install on a device or in a VM.
- Update, snapshot and recovery. An OS snapshot is not a LINA restore.
- Desktop integration: theme, bar, launcher, notifications and shortcuts.

LINA OS is an exploration. Nothing here is needed to use LINA, and nothing is built before LINA and the web app run on Omarchy. The definition is in [docs/definition.md](docs/definition.md); the roadmap is [#1](https://github.com/lina-system/lina-os/issues/1).

Versions follow `0.0.1-alpha.YYYYMMDD.N`.

## License

LINA OS is licensed under AGPL-3.0 ([LICENSE](LICENSE)).
