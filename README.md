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

## File naming convention

**Hyphens only. No spaces. No category prefix.**

- ✅ `Cash-Register.mp3`, `Whoosh-1.mp3`, `Camera-Shake.png`
- ❌ `Cash Register.mp3` — spaces get silently mangled (see below)
- ❌ `SFX-Cash-Register.mp3` — no category prefix needed; the category
  already lives in `catalog.json`'s own `category` field, and that's what
  the plugin uses to sort the asset into the right place — the filename
  itself doesn't need to repeat it.

**Why no spaces, specifically:** GitHub Releases silently rewrites spaces
(and a few other characters) in an uploaded asset's filename — e.g.
uploading `"Cash Register.mp3"` actually lands as `"Cash.Register.mp3"`,
with no warning. If `catalog.json`'s `filename` doesn't match EXACTLY what
GitHub actually named the file, the plugin's download 404s. Using hyphens
from the start means there's nothing left for GitHub to rewrite.

## Adding a new asset

1. Rename the file locally first, following the convention above — e.g.
   `Cash-Register.mp3`, not `Cash Register.mp3` or `SFX-Cash-Register.mp3`.
2. Upload it to the `assets-v1` release:
   ```
   gh release upload assets-v1 Cash-Register.mp3 --repo MattKimoy/framelab-assets
   ```
   (the GitHub web UI's "edit release" page works too — drag the file in)
3. Confirm the filename GitHub actually kept (should match what you
   uploaded, since it has no spaces to touch):
   ```
   gh release view assets-v1 --repo MattKimoy/framelab-assets --json assets --jq ".assets[].name"
   ```
4. Add an entry to `catalog.json`'s `items` array. Full example:
   ```json
   {
     "id": "general-cash-register",
     "name": "Cash Register",
     "namePt": "Caixa Registradora",
     "category": "Viral SFX",
     "filename": "Cash-Register.mp3",
     "version": 1
   }
   ```
   Field reference:
   - **`id`** — unique, stable string. Never reused for a different asset,
     never changed on an existing one (the plugin keys favorites/downloaded
     state by it).
   - **`name`** — display name shown in the plugin (English).
   - **`namePt`** — optional. Portuguese display name. If omitted, `name` is
     shown in both languages.
   - **`category`** — must exactly match one of the plugin's own
     subcategories: `"Titles"`, `"Viral SFX"`, `"Captions"`,
     `"Essential Icons"`, `"HUD Library"`.
   - **`filename`** — must exactly match what GitHub actually named the
     uploaded asset (step 3 above).
   - **`version`** — starts at `1`. See "Updating an existing asset's file"
     below for when to bump it.
5. Commit and push `catalog.json`.

## Updating an existing asset's file

Because the plugin caches downloaded files by filename, changing a file's
*content* without changing its *filename* would leave stale copies cached on
users' machines. Instead:

1. Upload the new file under a **new** filename, still following the hyphen
   convention above — e.g. `Cash-Register-v2.mp3`, not `Cash-Register.v2.mp3`.
2. Bump that item's `"version"` in `catalog.json` and update its `filename`
   to match.
3. The old file can stay on the release (harmless) or be removed once no
   longer referenced by `catalog.json`.
