# Writing a store entry

Start from an existing entry in `apps/` and check it locally:

```sh
cargo install --locked --git https://github.com/SpiritEXEWasTaken/hearth --features tool hearth-store
hearth-store lint .
hearth-store render . <id> --option gpu=vaapi     # the merged compose file for those options
hearth-store combos . --out /tmp/combos && for d in /tmp/combos/*/; do (cd "$d" && docker compose config -q); done
```

## Rules

- **id**: reverse-domain, lowercase, never changes (`org.jellyfin.jellyfin`). The folder has the same name.
- **One app, one entry.** Nvidia/VAAPI builds, editions, "with machine learning" etc. are `options` that pick
  `variants`. CI compares the main image and the upstream source with every other entry.
- **Tags are tracks** (`latest`, `stable`, `16`, `v3`, `9-alpine`). Anything with `<digit>.<digit>` or a digest is a
  pin and needs `pin_reasons: {<image without tag>: why}`.
- **Published ports use variables** from `ports` (`"${WEB_PORT}:8096"`), so Hearth can move them when the default is
  taken. Ports devices must find (DNS 53, discovery) are `movable: false`.
- **App data lives under `${APP_DATA}`** (Hearth fills in `/var/lib/hearth/data/<app>`), or in a folder option.
  Other bind mounts are limited to `/etc/localtime`, `/run/dbus`, `/dev/dri`, `/dev/bus/usb`, `/dev/serial`.
  List folders that must exist before the first start (and their owner) in `data_dirs`.
- **Variables Hearth fills in:** `PUID`, `PGID`, `TZ`, `APP_DATA`, `RENDER_GID`, `VIDEO_GID`. Everything else comes
  from `env` (fixed value or `generate: password|hex32`), `ports` or an option.
- **Secrets are generated**, never written in the entry. Mark the ones the user needs (an admin password) with
  `show_as`; admins see them under "Show sign-in details" on the app's page.
- **No `container_name`, no `x-hearth`** in compose files: Hearth names containers after the app and writes
  `x-hearth` itself.
- **Requirements are honest:** `ram_mb` (to run at all), `data_mb` (its own data, not your media),
  `architectures` (CI checks them), `gpu`, `host_network`, and `web.https` (`required` makes Hearth turn HTTPS on).
- **notice** for what's *not* included ("No movies, TV shows or music are provided").
- **Screenshots**: at least one; `source: own`, or the upstream URL plus its `license`.

## Install options

| kind | what it does |
|---|---|
| `gpu` | choices with `needs: dri / intel / amd / nvidia`; `default: auto` picks the best hardware found |
| `choice` | pick one; each choice can add `variants` and `env` values |
| `switch` | `on` / `off`, each with `variants` and `env` |
| `folder` | a host folder, stored in `env` (default under `${APP_DATA}`; other folders must already exist) |
| `text` | one line of text, stored in `env` |

Variants are compose override files. Lists are joined, `environment`/`labels` merge by key, `volumes`/`devices`
by container path, and `x-remove-services: [name]` drops a service (e.g. Immich without machine learning).
