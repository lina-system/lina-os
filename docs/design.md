# LINA OS design (moved from LINA Core)

Source: thisisjun786/lina `docs/design/runtime.md` (LINA OS composition) and `product-families.md` (installation modes and profiles) at d4207da0. Core keeps only what Core needs: it runs on any Linux.

## LINA OS composition

LINA OS is Linux based on [Omarchy](https://github.com/basecamp/omarchy) 4. It composes verified LINA Core, Node and LINA APP artifacts, pinned by manifest with version and digest, together with an operating, update and recovery environment. Its profiles are defined in [product-families.md](https://github.com/thisisjun786/lina/blob/dev/docs/design/product-families.md).

- LINA OS may offer to install Node.js, which the Claude adapter needs, as it offers a coding tool. It is the user's installed tool (see Installed tools); LINA OS neither pins nor owns it.
- LINA OS may offer to install a coding tool such as Codex for the user, which LINA then runs as a worker. It is the user's installed tool (see Installed tools); LINA OS neither pins nor owns it.
- An Omarchy root snapshot or an Omarchy update is never a restore or a readiness proof for LINA Core. The recovery units of LINA OS are defined in [filesystem.md](https://github.com/thisisjun786/lina/blob/dev/docs/design/filesystem.md).
- Omarchy 4's desktop shell is one Quickshell-based shell that provides the bar, the launcher and notifications. Its active theme is `~/.local/state/omarchy/current/theme/colors.toml`, which third-party apps follow through templates in `~/.config/omarchy/themed/` and hooks in `~/.config/omarchy/hooks/theme-set.d/`.
- Omarchy's default packages lack `fcitx5-hangul`. LINA OS adds it with its Hangul input setup, so Korean input works in the LINA app and the terminal. The desktop entry of the LINA app on LINA OS sets the input-method environment of the shell explicitly ([surfaces.md](https://github.com/thisisjun786/lina/blob/dev/docs/design/surfaces.md#korean-input)).

How LINA APP and the TUI adapt to Omarchy is defined in [surfaces.md](https://github.com/thisisjun786/lina/blob/dev/docs/design/surfaces.md).


## Installation modes

There are two intended installation modes. Both use the Linux filesystem of [filesystem.md](https://github.com/thisisjun786/lina/blob/dev/docs/design/filesystem.md).

- Device-OS install. LINA OS on a machine the user also uses. LINA works inside the user's environment, so the non-competition rules of [non-competition.md](https://github.com/thisisjun786/lina/blob/dev/docs/design/non-competition.md) apply. Non-competition is the responsibility of this install, not of Node.
- VM install. LINA OS as a virtual machine on an existing Mac, Windows or Linux host. This is LINA's own environment. Its permissions stay inside the VM sandbox and never reach the host. To act on the host or on any other user device, LINA uses a Node installed on that device.

A desktop install is not a third mode. It is the LINA app and Node installed on an existing Mac, Windows or Linux machine and connected to the main of a device-OS or VM install.

The LINA app and Node install independently. Installing the LINA app never requires Node, and installing Node never requires the LINA app. Both connect to the one main.

LINA OS has three profiles:

- Main. LINA Core, Node and the LINA app in one install, with exactly one update owner.
- Headless main. The main profile with the local LINA app autostart turned off. Users reach it through a remote LINA app. Headless never makes the LINA app a second brain.
- Connected device. Node and the LINA app, connected to the main. It never starts a second LINA Core.

Each mode and profile is built from one of the runtime compositions in [runtime.md](https://github.com/thisisjun786/lina/blob/dev/docs/design/runtime.md), which also defines what stays available when a piece is missing.

