# AGENTS.md — layer-crabbox

Standalone candy repo for the `crabbox` layer — the pinned Crabbox CLI, BUILT from
the `opencharly/crabbox` source at a pinned commit, plus its laptop prerequisites
and (as a separate candy) the containerd provider config.

Canonical files:

- `charly.yml` — the root `layer-crabbox:` candy entity (`CRABBOX_COMMIT`, the
  `require:` deps, the one fetch+build+discard `command:` step, and the `check:`
  assertions) and the embedded `crabbox-skill:` skill entity projected into the
  marketplace corpus as `/charly-tools:crabbox-cli`. Also the root `discover:`
  block for `crabbox-containerd-config/` and `box/`.
- `crabbox-containerd-config/charly.yml` — the separate containerd config
  candy (`CRABBOX_CONTAINERD_IMAGE`, `provider: containerd`,
  `containerd.image`), which `require:`s the `crabbox` candy, rendered through
  the `config:` verb + the `crabbox-yaml` CUE schema.
- `box/check-crabbox-cli/charly.yml` — the R10 witness: a disposable container
  running the source-built CLI and asserting the `containerd` provider surface.
- `.agents/skills/crabbox-cli/SKILL.md` — the owning skill.
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `README.md` — user overview only; never agent guidance.

## Load these skills first (R0)

- `/charly-tools:crabbox-cli` — the owning skill. The source build, the composed
  prerequisites, the provider surface, and the config candy.
- `/charly-image:layer` — the candy authoring reference (`charly.yml` schema,
  `plan:` step verbs incl. `command:`/`config:`, package sections).
- `/charly-check:check` — the bed/R10 authoring reference.

## Build / validate / test

- `charly box validate` at the repo root — the structural check (candies, the
  `plugin:`-less candy surface, the CUE-validated config step, the beds).
- `charly box build check-crabbox-cli-image` + `charly check box` /
  `charly check run check-crabbox-cli` — the functional evidence: the binary at
  `/usr/local/bin/crabbox`, `crabbox --version` reporting the pinned COMMIT,
  `crabbox providers` listing `containerd`, `providers describe containerd`
  succeeding, and the rendered config read back by `crabbox config show`.
- The merge gate is the **org-wide** `charly/pr-validator` (required check
  `validate / validate`, defined in `opencharly/.github`); this repo has **no**
  per-repo candy gate and ships only `.github/workflows/tag-on-merge.yml`.
- The checks require a Go toolchain that satisfies the pinned source's `go.mod`
  (layer-golang provides it; `GOTOOLCHAIN=auto` fetches the exact toolchain when the
  distro's Go is older). A build from the wrong commit fails the version check.

## Modify this repo

- Edit the `layer-crabbox:` candy entity AND the `crabbox-skill:` skill entity in
  `charly.yml` together. The skill is the projected usage source, so a version or
  behaviour change not mirrored in the skill leaves the corpus stale.
- New behaviour claims belong in the `plan:` as an observable `check:` step, and
  in the skill body.
- `CRABBOX_COMMIT` is the pin: the source tarball URL and the `-X ...version=`
  ldflag are the SAME value, so the reported version is always the built commit.
  Never fall back to a release archive — no `openclaw/crabbox` release carries the
  `containerd` provider.
- The config candy is a SEPARATE entity on purpose: a consumer that only
  wants the binary must not silently inherit a `provider:` default. Its ref is by
  path; the root `crabbox` ref keeps its meaning for every existing consumer.

## Landing

- PR-only. Every change lands through a pull request; the org-required
  `charly/pr-validator` validates the diff and body and arms native auto-merge
  on PASS. Direct pushes to `main` are blocked.
- History lives in `CHANGELOG/` (written by `tag-on-merge` at merge time); the
  PR body IS the changelog.
- The authoritative rulebook is the umbrella `AGENTS.md` in
  `opencharly/opencharly` and `charly/AGENTS.md` in the charly repo. Do not
  restate its rules here.
