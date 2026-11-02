# LINA OS definition

LINA OS is a manifest-only Linux distribution built around [LINA](https://github.com/lina-system/lina) on top of [Omarchy](https://omarchy.org), with Hangul input set up from the start. It ships no component code of its own: [`manifest/lina-os.toml`](../manifest/lina-os.toml) pins released artifacts (`lina`, `lina-app-web`, `ocx`, `omo`, `postgres`) across three profiles ([`main`](../profiles/main/README.md), [`headless`](../profiles/headless/README.md), [`device`](../profiles/device/README.md)), and LINA OS supplies only the OS layer around them. Tracked on the roadmap ([#1](https://github.com/lina-system/lina-os/issues/1), M0).

## Base

- **Omarchy baseline (decided, reversible):** Stock [Omarchy](https://github.com/omacom/omarchy) `v4.0.6` (latest stable release, identified by git tag rather than the `version` file) plus `fcitx5-hangul` and a Korean `fcitx5` profile. Move the floor to `4.5` once a stable `4.5` release is out (`v4.5.0-rc2` already adds `fcitx5-hangul` to base packages and `omarchy setup input hangul` with `TriggerKeys = Hangul`).
- **Hangul input on `v4.0.6`:** Stock Omarchy `v4.0.6` ships `fcitx5`, `fcitx5-gtk`, `fcitx5-qt`, `noto-fonts-cjk`, `omarchy-fcitx5.service` (tied to `graphical-session.target`), and input-method environment variables (`10-omarchy-fcitx.conf`), but no Hangul engine or active profile. On `v4.0.6`, LINA OS installs `fcitx5-hangul` and ships a Korean `fcitx5` profile (`keyboard-us` + `hangul`, switched by `Hangul` / Right Alt and `Super + I`). Once stable `4.5` is adopted, this overlay is dropped in favor of stock Omarchy's built-in Korean input setup.
- **What stock Omarchy already provides:** Docker (`docker`, `docker-buildx`, `docker-compose`, `docker.socket`), UFW (`deny incoming` with `ufw-docker`), a `uwsm` Wayland session (`graphical-session.target`) with `systemd-oomd` on `app.slice`, and the Quickshell notification server (`omarchy.notifications` over the standard freedesktop D-Bus API).
- **Profiles ([`profiles/`](../profiles/)):**
  - [`main`](../profiles/main/README.md): Omarchy desktop with Hangul input, LINA core, LINA web app, OCX, omo, the Postgres + pgvector container managed by LINA, and desktop integration.
  - [`headless`](../profiles/headless/README.md): LINA core, LINA web app served over the network, OCX, omo, and the Postgres + pgvector container, with no desktop installed.
  - [`device`](../profiles/device/README.md): A connected worker machine for a `main` LINA elsewhere, running `omo` and what handed-over work needs without its own LINA memory store.

## Install path

1. **Manifest-first on stock Omarchy (M1):** Nothing is built before LINA and the web app run on stock Omarchy. In M1, installation runs on an existing Omarchy machine or VM and reads [`manifest/lina-os.toml`](../manifest/lina-os.toml), where each component (`lina`, `lina-app-web`, `ocx`, `omo`, `postgres`) is pinned by `version`, `source` (release URL or container image reference), and `sha256` digest.
2. **Digest verification and placement:** The installer verifies each artifact's `sha256` against `manifest/lina-os.toml` before unpacking or pulling it, installs `fcitx5-hangul` and the Korean `fcitx5` profile on `main` (while on `v4.0.6`), and installs LINA's systemd user units into `/usr/lib/systemd/user/` (with `ConditionPathExists=` guards so units stay inert when a binary is absent), enabling them at first run as Omarchy enables its own units.
3. **Store initialization:** LINA pulls the pinned `[postgres]` (Postgres + pgvector) image and starts its own container bound to `127.0.0.1` with password authentication and schemas `memory`, `ctx`, `ltm`, and `vec` (not Omarchy's dev helper `omarchy-install-docker-dbs`, which runs `postgres:18` with `POSTGRES_HOST_AUTH_METHOD=trust`).
4. **Headless and device boot:** On `headless` and `device`, setup enables `loginctl enable-linger` for the LINA user (or system units) so services start at boot without a graphical login, and opens only the explicitly configured network port for `headless` through UFW.
5. **Images (`Later`):** Standalone installer ISOs and prebuilt VM images for the three profiles come only after the M1 manifest install on stock Omarchy is working.

## Update path

- **Base OS updates:** Stock Omarchy updates the base packages and desktop through its normal updater and `~/.config/omarchy/hooks/post-update.d/` hooks. Rebasing from `v4.0.6` to stable `4.5` removes LINA OS's custom `fcitx5` profile overlay and switches to Omarchy's native Hangul setup.
- **Manifest artifact updates:** Updating LINA OS components means bumping `version`, `source`, and `sha256` in [`manifest/lina-os.toml`](../manifest/lina-os.toml), verifying the new digests, replacing the pinned artifacts, and restarting the `lina` and `lina-app-web` user units.
- **Store and Honcho migrations:** LINA owns upgrades of the `[postgres]` container and the Honcho server (`services/honcho/`), following LINA's upgrade sequence: whole-cluster backup → rehearsal migration → migration job → smoke test.
- **OS snapshot vs. LINA restore:** An OS snapshot covers the root filesystem, packages, and unit files so a broken OS or manifest update can roll back cleanly. An OS snapshot is not a LINA restore: LINA's Postgres + pgvector cluster and versioned workspace (`Desk`) are backed up and restored by LINA itself so rolling back the OS never silently rewinds or corrupts memory.

## The first difference from Omarchy plus the LINA app

Installing the LINA core and web app on stock Omarchy leaves four OS-level gaps that LINA OS resolves first (from [#4](https://github.com/lina-system/lina-os/issues/4)):

1. **Hangul input active at first login:** Stock Omarchy `v4.0.6` includes `fcitx5` only for the compose key. LINA OS adds `fcitx5-hangul` and an active Korean `fcitx5` profile from the start (until stable `4.5` provides this upstream).
2. **How LINA reaches Docker for Postgres + pgvector (waiting for the owner's choice — reversible):**
   Stock Omarchy enables `docker.socket` and `ufw-docker`, but intentionally keeps the user out of the `docker` group because it is root-equivalent (`sudo` is required unless the user runs `omarchy-setup-security-sudoless-docker`). LINA runs as the unprivileged user and must install, start, and upgrade its pinned `[postgres]` container.

   | Option | How it works | What it costs |
   | --- | --- | --- |
   | **`docker` group** | Add the user to `docker` (`omarchy-setup-security-sudoless-docker`) so LINA talks to `/run/docker.sock` directly. | **Root-equivalent privilege** for every process running as the user, undoing stock Omarchy's security default. |
   | **Rootless Docker (recommended)** | Run `dockerd-rootless` under the user's systemd manager (`DOCKER_HOST=unix://$XDG_RUNTIME_DIR/docker.sock`) using the `docker` and `docker compose` CLI Omarchy already installs. | Requires `docker-rootless-extras`, `/etc/subuid` and `/etc/subgid` provisioning, a per-user image store under `~/.local/share/docker`, and user-namespace network overhead. |
   | **Podman** | Run the `[postgres]` container rootlessly with `podman` (and `podman-docker` / Quadlet user units) without a root daemon. | Adds a second container engine alongside Omarchy's preinstalled Docker stack, `/etc/subuid` and `/etc/subgid` setup, and maintaining Podman compatibility in LINA's Docker tooling. |

   - **Recommendation:** **Rootless Docker**, because it preserves stock Omarchy's refusal to grant root-equivalent `docker` group access while keeping LINA on the upstream `docker` and `docker compose` CLI already shipped by Omarchy. Its cost is adding `docker-rootless-extras`, provisioning `subuid`/`subgid`, and running a user-scoped daemon with its own storage directory. *(Status: waiting for the owner's choice.)*
3. **Service lifecycle and isolation from Omarchy's Hermes integration:**
   - Stock Omarchy has no `loginctl enable-linger`, so user units stop at logout and never start on `headless` or `device`. LINA OS ships `lina` and `lina-app-web` user units and enables linger off the desktop.
   - LINA's core is a fork of Hermes Agent, while stock Omarchy ships its own Hermes Desktop installer and remover (`bin/omarchy-remove-ai-hermes`), which touches `~/.hermes` and stops and deletes `hermes-gateway*.service` user units. LINA OS keeps all binary names, state paths, and unit names under `lina*` (never `hermes`, `~/.hermes`, or `hermes-gateway*.service`) so Omarchy's Hermes scripts cannot overwrite or stop LINA.
4. **Notification rules:** Stock Omarchy's `omarchy.notifications` Quickshell plugin only bypasses Do Not Disturb for app name `omarchy-action` or `critical` urgency. LINA sends notifications over standard freedesktop D-Bus under its own app name, never impersonates `omarchy-action`, reserves `critical` urgency for alerts the user must see, and wires click actions to open the LINA web app.
