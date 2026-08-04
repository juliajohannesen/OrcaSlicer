# HackPGH README

Heya! Chances are, if you're looking at this repository, something has broken
with the automated HackPGH OrcaSlicer builds.

## How this fork works

`main` is rebuilt daily by `.github/workflows/hackpgh-sync.yml` as
**latest `upstream/main` + every `origin/hackpgh/*` branch merged in** and
force-pushed. Anything committed straight to `main` is wiped on the next sync —
all fork-only content must live on a `hackpgh/*` branch.

Releases mirror upstream stable tags: `hackpgh-release.yml` cherry-picks each
`hackpgh/*` branch's own commits onto the tag and publishes
`<tag>+hackpgh.<N>` with artifacts built by the stock `build_all.yml`.

### Patch branches (`hackpgh/<name>`)

Ordinary branches carrying commits, merged (sync) or cherry-picked (release).
Current: `hackpgh/automation` (these workflows), `hackpgh/filament-monitoring`,
`hackpgh/updater`.

### Generate branches (`hackpgh/generate/<name>`)

For changes too widespread to survive daily upstream churn as a diff (e.g. the
branding recolor touches 500+ files). Such a branch commits a **transformation
script** at `scripts/hackpgh/<name>/generate.sh` instead of the transformed
files. After ALL `hackpgh/*` branches are merged/cherry-picked, the workflows
run every `scripts/hackpgh/*/generate.sh` found in the assembled tree (sorted)
and commit the output.

A generate script must:

- run from the repo root, on Ubuntu CI runners and locally;
- be **deterministic**: same input tree → byte-identical output;
- **exit nonzero on upstream drift** (anchor/count assertions) — a failure
  aborts the run (`main`/the release stays untouched) and opens or refreshes a
  tracking issue, instead of silently shipping a half-transformed build.

Current: `hackpgh/generate/branding` — full HackPGH recolor (gold accents
#FFAD0A/#ffb319, purple chrome #7871aa, #0c0a13-anchored dark surfaces,
dark-mode-only, regenerated SVG/PNG/ICO/ICNS assets). See
`scripts/hackpgh/branding/brandkit/palette.py` for the palette and derivation.

### Required secrets

- **`HACKPGH_PUSH_TOKEN`** — fine-grained PAT on this repo, permissions
  **Contents: Read/Write + Workflows: Read/Write**. The release branch carries
  the upstream tag's history (including `.github/workflows` files unreachable
  from fork refs), which the Actions `GITHUB_TOKEN` is forbidden to push.
  The release workflow fails fast without it. Renew before expiry.
- **`GPG_PUBLIC_KEY` / `GPG_PRIVATE_KEY`** *(optional)* — sign the automation's
  commits; runs degrade to unsigned with a warning when absent or unusable.

### When something breaks

- **`Sync conflict:` issue** — a patch branch no longer merges onto upstream;
  rebase it and re-run *HackPGH Sync*.
- **`Generate failure:` issue** — upstream moved an anchor a generate script
  asserts on; fix the script on its branch and re-run *HackPGH Sync*.
- **`Release conflict:` / `Release generate failure:` issue** — same stories
  against a release tag; re-run *HackPGH Release* with the tag once fixed.
