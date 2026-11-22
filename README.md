# Frontseat images

**Small, secure images for everything Frontseat runs.**

- **Minimal by design.** Each image holds only what its job needs, runs as a non-root user, and starts from [Wolfi](https://github.com/wolfi-dev).

- **Always patched.** Images are rebuilt every night, so security fixes reach you without anyone lifting a finger.

- **Tested and signed.** Every image is tested before release and signed, so you know exactly what you run.

- **Pin and forget.** Every release lists each image's digest. Pin one and nothing changes under you.

## Images

- **`base`**: the foundation every Frontseat image builds on.
- **`node`**: `base` with Node.js, for Frontseat's web services.
- **`buildbarn-runner`**: `base` with the tools build actions run with.
- **`github-runner`**: `base` with the GitHub Actions runner.

Each is published at `ghcr.io/frontseat-dev/<name>`.

## Contributing

Want to change or add an image? See [CONTRIBUTING.md](CONTRIBUTING.md).
