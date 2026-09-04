# FrameLab Assets

Remote asset catalog for the FrameLab Premiere Pro plugin (SFX, icons, MOGRTs).

## Structure

- `catalog.json` — the list of available assets. Fetched by the plugin from
  this file's raw URL on the default branch, so updating it here (metadata
  only — id, display names, category, filename, version) takes effect for
  every user without a plugin update.
- Binary files (audio, images, MOGRTs) are **not** stored in this repo's git
  history. They're uploaded as assets on the `assets-v1` GitHub Release
  instead, and downloaded by the plugin on demand at:
  `https://github.com/MattKimoy/framelab-assets/releases/download/assets-v1/<filename>`

## Adding a new asset

1. Upload the file to the `assets-v1` release (via `gh release upload assets-v1 <file>`
   or the GitHub web UI).
2. Add an entry to `catalog.json` with a unique `id`, display names, category,
   the exact `filename` uploaded, and `"version": 1`.
3. Commit and push `catalog.json`.

## Updating an existing asset's file

Because the plugin caches downloaded files by filename, changing a file's
*content* without changing its *filename* would leave stale copies cached on
users' machines. Instead:

1. Upload the new file under a **new** filename (e.g. add a `.v2` before the
   extension: `stomp-clap.v2.wav`).
2. Bump that item's `"version"` in `catalog.json` and update its `filename`
   to match.
3. The old file can stay on the release (harmless) or be removed once no
   longer referenced by `catalog.json`.
