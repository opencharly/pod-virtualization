# AGENTS.md — pod-virtualization

Standalone candy repo for the `virtualization` candy — the QEMU/KVM/libvirt
stack for both supervisord and systemd init systems, with a distro-divergent
daemon set. The candy lives in `charly.yml` at the repo root.

Canonical files:

- `charly.yml` — the `virtualization:` candy entity (description, `distro`,
  `service`, `plan`) and its `skill:` entity.
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `CHANGELOG/` — per-CalVer history.
- `README.md` — user overview only; never agent guidance.

## Load these skills first (R0)

- `/charly-infrastructure:virtualization` — the owning skill: the canonical
  mixed-`service:` polymorphism worked example and the rootless
  `qemu:///session` model. Load before editing, building, deploying, or
  troubleshooting this candy.
- `/charly-tools:charly` — the full toolchain that pulls this candy into
  charly-toolchain boxes.
- `/charly-distros:container-nesting` — pairs with this candy for boxes needing
  both nested containers and nested VMs.
- `/charly-image:layer` — the candy authoring reference (`charly.yml` schema,
  `plan:` step verbs incl. `run:` / `check:`, per-distro sections, the mixed
  `service:` schema).
- `/charly-check:check` — the check/R10 framework (`charly check box`,
  `charly check run <bed>`).
- `/charly-pod:pod` — the `kind: pod` / deploy schema reference (this candy is
  composed into a box; services).
- `/charly-internals:git-workflow` — before any git/PR action.

## Build / validate / test

- `charly box validate` at the repo root — the structural check: the manifest
  must parse and validate at the installed charly.
- The merge gate is the **org-wide** `charly/pr-validator` (required check
  `validate / validate`, defined in `opencharly/.github`); this repo has **no**
  per-repo candy gate. Its only workflow file is
  `.github/workflows/tag-on-merge.yml`.
- The candy's own `check:` steps assert the `virsh`, `qemu-system-x86_64`, and
  `qemu-img` binaries and the libvirt qemu driver package (via `package_map:`),
  and — at deploy scope — `virsh -c qemu:///session list --all` exiting 0.

## Modify this repo

- Edit the `virtualization:` candy entity in `charly.yml`; the `skill:` entity in
  the same file is the owning skill's source — a candy change and its skill
  change land together.
- Preserve the **mixed `service:` polymorphism**: each daemon appears twice with
  the same `name:` — a `use_packaged: <unit>.socket` form and a custom `exec:`
  form — plus a per-entry `distro:` filter. Do not collapse it into a
  `<name>-host` sibling (R3).
- `--timeout 0` keeps each daemon in the foreground for supervisord; keep it.
- Keep the `package_map:` on the qemu-driver check in step with the per-distro
  package names (Arch bundles into `libvirt`, deb bundles into
  `libvirt-daemon-system`).
- The `skill:` entity is the source for `/charly-infrastructure:virtualization`;
  never edit the generated `SKILL.md` — regenerate it.

## Landing

- PR-only. Every change lands through a pull request; the org-required
  `charly/pr-validator` validates the diff and body and arms native auto-merge on
  PASS. Direct pushes to `main` are blocked.
- History lives in `CHANGELOG/` (written by `tag-on-merge` at merge time); the PR
  body IS the changelog.
- The authoritative rulebook is the umbrella `AGENTS.md` in
  `opencharly/opencharly` and `charly/AGENTS.md` in the charly repo. Do not
  restate its rules here.
