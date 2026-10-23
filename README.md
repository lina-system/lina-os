# LINA OS

A Linux distribution built around [LINA](https://github.com/thisisjun786/lina): an operating, update and recovery environment that composes verified LINA Core, Node and [LINA app](https://github.com/thisisjun786/lina-app) releases, pinned by manifest.

> Exploration. Nothing here is required to use LINA. LINA Core runs on any Linux; the app and Node install on macOS, Windows and Linux without this project.

## Scope

- A base on [Omarchy](https://github.com/basecamp/omarchy), with Hangul input set up out of the box.
- Profiles: main (Core, Node and the app on one machine), headless main, connected device.
- Device install and VM install.
- Update, snapshot and recovery that never treat an OS snapshot as a LINA restore.
- Desktop integration: theme, bar, launcher, notifications and keybindings.

LINA OS ships no code of its own components. It pins released artifacts and adds the OS layer around them.

## License

Apache License, Version 2.0 ([LICENSE](LICENSE)).
