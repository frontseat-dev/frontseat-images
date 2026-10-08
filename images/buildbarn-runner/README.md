# buildbarn-runner

The root filesystem Buildbarn's build actions run in: `base` plus a C
toolchain, Python, a virtual display for SWT/GTK UI tests (Xvfb, GTK 3,
fonts), ICU for .NET, the Docker CLI, and mise, which provides each
workspace's own toolchains. bb_runner itself is staged in by its installer;
actions run as uid 65534. `/bin/sh` is dash, as on Debian: `xvfb-run`
depends on it. `apko.yaml` says why each package is there.

`ghcr.io/frontseat-dev/buildbarn-runner`
