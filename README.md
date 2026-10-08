# Frontseat images

The images Frontseat runs on, built from [Wolfi](https://github.com/wolfi-dev)
packages with [apko](https://github.com/chainguard-dev/apko), for linux/amd64
and linux/arm64, published to `ghcr.io/frontseat-dev`, signed with cosign.

| Image | What it is |
|---|---|
| [`base`](images/base) | What every Frontseat service runs on: Wolfi's base layout, CA bundle, timezones, git, a non-root user |
| [`buildbarn-runner`](images/buildbarn-runner) | `base` plus the tools Buildbarn's build actions run with |

Wolfi does not have everything yet. What it lacks is built here with
[melange](https://github.com/chainguard-dev/melange), only for these images
(never published as packages):

| Package | |
|---|---|
| [`mise`](packages/mise) | until Wolfi has it |

## Layout

```
images/<name>/apko.yaml      what the image is made of
images/<name>/tests.yaml     what it must do (container-structure-test)
images/<name>/packages.txt   what it resolves to today, per architecture
packages/<name>/melange.yaml a package Wolfi lacks
```

The directory is the image's name: `images/base` is `ghcr.io/frontseat-dev/base`.

## Pipeline

| Workflow | When | What |
|---|---|---|
| `ci.yml` | every pull request and push to `main` | builds every package and image for both architectures and tests both; publishes nothing |
| `release.yml` | push to `main` | [release-please](https://github.com/googleapis/release-please) keeps a release pull request open; merging it tags `vX.Y.Z`, then every image is built, tested, published as `X.Y.Z`, signed, and listed with its digest in the GitHub release |
| `refresh.yml` | every night | resolves every image against Wolfi as it is now; when a package moved, opens `fix(images): Wolfi package updates` with the `packages.txt` diff |

`packages.yml` and `images.yml` are the steps the others share.

## Releasing

Commit with [conventional commits](https://www.conventionalcommits.org):
`feat:` for a new image or tool (a minor release), `fix:` for a fix (a
patch). release-please turns them into the next version and its changelog,
in a pull request titled `chore(main): release X.Y.Z`. Merge it to release.

Wolfi's security fixes arrive the same way: merge the nightly
`fix(images): Wolfi package updates`, then the release pull request.

Consumers pin a release by digest, e.g.
`ghcr.io/frontseat-dev/base:0.1.0@sha256:…`; the release lists them.

## Adding an image

1. `images/<name>/apko.yaml` (to build on base: `include: images/base/apko.yaml`)
2. `images/<name>/tests.yaml`
3. its name in the `matrix` of `.github/workflows/images.yml`
4. a `feat:` commit

## Updating the tools

The workflows' actions are kept current by Renovate. These are pinned by
checksum and moved by hand: apko and container-structure-test in
`.github/actions/setup/action.yml`, melange's image in
`.github/workflows/packages.yml`, mise in `packages/mise/melange.yaml`.

## Building locally

```sh
melange=cgr.dev/chainguard/melange
docker run --rm -v "$PWD":/work -w /work $melange keygen
docker run --rm --privileged -v "$PWD":/work -w /work $melange \
  build packages/mise/melange.yaml --arch aarch64 --signing-key melange.rsa --out-dir apks
apko build images/buildbarn-runner/apko.yaml buildbarn-runner:test image.tar --arch aarch64 --sbom=false \
  --repository-append "$PWD/apks" --keyring-append "$PWD/melange.rsa.pub"
docker load -i image.tar
container-structure-test test --image buildbarn-runner:test-arm64 --config images/buildbarn-runner/tests.yaml
```

SBOMs are off for now: apko 1.4.8 fails its own SBOM check on Wolfi's
`libLLVM` (which GTK pulls in).
