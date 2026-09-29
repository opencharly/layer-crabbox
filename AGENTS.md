# AGENTS.md — layer-crabbox

Standalone candy repo for the `crabbox` layer — the pinned Crabbox CLI plus its
laptop prerequisites. The candy lives in `charly.yml` at the repo root: the
`CRABBOX_VERSION` var, the `require:` deps, the `download:`/extract `plan:`
steps, the `check:` assertions, and the embedded `skill:` entity projected into
the marketplace corpus as `/charly-tools:crabbox`.

Canonical files:

- `charly.yml` — the `crabbox:` candy entity and the `crabbox-skill:` skill entity.
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `README.md` — user overview only; never agent guidance.

## Load these skills first (R0)

- `/charly-tools:crabbox` — the owning skill. The pinned release, the composed
  prerequisites, and the credential-free local provider surface. Load before
  editing or troubleshooting the candy.
- `/charly-image:layer` — the candy authoring reference (`charly.yml` schema,
  `plan:` step verbs incl. `download:`/`check:`, package sections, service
  declarations). Load before editing any entity field or plan step.

## Build / validate / test

- The merge gate is the **org-wide** `charly/pr-validator` (required check
  `validate / validate`, defined in `opencharly/.github`); this repo has **no
  per-repo candy gate** and ships only `.github/workflows/tag-on-merge.yml`.
- The candy's `plan:` `check:` steps are the functional evidence: the binary at
  `/usr/local/bin/crabbox`, `crabbox --version` reporting the pinned release,
  `crabbox providers` listing the credential-free matrix, and a broker-less
  `crabbox doctor` pass. They must stay valid on every distro arm they run on.
- Keep `CRABBOX_VERSION` in lockstep with the release archive name (the download
  tag is `v`-prefixed; the asset name is `v`-less).

## Modify this repo

- Edit the `crabbox:` candy entity AND the `crabbox-skill:` skill entity in
  `charly.yml` together. The skill is the projected usage source, so a version
  or behaviour change not mirrored in the skill leaves the corpus stale.
- New behaviour claims belong in the `plan:` as an observable `check:` step, and
  in the skill body.

## Landing

- PR-only. Every change lands through a pull request; the org-required
  `charly/pr-validator` validates the diff and body and arms native auto-merge
  on PASS. Direct pushes to `main` are blocked.
- History lives in `CHANGELOG/` (written by `tag-on-merge` at merge time); the
  PR body IS the changelog.
- The authoritative rulebook is the umbrella `AGENTS.md` in
  `opencharly/opencharly` and `charly/AGENTS.md` in the charly repo. Do not
  restate its rules here.
