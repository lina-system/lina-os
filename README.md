# LINA OS

A Linux distribution built around [LINA](https://github.com/lina-system/lina): an operating, update and recovery environment that composes verified LINA and [LINA app](https://github.com/lina-system/lina-app) releases, pinned by manifest.

> Exploration. Nothing here is required to use LINA. LINA and the app install on macOS, Windows and Linux without this project.

## Scope

- A base on [Omarchy](https://github.com/basecamp/omarchy), with Hangul input set up out of the box.
- Profiles: main (LINA and the app on one machine), headless main, connected device.
- Device install and VM install.
- Update, snapshot and recovery that never treat an OS snapshot as a LINA restore.
- Desktop integration: theme, bar, launcher, notifications and keybindings.

LINA OS ships no code of its own components. It pins released artifacts and adds the OS layer around them.

## License

Available under the Sustainable Use License ([LICENSE](LICENSE)). Third-party components keep their own licenses.
