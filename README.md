# workflows

Reusable GitHub Actions workflows for my projects.

## build-deploy

[`build-deploy.yml`](.github/workflows/build-deploy.yml) builds a project's Docker image(s),
pushes them to `ghcr.io/<owner>/<name>` and pins the new version in a GitOps repo. Dockhand
redeploys from that repo via webhook. No `latest`: the version in the GitOps repo is what
runs, and a rollback is a `git revert` there.

| Event | Images | Git tag | Deploy |
| --- | --- | --- | --- |
| Push to default branch | `1.0.4`, `1.0`, `1`, `sha-<commit>` | `v1.0.4` (next patch, created after a successful build) | `1.0.4` |
| Tag `v1.1.0` (minor/major) | `1.1.0`, `1.1`, `1`, `sha-<commit>` | - | `1.1.0` |
| Push to another branch | `sha-<commit>` | - | - |
| Pull request | built, not pushed | - | - |

The deployed version is passed to the build as `APP_VERSION`. Declare `ARG APP_VERSION` and
pass it on as `ENV` in the Dockerfile to show it in the app.

The GitOps commit only touches the `image: ghcr.io/<owner>/<name>:<tag>` lines of the project's
images. The logs only show those lines, so public repos don't expose the compose file.

### Use it

1. Copy [`templates/caller.yml`](templates/caller.yml) (or
   [`caller-with-tests.yml`](templates/caller-with-tests.yml)) to
   `.github/workflows/build-deploy.yml`. Only set inputs that differ from the convention.
2. Set the `GITOPS_TOKEN` secret: a fine-grained PAT with *Contents: Read and write* on the
   GitOps repo only. One PAT can serve all projects.
   ```sh
   gh secret set GITOPS_TOKEN -R <owner>/<project> < token.txt
   ```
3. The stack needs to exist in the GitOps repo with the images written out in full
   (`image: ghcr.io/<owner>/<name>:<some tag>`). How to set it up in Dockhand is described in
   the GitOps repo's README.
