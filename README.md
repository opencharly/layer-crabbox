# crabbox

The Crabbox CLI for OpenCharly images — a remote software testing and execution
control plane — BUILT from the pinned [`opencharly/crabbox`](https://github.com/opencharly/crabbox)
source so it carries the `containerd` SSH-lease provider.

The `crabbox` candy builds the single static binary `/usr/local/bin/crabbox` from
the commit named by `CRABBOX_COMMIT` (`go build -trimpath ./cmd/crabbox`), with
that commit injected as the reported version. It composes the CLI's laptop
prerequisites from sibling candies instead of re-shipping them (R3): `git` via
`layer-gh`, `ssh`/`ssh-keygen` via `layer-ssh-client`, the Go toolchain via
`layer-golang`, plus the `rsync` and `curl` packages.

## What it provides

| Property | Value |
|---|---|
| Layer / candy | `crabbox` |
| Binary | `/usr/local/bin/crabbox` |
| Source | pinned `opencharly/crabbox` commit (`CRABBOX_COMMIT`) |
| Version | the pinned commit (`-X .../internal/cli.version=<commit>`) |
| Requires | `layer-golang`, `layer-gh`, `layer-ssh-client`, `rsync`, `curl` |
| Config sibling | `crabbox-containerd-config` |
| Alias | `crabbox` |
| Service / port | none |

## Why a source build

No `openclaw/crabbox` release carries the `containerd` SSH-lease provider (W2
seam 1, `opencharly/crabbox#7`), so a downloaded release cannot lease on
containerd. The build pins the merge commit, and `crabbox --version` reports it —
the binary names its own source.

## How to use it

Compose the layer by pinning this repo in a box's `candy:` list (use the CalVer
tag minted when this repo merges; `main` below is the pre-tag form):

```yaml
my-box:
  candy:
    base: fedora
    candy:
      - '@github.com/opencharly/layer-crabbox:main'
```

Select the containerd provider with the separate config candy (by path; the root
ref's STRING is unchanged, but the resolved candy is now named `layer-crabbox`, so
a consumer that composes it by NAME follows the rename):

```yaml
    candy:
      - '@github.com/opencharly/layer-crabbox/crabbox-containerd-config:main'
```

Then, inside the built image (or on a dev host via the `charly alias` wrapper):

```bash
crabbox --version                   # the pinned commit
crabbox providers                   # includes containerd
crabbox providers describe containerd
crabbox config show                 # provider=containerd image=<lease image>
```

## Verification

The CLI candy's `plan:` asserts the binary at the fixed path, `crabbox --version`
reporting the pinned commit, `crabbox providers` listing `containerd`, and
`crabbox providers describe containerd` succeeding. The `crabbox-containerd-config`
candy separately asserts that its rendered file is read back by
`crabbox config show` with `provider=containerd` and the lease image.
`crabbox doctor` is not asserted at image time — it judges a LIVE venue's provider
prerequisites, which a build container does not have. The repo's R10 bed
`check-crabbox-cli` runs the whole surface in a disposable container.

## Layout

- `charly.yml` — the `layer-crabbox:` candy entity (`CRABBOX_COMMIT`, the `require:`
  deps, the ONE fetch+build+discard `command:` step (curl → tar → `go build`,
  source removed in the same RUN), the `check:` assertions), the root
  `discover:` block, and the embedded `crabbox-skill:` skill entity.
- `crabbox-containerd-config/charly.yml` — the containerd config candy.
- `box/check-crabbox-cli/charly.yml` — the R10 witness.
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `README.md` — this user overview.

## Related

- Owning skill: `/charly-tools:crabbox-cli`
- Check verb: `/charly-check:crabbox` (`opencharly/plugin-crabbox`)
- Upstream: [openclaw/crabbox](https://github.com/openclaw/crabbox)
- [`opencharly/charly`](https://github.com/opencharly/charly) — the charly CLI and image builder
- [`opencharly/opencharly`](https://github.com/opencharly/opencharly) — the umbrella

## Deliberate narrowing: the runtime pack is not installed

The build compiles **only `./cmd/crabbox`**; upstream's release archives also carry
a **native runtime pack** — CLI-side command-supervision companions that let
`crabbox` run commands on a Linux target **without Bash**:

> "Complete installations now include a native runtime pack. … the Linux companions let
> independent Linux and WSL2 SSH commands run without Bash." — upstream `CHANGELOG` at `v0.79.0`

That is deliberate here, and the cost is bounded: **every box this candy composes into
ships Bash** (Fedora, Arch, Debian, Ubuntu, CachyOS), so the narrowed install is
sufficient for leasing containers and driving a local coordinator. A **Bash-less Linux
target** — WSL2, a minimal or Alpine guest, a Bash-less runner — would need
`./cmd/crabbox-runtime` built and installed too, and `crabbox --preflight
--preflight-tools default,bash` reports the absent state there.

Recorded because the narrowing is invisible in `charly.yml` unless you know what the
source tree carries, and because the pack's absence is otherwise indistinguishable from a
failure on such a target.
