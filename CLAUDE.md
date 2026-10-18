# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository purpose

This is a personal reference repo, not a software project — there is no code to build, lint, or test. It documents one person's Linux desktop setup, for Arch Linux and Ubuntu:

- [README.md](README.md) — public-facing overview of the repo and a summary of what the runbook covers.
- [setup-arch.md](setup-arch.md) — a personal runbook of manual setup steps for a fresh Arch Linux install (post-install network/user/GNOME bootstrap, package installs, theming, dotfile snippets, and app configuration). It's written as an ordered checklist with inline shell commands and config file contents, not a script.
- [setup-ubuntu-26.04.md](setup-ubuntu-26.04.md) — the same runbook adapted for Ubuntu Desktop 26.04 LTS.
- [starship-dracula.toml](starship-dracula.toml) — a ready-to-use [Starship](https://starship.rs) prompt config implementing a Dracula color palette, referenced from the "Setup console" section of `setup-arch.md` (copy to `~/.config/starship.toml` to activate).

## Working in this repo

- `setup-arch.md` is consumed by a human running commands manually on a real Arch machine, not executed automatically — treat edits as documentation changes, not code changes. Preserve its structure: top-level `##` sections mirror major setup phases (post-install → package setup → personalization → applications), with `###`/`####` subsections per tool or task.
- Shell commands and file contents inside `setup-arch.md` are illustrative snippets (some are commands to run, others are full config file contents to paste in, e.g. `~/.vimrc`, `~/.config/git/config`). When editing, keep commands accurate for the file's distro: `pacman`/`yay` in the Arch guide, `apt`/Snap/vendor repos in the Ubuntu guide.
- `starship-dracula.toml` is the actual artifact a user installs; keep its `palette = "dracula"` colors consistent with the palette table at the bottom of the file (`[palettes.dracula]`) if you change any styling.
- The "TODOs" and "Later" sections at the end of `setup-arch.md` track outstanding work (AWS CLI, Go, scripting the whole setup, etc.) — check there before assuming a task is already covered.
- This repo is public: never commit real personal data (emails, employer/client names, SSH key names, real UUIDs/MACs). Use placeholders such as `<username>`, `you@example.com`, `~/workspace/work/`, `<uuid>`, `<device_mac>`.
- If you add or rename a `setup-arch.md` section, update its Table of Contents and the summary in `README.md`.

## Multi-distro guides

- `setup-arch.md` is the **base guide**. Write new steps there first, then port them to every other distro guide.
- Name other guides `setup-<distro>-<version>.md` (e.g. `setup-ubuntu-26.04.md`) and state the version in a note at the top of the file. Check package names against that release (e.g. on packages.ubuntu.com), not from memory.
- Use the base guide's section numbers in every distro guide:
  - Step identical on both distros: link to the Arch section (`Same as [Arch 2.1.4](setup-arch.md#214-...)`), don't copy it.
  - Arch-only step with nothing to do on the other distro: leave it out entirely (no heading, no "not needed" note). Its number is simply skipped.
  - Arch-only step that has a real replacement (e.g. `yay` → software sources): keep the number and write the replacement steps.
  - Distro-only step: add it under a new number at the end of the matching section (e.g. Ubuntu 1.4–1.7).
- Ubuntu software sources, in order: apt → vendor apt repo/`.deb` (apps that need system integration) → Snap (desktop apps missing or outdated in apt) → no Flatpak. Store repo keys in `/etc/apt/keyrings/` with `signed-by=`; never use `apt-key`.
- When the base guide changes, update the matching section in every distro guide, both Tables of Contents, and `README.md`. Check that links to `setup-arch.md#...` anchors still resolve after any heading rename.
