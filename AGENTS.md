# Authoring & maintaining the `gpu-server-setup` skill

Agent guide for working **inside this skill repository**. Read it before editing
`SKILL.md`, the references, the presets, `scripts/check.sh`, or any plugin
manifest.

> Naming (Russian): the adjective for "agent / agentic" is **«агентный»**, never
> «агентский».

## Repository layout

This skill is packaged for **Claude Code**, **Cursor**, and **OpenAI Codex**
(and installs as a plain skill folder for other agents). The layout is **flat** —
`SKILL.md` and its assets live at the repo root, and the plugin manifests point
their skill source at `./`.

```
gpu-server-setup/
├─ .claude-plugin/
│  ├─ plugin.json        # Claude Code plugin manifest
│  └─ marketplace.json   # single-plugin marketplace (plugin lives at "./")
├─ .cursor-plugin/plugin.json    # Cursor manifest
├─ .codex-plugin/plugin.json     # Codex manifest (interface block + skills: "./")
├─ SKILL.md              # CANONICAL skill: YAML frontmatter + instructions
├─ references/           # deep knowledge, loaded on demand from SKILL.md
├─ presets/              # ready docker-compose stacks (adapt, don't paste verbatim)
├─ scripts/check.sh      # staged verification script (pure bash)
├─ README.md  AGENTS.md  LICENSE
```

There is no `skills/<name>/` subfolder — the root **is** the skill.

## Editing rules

- **Keep `SKILL.md` thin.** It holds the workflow, the stage gates, and the hard
  rules. Depth goes in `references/` and `presets/`. If a section grows past a
  screen, move the detail into a reference and link it.
- **One package source.** Every install path in this skill uses the official
  NVIDIA CUDA apt repo + the official Docker repo. Never add instructions that
  mix distro `nvidia-*` packages, `ubuntu-drivers`, or `.run` installers into
  the same flow — mixed sources are the #1 support burden this skill exists to
  prevent.
- **Gates are not optional.** Any edit must preserve the staged verification
  order (lspci → nvidia-smi → GPU-in-Docker → service health) and the rule that
  the agent stops on a failing gate.
- **Presets must stay runnable.** They are working configs with
  `<PLACEHOLDER>` tokens for site-specific values (IPs, tokens, models where
  relevant). Never bake in real IPs or secrets. Version numbers in `image:`
  tags are allowed (they document a known-good combination) but prefer noting
  "check for the current release" where staleness bites.
- **check.sh is pure bash**, no dependencies beyond coreutils/curl/docker. It
  must stay read-only except the single `docker run … nvidia-smi` gate, must
  not use `set -e` (it collects failures), and must exit non-zero on any FAIL.
  After editing run `bash -n scripts/check.sh` and exercise it on a machine
  with and without a configured GPU.
- **Version drift.** Driver branches, CUDA majors, and image tags in the text
  are examples of a known-good state, not eternal truth. When refreshing them,
  update `nvidia-cuda.md`, the presets, and the README consistently in one
  commit.

## Keeping metadata in sync

When you change the description or bump the version, update it in **all** of:

- `SKILL.md` frontmatter (`version`, `description`)
- `.claude-plugin/plugin.json` **and** `.claude-plugin/marketplace.json`
- `.cursor-plugin/plugin.json`
- `.codex-plugin/plugin.json` (both top-level and the `interface` block)
- `README.md`

Keep the `name` identical everywhere: `gpu-server-setup`. Inside this repo's own
`.claude-plugin/marketplace.json` both the plugin `version` and the top-level
`metadata.version` track this skill, so bump both.

## What this skill does NOT do

- It does not manage NVIDIA on Windows/WSL, macOS, RHEL-family, Arch, or NixOS.
- It does not cover AMD/Intel GPUs, ROCm, or oneAPI.
- It does not do Kubernetes (device plugin, GPU operator) — single-host Docker
  Compose only.
- It does not benchmark or fine-tune models; it prepares the server and brings
  the serving stack up.
