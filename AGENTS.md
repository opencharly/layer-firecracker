# AGENTS.md — layer-firecracker

The **Firecracker microVM host layer**: the pinned VMM + jailer, the kernel artefact the fireactions
pools boot, and the host posture a microVM needs — group-based `/dev/kvm` access with asserted
membership, and IP forwarding. The network half of the posture (NAT/FORWARD for `mode: tap`, and the
`mode: cni` conflist with its pinned plugin) is **opencharly/layer-firecracker#3**; a booted microVM
inside the guest is **opencharly/layer-firecracker#4**. Neither is described as if it were here.

## Canonical files

- `candy/layer-firecracker/charly.yml` — the ARTEFACTS: `var:` pins, the download steps, and the checks
  that ASSERT every pin.
- `box/check-firecracker-app/charly.yml` — the disposable R10 bed IMAGE for the artefact half.
- `charly.yml` — the project manifest: `firecracker-host-app` (the posture template),
  `check-firecracker-vm` (the **guest entity** the posture bed boots — a reusable `kind: vm`, never
  deployed on its own, so it carries no `disposable:`), and the two beds:
  `check-firecracker-app` (the artefact half, a pod) and `check-firecracker-posture` (the posture,
  inside a disposable VM).
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.

## Load these skills first (R0)

- `/charly-image:layer` — candy authoring (`var:`, the verb catalog, `download:` and its caching).
- `/charly-local:local-spec` + `/charly-local:local-deploy` — the `kind: local` template, the `host:`
  field, the root gates, and the nesting semantics. **Before** authoring anything host-scoped.
- `/charly-check:check` — before running or authoring a bed.
- `/charly-internals:disposable` — before any autonomous destroy/rebuild.
- `/charly-internals:git-workflow` — before any git/PR action.

## Build / validate / test

- `charly box validate` at the repo root — the structural gate.
- `charly check run check-firecracker-app` — the artefact half (a pod).
- `charly check run check-firecracker-posture` — the host posture (a nested deploy inside a VM).
- The merge gate is the **org-wide** `charly/pr-validator` (required check `validate / validate`).

## Rules that are easy to get wrong here

- **Every pin is asserted, not just used.** The `download:` verb's cache is content-addressed by URL — a
  CACHE key, not an integrity pin. The sha256 assertions in the `plan:` ARE the pin.
- **One kernel for the layer and the pools (R2).** It comes from the fireactions distribution channel at
  the version the pools pin. Never introduce a second kernel.
- **Host posture belongs to a `kind: local` template, never to a candy's install plan.** A candy step
  runs inside an IMAGE BUILD too, where udev/sysctl/packet-filter work is meaningless. The template is
  the deliberate act; the bed nests it inside a disposable VM.
- **A bed must never write to the operator's workstation.** Any host-scoped R10 proof nests a `local:`
  member under a disposable `vm:` — the canonical pattern (`check-arch-vm` → `arch-host`).
- **A `candy:` node has no `version:` field** (it was removed from `#Candy`). Adding one fails
  validation with a CUE disjunction error that names an unrelated field — do not trust the field name in
  that message.
- **A bzImage is not a kernel here.** Assert the ELF magic whenever a kernel artefact is installed.
- **`/dev/kvm` is never `0666`.** Group access is the posture; a world-writable device is the defect
  this layer exists to replace.

## Landing

- PR-only; the org validator arms native auto-merge on PASS and `tag-on-merge` writes
  `CHANGELOG/<CalVer>.md` from the PR body. Direct pushes to `main` are blocked.
- The authoritative rulebook is the umbrella `AGENTS.md` in `opencharly/opencharly`. Do not restate its
  rules here.
