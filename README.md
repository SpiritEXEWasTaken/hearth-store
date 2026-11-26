# hearth-store

The app catalog for [Hearth](https://github.com/SpiritEXEWasTaken/hearth), the home-server manager.

**Browse it:** https://spiritexewastaken.github.io/hearth-store/ (install from your Hearth: **Store** in the menu,
or `hearth store install <app>`).

- **One entry per app.** GPU builds, editions and optional parts are *install options* of the same entry
  (variants), never separate entries. CI rejects a second entry for the same app.
- **No pinned versions.** Images use a track tag (`latest`, `stable`, a major such as `16` or `v3`), so Hearth's
  update checks keep working. A pin needs a written reason (`pin_reasons`).
- **Checked against the registry.** CI checks that every default image exists for every CPU type the entry
  claims, and records download sizes for the store's disk badges.
- **Signed.** The published `index.json` is signed (minisign, Ed25519). Hearth refuses a catalog without a valid
  signature, an older catalog than the one it has, and any file whose hash doesn't match.

## Layout

```
apps/<id>/app.yaml          the entry (format: schema/app.schema.json; CONTRIBUTING.md explains each field)
apps/<id>/compose.yaml      base compose file, no x-hearth block, no container_name
apps/<id>/variants/*.yaml   overlays used by install options (merged like `docker compose -f a -f b`)
apps/<id>/icon.svg|png
apps/<id>/screenshots/*     at least one, max 800 KB each
```

## Checks and publishing

`.github/workflows/store.yml` runs the `hearth-store` tool from the Hearth repo on every push and pull request:
`lint` (format, ids, categories, files, variables, ports, bind mounts, pins, duplicates), `combos` (every
combination of install options through `docker compose config`), `images` (registry and CPU types) and `build`.
On `main` (and weekly, to refresh image sizes) it signs the catalog with the store key and publishes it, with the
app browser, to GitHub Pages.

Icons come from [dashboard-icons](https://github.com/homarr-labs/dashboard-icons) (Apache-2.0); the logos belong
to their projects. Screenshots marked `source: own` were taken by the store maintainers in a test VM.
