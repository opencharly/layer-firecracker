# layer-firecracker

The **Firecracker microVM host layer** for charly: the pinned VMM, the kernel the pools boot, and the
**host posture** a microVM needs — every artefact pinned, and every pin **asserted** so a substituted
or truncated binary fails at check time rather than at boot.

| Artefact | Pin |
|---|---|
| `firecracker` VMM + `jailer` | upstream release **v1.17.0** (x86_64), verified byte-for-byte by sha256 |
| kernel (`vmlinux`) | **fireactions' own 5.10** kernel — the SAME kernel the pools boot, never a second one |
| kernel shape | asserted to be the uncompressed **ELF** the VMM loads, never a distro `bzImage` |

```console
$ charly box validate                          # the candy, the bed image, the templates and the beds
$ charly check run check-firecracker-app       # the ARTEFACT half, on a disposable pod
$ charly check run check-firecracker-posture   # the HOST posture, inside a disposable VM
```

## Two halves, two beds — and why they are different targets

**The artefacts are container work; the posture is host work.** The artefact half installs files, so a
pod proves it cheaply and the pod bed (`check-firecracker-app`) does exactly that.

The **posture** is udev rules, sysctls and packet-filter rules. Those have no meaning in an image build,
and applying them to an operator's workstation uninvited is the one thing this layer must never do — so
they live in a `kind: local` **host template** (`firecracker-host-app`) that a deploy applies
deliberately, and the bed proves them by **nesting**: `check-firecracker-posture` boots a disposable VM
and a nested `local:` member applies the template *inside the guest*. A bed never writes to your
workstation.

## Using it on a host

```yaml
my-microvm-host:
  local:
    from: firecracker-host-app
    host: my-microvm-host.lan     # or `local` for this machine — deliberately
```

That installs the pinned VMM + kernel and applies the posture:

- **`/dev/kvm` becomes group-accessible, never world-writable.** A `0660` node owned by group `kvm`,
  written as a udev rule, with the VMM's account added to the group. A world-writable `/dev/kvm` lets
  any local process reach every guest's memory and the host kernel through it.
- **IP forwarding is enabled**, both as a sysctl drop-in and applied immediately — a TAP-backed guest is
  a router's neighbour, and the host that filed this integration's WI-6 had the forwarding *policy* at
  `DROP` while `ip_forward` was already `1`, which is why the rule and the value are both written.
- **Self-gating and idempotent**: a target with no `/dev/kvm` prints why and skips rather than
  installing a rule that can never apply, and re-running changes nothing.

## What is not proved yet

Stated rather than implied: **`mode: cni`** (the pinned `tc-redirect-tap` plugin plus the conflist) and
a **booted microVM inside the guest**. Both are their own slices; the ledger carries them.
