# PhotoPrism: upstream sync, Linux builds, and release publishing

## Repository policy
- Official upstream: https://github.com/photoprism/photoprism, branch `develop`.
- This fork's `develop` is safe-fast-forwarded to official `develop`.
- The repository default branch is `arch/armv7`; it retains its own history and local changes. GitHub schedule and dispatch workflows live there, outside the tracked source branch.
- Ref `photoprism-autosync-state` records the last **dispatched** commit SHA, not build success.

## Workflow
- Sync: every 3 hours UTC (minute 23), manual dispatch, and initial push bootstrap.
- Check official `develop`, update the fork only by fast-forward, and reject divergence rather than force-overwrite.
- Only when the upstream source SHA differs from the dispatch state, trigger `build-release.yml` on the default branch using the exact source SHA.
- Native Linux AMD64 and ARM64 runners build the upstream production `docker/photoprism/resolute/Dockerfile`. This official Dockerfile calls `make all install`, compiling the web interface and backend and installing runtime payloads.
- For each architecture, smoke-test the program in the built image and export `/opt/photoprism` to a Linux tar.gz and SHA256 checksum.
- Push per-architecture Docker images to GHCR. After both jobs succeed, create prerelease `dev-<SHA12>` and attach archives, then combine images into multiarch `ghcr.io/969981/photoprism:dev-<SHA12>` and `:edge`.

## Limits and operating notes
- PhotoPrism is a Linux server app. Its tarballs require appropriate Linux system dependencies, not Windows or native macOS executables.
- Docker builds may be large, slow, and expensive in Actions minutes, storage and GHCR storage; monitor quotas.
- The official Dockerfile relies on upstream development/base images and downloads which may change or fail.
- Workflow dispatch is not a success signal. Failed builds require manual re-run; they are not retriggered automatically on the same SHA.
- Actions permissions must allow contents and actions write on the sync workflow, contents/package write on publication; GHCR packages may need public visibility configured.
- To retry: Actions -> `PhotoPrism Linux builds and Releases` -> Run workflow on `arch/armv7`, input full upstream SHA.
- GitHub schedules are best effort. Releases are **unofficial development snapshots**, not PhotoPrism stable releases.
- Existing custom `arch/armv7` changes remain untouched by upstream sync.

## URLs
- https://github.com/969981/photoprism/actions
- https://github.com/969981/photoprism/releases
- https://github.com/969981/photoprism/pkgs/container/photoprism
- https://github.com/photoprism/photoprism/blob/develop/setup/pkg/linux/README.md
