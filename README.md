# SanctuWrite — Releases

Public host for SanctuWrite's auto-updater manifest (`latest.json`) and, via
**GitHub Releases**, the signed installers themselves. This is a **separate
repo** from the app source; its local working copy lives inside the app project
folder but is gitignored there.

- **This repo must be PUBLIC** — the updater fetches `latest.json` and the
  release assets over plain HTTPS with no auth token.
- The app checks for updates **manually only** (Settings → "Check for
  updates"). It never phones home on its own; the fully-offline promise holds.

## How it's laid out

- **In the repo tree**: only `latest.json` (and this README). The repo stays
  tiny forever.
- **As Release assets**: each version's `SanctuWrite_<ver>_x64-setup.exe` and
  its `.sig`, attached to the tag `v<ver>`. Assets live in GitHub's file
  store, not git history. Every version stays downloadable at
  <https://github.com/Joitri/sanctuwrite-releases/releases> — that page (or
  `releases/latest`) is also the human download link for a website.

The app compares its running version to `latest.json`'s `version`; if newer,
it downloads the `url`, verifies `signature` against the public key baked into
the app, and installs. The manifest endpoint baked into shipped apps is:

```
https://raw.githubusercontent.com/Joitri/sanctuwrite-releases/main/latest.json
```

(Never move/rename this file — installed apps read exactly this URL.)

## Publishing a new release

One command from the app repo (needs the TAURI_SIGNING_* env vars and a
one-time `gh auth login`):

```
npm run release -- --publish --notes "What changed."
```

That builds the signed bundle, creates the GitHub Release `v<ver>` with the
installer + `.sig` as assets, rewrites `latest.json` to point at the new
asset, and pushes it. See `scripts/release.mjs` in the app repo.

## `latest.json` format (Tauri v2 static manifest)

```json
{
  "version": "1.4.1",
  "notes": "What changed in this release.",
  "pub_date": "2026-07-08T00:00:00Z",
  "platforms": {
    "windows-x86_64": {
      "signature": "<full contents of the .exe.sig file>",
      "url": "https://github.com/Joitri/sanctuwrite-releases/releases/download/v1.4.1/SanctuWrite_1.4.1_x64-setup.exe"
    }
  }
}
```

Add more `platforms` keys (`darwin-x86_64`, `darwin-aarch64`, `linux-x86_64`)
if/when macOS/Linux builds ship.

## Rollback

If a release ships broken: edit `latest.json` to point `version`, `url`, and
`signature` back at the previous release's asset and push. Installed apps
update (or stay) accordingly — no rebuild needed, since every old installer
remains attached to its release.
