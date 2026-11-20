# Changelog

All notable changes to this project will be documented in this file. See [commit-and-tag-version](https://github.com/absolute-version/commit-and-tag-version) for commit guidelines.

## [2.0.0](https://github.com/tonylea/helm-chart-tautulli/compare/v1.0.0...v2.0.0) (2026-11-20)

### ⚠ BREAKING CHANGES

* image.repository is removed; the chart only deploys ghcr.io/tautulli/tautulli.

* always deploy the official Tautulli image ([91b6c3f](https://github.com/tonylea/helm-chart-tautulli/commit/91b6c3f14f97689c19c970d4bd32603e64b69cad))

### Documentation

* describe the settings that connect Tautulli to Plex, and which token to use ([ec6ebc5](https://github.com/tonylea/helm-chart-tautulli/commit/ec6ebc55ef20edd948daa0777faca710b1348124))
* list the settings to give and what each does ([07173d0](https://github.com/tonylea/helm-chart-tautulli/commit/07173d06d4d4b7b554b669a4b53cab205bd4cb28))

## 1.0.0 (2026-11-20)

### Features

* deploy Tautulli as a non-root user, with a claim for /config and its settings from values and a Secret ([e63a12a](https://github.com/tonylea/helm-chart-tautulli/commit/e63a12a0f61fe9cfdd71adbefbfd3262d8749dbe))

### Documentation

* describe the chart, its values and how to contribute ([9bd1a20](https://github.com/tonylea/helm-chart-tautulli/commit/9bd1a2018dd73915254d0dee7f5b06e9cf9604f5))
