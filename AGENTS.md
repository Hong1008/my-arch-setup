# Agent Handoff — My Arch Linux Environment

## Read first

Before changing anything, read `ARCH-ENVIRONMENT.md`.

It is the authoritative document for:
- hardware
- OS layout
- current configuration
- completed work
- pending work
- design decisions
- learning goals
- backup architecture

## Mission

Help the user build and understand a personal Arch Linux development OS on:

- Lenovo IdeaPad Slim 3 14AHP10-83K9000CKR
- Ryzen 7 8840HS
- Radeon 780M
- target RAM: 24GB (8GB onboard + 16GB SO-DIMM)

Primary use cases:
- Spring Boot / backend
- RAG / AI applications
- MCP
- frontend / full-stack
- Linux learning
- reproducible personal OS

## Working style

- Preserve KISS.
- Prefer official Arch packages and ArchWiki.
- Explain why before adding configuration.
- Verify the current system before changing it when state matters.
- Do not repeat completed installation steps unnecessarily.
- Distinguish `done`, `reported done`, `verified`, `planned`, and `unknown`.
- Avoid speculative tuning.
- Use small, reversible steps.
- Give dangerous commands extra context and ask the user to verify the target before destructive operations.
- Do not automatically run full system upgrades.
- Prefer automatic checks/notifications + user approval for updates.

## Current architecture

```text
UEFI
  ↓
systemd-boot
  ↓
Arch Linux
  ↓
KDE Plasma / Wayland
  ↓
AMDGPU + Mesa
  ↓
NetworkManager / PipeWire / BlueZ / power-profiles-daemon
  ↓
Fcitx5 + Hangul
  ↓
Git / SSH / Zsh / tmux / rg / fd / fzf / Neovim / mise
```

## Backup architecture

```text
configuration  → dotfiles Git
OS definition  → arch-setup Git
secrets        → SOPS + age
actual data    → restic
```

## Update philosophy

Automate:
- firmware metadata refresh
- package update checks
- package cache cleanup
- backup scheduling

Do not automate blindly:
- `pacman -Syu`
- AUR updates
- firmware installation

Preferred flow:

```text
automatic check
  ↓
notification
  ↓
user approval
  ↓
update
  ↓
verification
  ↓
change log
```

## Suggested next work

1. Finish backup implementation and verify restore.
2. Build the bootstrap/reprovisioning scripts.
3. Add JDK 21 + Gradle.
4. Add Python + uv.
5. Add Node.js + pnpm.
6. Add Go/Rust as needed.
7. Add Docker.
8. Add PostgreSQL + pgvector + Valkey.
9. Add Ollama according to available memory/GPU constraints.
10. Build Spring/RAG/MCP/frontend development workflows.

## Handoff rule

At the end of a meaningful change:
- update `ARCH-ENVIRONMENT.md`
- record the reason for the change
- record important commands/config paths
- update the status/checklist
- add a change-log entry
