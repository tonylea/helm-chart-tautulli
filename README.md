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
| `image.tag` | the chart's `appVersion` |
| `user` | `1000` |
| `group` | `1000` |
| `timezone` | `UTC` |
| `settings` | `SYSTEM_ANALYTICS: 0` |
| `settingsSecret` | none |
| `resources` | none; recommended requests are `cpu: 2m`, `memory: 133Mi`, Tautulli's typical use with one Plex server |
| `persistence.size` | `2Gi` |
| `persistence.storageClassName` | the cluster's default class |
| `affinity` | none |
| `httpRoute.enabled` | `false` |
| `httpRoute.parentRefs` | none; required when enabled |
| `httpRoute.hostnames` | none |

### Settings

`settings` sets Tautulli's settings, and `settingsSecret` names a Secret whose keys are settings, for the secret ones.
Settings given either way can't be changed in the web UI.

| Setting | In | Purpose |
| --- | --- | --- |
| `FIRST_RUN_COMPLETE` | `settings` | `1` skips the setup wizard. |
| `PMS_URL` | `settings` | The Plex server's URL. |
| `PMS_URL_MANUAL` | `settings` | `1` stops Tautulli replacing `PMS_URL` with one from plex.tv. |
| `PMS_IP`, `PMS_PORT` | `settings` | The Plex server's host and port. |
| `PMS_IDENTIFIER` | `settings` | The Plex server's machine identifier, from its `/identity`. |
| `PMS_TOKEN`, `PMS_CLIENT_ID` | Secret | A Plex token issued to Tautulli, and the client ID it was issued to; never the server's own `PlexOnlineToken`. |
| `HTTP_USERNAME`, `HTTP_PASSWORD` | Secret | The web UI's login. |
| `API_KEY` | Secret | The key for Tautulli's API. |
| `SYSTEM_ANALYTICS` | `settings` | Defaults to `0`, so no usage analytics are sent to Google. |

### User and group

Tautulli runs as `user` and `group`, never as root, and `/config`'s volume is given to the group.
The image's `PUID` and `PGID` aren't used.

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
