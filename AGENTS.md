# AGENTS.md — layer-firecracker

The **Firecracker microVM host layer**: the pinned VMM + jailer, the kernel artefact the fireactions
pools boot, and (coming) the host posture a microVM needs — the KVM access mode, the TAP device,
forwarding/NAT, `ip_forward`, and the `mode: cni` conflist with `tc-redirect-tap`.

## Canonical files

- `candy/layer-firecracker/charly.yml` — the layer: `var:` pins, the download steps, and the checks
  that ASSERT every pin.
- `box/check-firecracker-app/charly.yml` — the disposable R10 bed IMAGE (a Fedora base composing the
  layer, with its own checks on the running pod).
- `charly.yml` — the project manifest + the `check-firecracker-app` bed.
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.

## Load these skills first (R0)

- `/charly-image:layer` — candy authoring: the `candy:` node, `var:`, the plan-step verb catalog, the
  `download:` verb and its caching. Load before touching `charly.yml`.
- `/charly-check:check` — before running or authoring the bed.
- `/charly-internals:disposable` — before any autonomous destroy/rebuild of a target.
- `/charly-local:local-spec` + `/charly-local:local-deploy` — before authoring the `kind: local`
  template + deploy that owns the host posture.
- `/charly-internals:git-workflow` — before any git/PR action.

## Build / validate / test

- `charly box validate` at the repo root — the structural gate (the candy, the bed image, the bed).
- `charly check run check-firecracker-app` — the R10 bed (`disposable: true`).
- The merge gate is the **org-wide** `charly/pr-validator` (required check `validate / validate`).

## Rules that are easy to get wrong here

- **Every pin is asserted, not just used.** The `download:` verb's cache is content-addressed by URL —
  a CACHE key, not an integrity pin. The sha256 assertions in the `plan:` are the pin; without them a
  substituted or truncated download is undetectable.
- **One kernel for the layer and the pools (R2).** The kernel comes from the fireactions distribution
  channel at the version the pools pin. Never introduce a second kernel.
- **A `candy:` node has no `version:` field** (it was removed from `#Candy`) — adding one fails
  validation with a CUE disjunction error that names an unrelated field, so do not trust the field
  name in that message; remove `version:` and re-validate.
- **Root-scoped host work does not belong in an image check.** `/dev/kvm` access, TAP creation and
  forwarding are `kind: local` deploy steps on a real host, proved on a disposable VM.
- **A bzImage is not a kernel here.** Assert the ELF magic whenever a kernel artefact is installed.

## Landing

- PR-only; the org validator arms native auto-merge on PASS and `tag-on-merge` writes
  `CHANGELOG/<CalVer>.md` from the PR body. Direct pushes to `main` are blocked.
- The authoritative rulebook is the umbrella `AGENTS.md` in `opencharly/opencharly`. Do not restate
  its rules here.
