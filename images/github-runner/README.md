# github-runner

A GitHub Actions runner for one job: `base` plus the runner
(`packages/actions-runner`), mise, bash (also `/bin/sh`), GNU coreutils,
findutils, sed, grep, awk, tar and gzip, curl, the OpenSSH client, gpg
and bubblewrap. It runs as `nonroot` (65532), with `HOME` at `/home/nonroot`.

bubblewrap (`bwrap`) is frontseat's sandbox: `frontseat publish` runs its
effects on a grid inside the job, which confines each action with it.
`bwrap` is not setuid; as nonroot it needs the pod to allow user
namespaces, a seccomp profile that lets it make them and an unmasked
`/proc`, which the controller gives runner boxes. Under a plain `docker
run` it reports `No permissions to create a new namespace`.

The controller mounts the runner's just-in-time configuration, a Secret, at
`/run/github-runner/jitconfig`; the entrypoint runs the runner once with
it:

```sh
exec /usr/share/actions-runner/run.sh --jitconfig "$(cat /run/github-runner/jitconfig)"
```

A JIT runner takes one job, never updates itself, and ends. The runner's
directory, `/usr/share/actions-runner`, is nonroot's: the runner writes its
configuration and logs (`_diag`) there, and jobs work in its `_work`, the
image's working directory. `RUNNER_MANUALLY_TRAP_SIG` is set, so a SIGTERM
reaches the listener and ends the job. `apko.yaml` says why each package
is there.

`ghcr.io/frontseat-dev/github-runner`
