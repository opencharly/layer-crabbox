# crabbox

The Crabbox CLI for OpenCharly images — a remote software testing and execution
control plane.

The `crabbox` candy installs the pinned upstream
[Crabbox](https://github.com/openclaw/crabbox) CLI — lease, sync, run, stream,
and release remote test execution — as the single static binary
`/usr/local/bin/crabbox`. It composes the CLI's laptop prerequisites from sibling
candies instead of re-shipping them (R3): `git` via `layer-gh`, `ssh`/`ssh-keygen`
via `layer-ssh-client`, plus the `rsync` and `curl` packages.

## What it provides

| Property | Value |
|---|---|
| Layer / candy | `crabbox` |
| Binary | `/usr/local/bin/crabbox` |
| Version | pinned `0.51.0` (`CRABBOX_VERSION` var; GoReleaser release archive) |
| Requires | `layer-gh` (git), `layer-ssh-client` (ssh/ssh-keygen), `rsync`, `curl` |
| Alias | `crabbox` |
| Service / port | none |

## How to use it

Compose the layer by pinning this repo in a box's `candy:` list:

```yaml
my-box:
  candy:
    base: fedora
    candy:
      - '@github.com/opencharly/layer-crabbox:v2026.250.1023'
```

Then, inside the built image (or on a dev host via the `charly alias` wrapper):

```bash
crabbox --version       # reports the pinned 0.51.0 release
crabbox providers       # the credential-free local provider matrix
crabbox doctor          # passes with local prerequisites only
```

The candy's `plan:` asserts exactly this: the binary lands at the fixed path,
`crabbox --version` reports the pinned release on stdout, `crabbox providers`
lists `local-container`, and `crabbox doctor` passes in a broker-less context —
so a missing download or a broken binary fails the checks.

## Layout

- `charly.yml` — the `crabbox:` candy entity (the `CRABBOX_VERSION` var, the
  `require:` deps, the `download:`/extract `plan:`, and the `check:` assertions)
  and the embedded `crabbox-skill:` skill entity.
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `README.md` — this user overview.

## Related

- Owning skill: `/charly-tools:crabbox`
- Sibling binary-download candy: `/charly-tools:cue`
- Upstream: [openclaw/crabbox](https://github.com/openclaw/crabbox)
- [`opencharly/charly`](https://github.com/opencharly/charly) — the charly CLI and image builder
- [`opencharly/opencharly`](https://github.com/opencharly/opencharly) — the umbrella
