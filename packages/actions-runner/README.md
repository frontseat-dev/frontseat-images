# actions-runner

The [GitHub Actions runner](https://github.com/actions/runner)'s own
release, pinned by checksum per architecture, for the `github-runner`
image. Wolfi does not package it. It is installed as released under
`/usr/share/actions-runner` (`run.sh`, `bin/`, `externals/`), less the
Alpine Node.js builds and .NET's LTTng tracing provider, which nothing here
can load (`melange.yaml` says why).

A new version: its `version`, and both `expected-sha256` from the
release's notes ("SHA-256 Checksums"), or each asset's digest
(`gh release view -R actions/runner --json assets`); and the version the
image's `tests.yaml` expects.
