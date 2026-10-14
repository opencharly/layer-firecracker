# layer-firecracker

The **Firecracker microVM host layer** for charly: the pinned VMM, the kernel the pools boot, and the
host posture a microVM needs — every artefact pinned, and every pin **asserted** so a substituted or
truncated binary fails at check time rather than at boot.

| Artefact | Pin |
|---|---|
| `firecracker` VMM + `jailer` | upstream release **v1.17.0** (x86_64), verified byte-for-byte by sha256 |
| kernel (`vmlinux`) | **fireactions' own 5.10** kernel — the SAME kernel the pools boot, never a second one |
| kernel shape | asserted to be the uncompressed **ELF** the VMM loads, never a distro `bzImage` |

```console
$ charly box validate                 # the candy, the bed image and the bed
$ charly check run check-firecracker-app   # the disposable R10 bed
```

## Why a microVM needs its own layer

Firecracker boots a guest with **no libvirt, no domain and no host arbiter lease** — which is exactly
why the substrate reuses the `ssh` venue and declares **no** exclusive venue (an exclusive venue would
make the kernel misroute the deploy into `charly vm build`/`charly vm create`). What it does need is a
different set of host things:

- the **VMM and its jailer**, pinned;
- an uncompressed **ELF kernel** — the VMM loads it directly, so a distro `bzImage` is a guest that
  never reaches its first instruction. The check asserts the ELF magic for that reason;
- `/dev/kvm` reachable by the account that runs the VMM (a `kvm` group or an ACL — **not** a
  world-writable `0666`);
- a **TAP** device plus forwarding/NAT and `ip_forward` for `mode: tap`, or the pinned
  `tc-redirect-tap` + conflist for `mode: cni`.

## What is proved, and where

The bed in this repo is a **disposable pod**: it proves the **artefact half** — the pins, the
binaries, the kernel's ELF shape, the installed paths — cheaply and on a real deploy.

The **host posture** (KVM, TAP, forwarding, CNI) needs a real kernel and a real host, so it is applied
by the `kind: local` deploy in this repo and proved on a **disposable VM**, never on an operator's
workstation. Until that lands, the layer does not claim it.
