# Tautulli Helm Chart

Deploys [Tautulli](https://tautulli.com/) from its official image, [`ghcr.io/tautulli/tautulli`](https://github.com/Tautulli/Tautulli).
Tautulli watches a Plex Media Server through its API, and keeps the history, statistics and notifications of what's played on it.

The chart runs one replica:

- A Deployment that stops the old pod before starting the new one, since `/config` is `ReadWriteOnce` and holds a SQLite database.
- A PersistentVolumeClaim for `/config`: `config.ini`, Tautulli's database, its backups, cache and logs.
  It's an ordinary claim the chart owns, so its size can be raised in the values; most storage classes allow claims to grow but not shrink.
- A Service on Tautulli's port, 8181, and an optional Gateway API `HTTPRoute` to it.
- A start-up, liveness and readiness probe on `/status`, the endpoint the image's own health check uses.
  Start-up gets 5 minutes, since Tautulli upgrades its database before it listens.

Tautulli reads Plex only through its API, so it mounts no media.

## Values

| Value | Default |
| --- | --- |
| `image.repository` | `ghcr.io/tautulli/tautulli` |
| `image.tag` | the chart's `appVersion` |
| `user` | `1000` |
| `group` | `1000` |
| `timezone` | `UTC` |
| `settings` | `SYSTEM_ANALYTICS: 0` |
| `settingsSecret` | none |
| `resources` | requests `cpu: 100m`, `memory: 256Mi`; no limits |
| `persistence.size` | `2Gi` |
| `persistence.storageClassName` | the cluster's default class |
| `affinity` | none |
| `httpRoute.enabled` | `false` |
| `httpRoute.parentRefs` | none; required when enabled |
| `httpRoute.hostnames` | none |

### Settings

Tautulli reads any of its `config.ini` settings from an environment variable named `TAUTULLI_` and the setting's name, upper-cased.
A setting given that way overrides `config.ini`, and the web UI can't change it.

`settings` passes plain settings that way, and `settingsSecret` names a Secret whose keys are setting names, for the secret ones.
For example, `PMS_TOKEN` in the Secret becomes `TAUTULLI_PMS_TOKEN`.

The chart sets `SYSTEM_ANALYTICS` to `0`, since Tautulli otherwise sends usage analytics to Google.
Set it to `1` to send them.

Notification agents, newsletters and per-user and per-library settings are kept in Tautulli's database, not `config.ini`, so they're set in the web UI or through Tautulli's API.

### Connecting to Plex

Tautulli connects to Plex Media Server only once its setup wizard is complete.
`FIRST_RUN_COMPLETE` set to `1` skips the wizard, and these settings take its place:

| Setting | Purpose |
| --- | --- |
| `PMS_URL` | The URL Tautulli connects to. When it's empty, Tautulli builds a plain `http` URL from `PMS_IP` and `PMS_PORT`. |
| `PMS_URL_MANUAL` | `1` keeps `PMS_URL` as given. Otherwise Tautulli replaces it with one from plex.tv every 12 hours. |
| `PMS_IP`, `PMS_PORT` | Needed even with `PMS_URL` set: start-up refreshes the server's details and libraries only when they're set. |
| `PMS_IDENTIFIER` | The server's machine identifier, from `http://<server>:32400/identity`. plex.tv lookups and the user list depend on it. |
| `PMS_TOKEN`, `PMS_CLIENT_ID` | Tautulli's own Plex token, and the client ID it was issued to. |

The token must be one issued to Tautulli, never the server's own `PlexOnlineToken` from its `Preferences.xml`.
Tautulli's requests to plex.tv name Tautulli as the product, so with the server's token plex.tv records the server as running Tautulli's version.
Tautulli logs an error when it sees this.

Tautulli gets its own token from **Settings → Plex Media Server → Fetch New Token**, which saves the token with its client ID.
A `PMS_TOKEN` set by environment variable overrides what that saves.

### User and group

The image's entrypoint starts as root, remaps its `tautulli` user to `PUID` and `PGID`, takes ownership of `/config`, and then drops to that user.
The chart skips it and runs Tautulli directly as user and group `1000`, the image's own `tautulli` user, so the pod never runs as root.
The pod's `fsGroup` is the same group, so Tautulli can write to `/config`'s volume.
`PUID` and `PGID` do nothing here; set `user` and `group` instead.

## Requirements

- A storage class for `/config`, the cluster's default unless named.
- The Gateway API's `HTTPRoute` CRD, when `httpRoute.enabled` is set.

## Usage

```sh
helm repo add tautulli https://tonylea.github.io/helm-chart-tautulli
helm install tautulli tautulli/tautulli --values my-values.yaml
```

```yaml
timezone: Europe/London
settings:
  FIRST_RUN_COMPLETE: 1
  PMS_URL: http://plex.media.svc.cluster.local:32400
  PMS_URL_MANUAL: 1
  PMS_IP: plex.media.svc.cluster.local
  PMS_PORT: 32400
  PMS_IDENTIFIER: 0123456789abcdef0123456789abcdef01234567
# Keys: PMS_TOKEN, PMS_CLIENT_ID, HTTP_USERNAME, HTTP_PASSWORD and API_KEY.
settingsSecret: tautulli-settings
```

## Development

Pull requests run:

- `helm lint --strict`, and the rendered chart checked by kubeconform against Kubernetes 1.37.
- helm-unittest's tests in `tests/`.
- An install and upgrade with chart-testing in a kind cluster.

Commits follow Conventional Commits.
Releases are versioned and tagged automatically from `main`, and published to the Helm repository on this repo's GitHub Pages.
