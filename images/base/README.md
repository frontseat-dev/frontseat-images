# base

What every Frontseat service runs on: Wolfi's base layout, the CA bundle,
timezones, git and the non-root user `nonroot` (65532), which it runs as.
No shell: services are statically linked Go programs, laid on top of it by
gib in frontseat-enterprise.

`ghcr.io/frontseat-dev/base`
