# Frontseat images

The base images Frontseat's services and builds run on, built from
[Wolfi](https://github.com/wolfi-dev) packages with
[apko](https://github.com/chainguard-dev/apko), plus the few packages Wolfi
does not have yet, built with [melange](https://github.com/chainguard-dev/melange).

| Image | Published as | What it is |
|---|---|---|
| `images/base` | `ghcr.io/frontseat-dev/base` | Wolfi's base layout, CA bundle, timezones, a non-root user: what a static Go program runs on |
| `images/git` | `ghcr.io/frontseat-dev/base-git` | `base` plus git |
| `images/runner` | `ghcr.io/frontseat-dev/buildbarn-runner` | the root filesystem Buildbarn's build actions run in (toolchain, Xvfb/GTK, ICU, Docker CLI, mise) |

| Package | Why it is here |
|---|---|
| `packages/mise.yaml` | Wolfi has no mise yet ([wolfi-dev/os#78684](https://github.com/wolfi-dev/os/pull/78684)); drop it when it lands |

`.github/workflows/build.yml` builds every package and image for amd64 and
arm64 every night and on every change to `main`, and publishes the images
signed with cosign (keyless), tagged `YYYYMMDD-<commit>`. The packages are
signed with a key made for the run and consumed in it; nothing else
publishes or hosts them. Consumers pin an image by digest; the run's
summary lists each image's tag and digest.

## Building locally

```sh
docker run --rm -v "$PWD":/work -w /work cgr.dev/chainguard/melange keygen
docker run --rm --privileged -v "$PWD":/work -w /work cgr.dev/chainguard/melange \
  build packages/mise.yaml --arch aarch64 --signing-key melange.rsa --out-dir apks
apko build images/runner/apko.yaml runner:local runner.tar --arch aarch64 --sbom=false \
  --repository-append "$PWD/apks" --keyring-append "$PWD/melange.rsa.pub"
docker load -i runner.tar
```
