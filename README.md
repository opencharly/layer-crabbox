# layer-crabbox

The `layer-crabbox` candy of the [opencharly/charly](https://github.com/opencharly/charly)
candy library, as a standalone repo (kind-prefixed naming). The candy manifest lives at
the repo root; the charly resolver fetches this repo at the pinned tag.

Installs the [Crabbox](https://github.com/openclaw/crabbox) CLI — a remote software
testing and execution control plane (lease/sync/run/job/stop) — from the pinned
GoReleaser release archive, with git/ssh/rsync/curl prerequisites composed from
sibling candies.

```sh
charly box build <box-composing-crabbox> && charly alias install <box>
crabbox doctor
```