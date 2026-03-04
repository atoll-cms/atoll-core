# atoll-core

Core runtime for atoll-cms.

## Contains

- PHP runtime (`src/`)
- Admin SPA assets (`admin/`)
- Built-in fallback theme (`themes/default`)
- Island bundles (`islands/`)
- Core migrations (`migrations/`)
- Release utilities (`tools/`)

## Official external themes

Official themes are maintained as separate repositories:

- [atoll-theme-skeleton](https://github.com/atoll-cms/atoll-theme-skeleton)
- [atoll-theme-business](https://github.com/atoll-cms/atoll-theme-business)
- [atoll-theme-editorial](https://github.com/atoll-cms/atoll-theme-editorial)
- [atoll-theme-portfolio](https://github.com/atoll-cms/atoll-theme-portfolio)

## Versioning

Semantic versioning via `VERSION`.

## Building a release artifact

```bash
php tools/build-release.php
```

This generates `releases/atoll-core-<version>.zip` with a top-level `core/` directory,
compatible with `atoll core:update` / `atoll core:update:remote` in starter projects.

## Benchmark tools

Core ships benchmark tooling consumed by starter CLI commands:

- `tools/benchmark-run.php`
- `tools/benchmark-report.php`

Use via starter:

```bash
php bin/atoll benchmark:run --rounds=3
php bin/atoll benchmark:report --out=benchmarks/results/latest.md
```

## Signing a release

```bash
php tools/sign-release.php \
  --private-key=/path/release-private.pem \
  --version=<version> \
  --sha256=<artifact_sha256>
```
