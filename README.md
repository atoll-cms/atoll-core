# atoll-core

Core runtime for atoll-cms.

## Contains

- PHP runtime (`src/`)
- Admin SPA assets (`admin/`)
- Default theme fallback (`themes/default/`)
- Island bundles (`islands/`)
- Core migrations (`migrations/`)
- Release utilities (`tools/`)

## Versioning

Semantic versioning via `VERSION`.

## Building a release artifact

```bash
php tools/build-release.php
```

This generates `releases/atoll-core-<version>.zip` with a top-level `core/` directory,
compatible with `atoll core:update` / `atoll core:update:remote` in starter projects.

## Signing a release

```bash
php tools/sign-release.php \
  --private-key=/path/release-private.pem \
  --version=<version> \
  --sha256=<artifact_sha256>
```
