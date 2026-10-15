# Shared GitHub workflows

## `k8s-deploy.yml`

Build an app image to GHCR and deploy it to the zonnestralen k3s cluster with the app's
own ARC runner. Setup of the cluster side (namespace, RBAC, runner): `infra-tg`.

```yaml
# .github/workflows/deploy.yml in an app repo
name: build and deploy
on:
  push:
    branches: [master]
  workflow_dispatch:
jobs:
  deploy:
    uses: Davvos11/.github/.github/workflows/k8s-deploy.yml@v1
    with:
      app: my-app            # = namespace
      # rollout: ""          # CronJob-only app: skip rollout status
    permissions:
      contents: read
      packages: write
```

Inputs: `app` (required), `context`, `dockerfile`, `k8s-dir` (default `k8s`), `rollout`
(Deployments to wait for, default `app`), `rollout-timeout`, `runs-on` (default `zonnestralen`).

**Private dependencies** (e.g. a private git dependency in `Cargo.toml`): pass a GitHub App with read access
to those repos. The build gets a short-lived token as Docker build secret `github_token`:

```yaml
    with:
      app: my-app
      build-app-id: ${{ vars.MY_APP_ID }}
      build-app-repositories: private-dep   # repos of the caller's owner, comma-separated
    secrets:
      build-app-private-key: ${{ secrets.MY_APP_PRIVATE_KEY }}
```

```dockerfile
RUN --mount=type=secret,id=github_token,required=false \
    export GIT_CONFIG_COUNT=1 \
        GIT_CONFIG_KEY_0="url.https://x-access-token:$(cat /run/secrets/github_token)@github.com/.insteadOf" \
        GIT_CONFIG_VALUE_0="https://github.com/" \
    && <build>
```

Setting git's config through environment variables keeps the token out of the image layers and the build cache.

Releases: tag `vX.Y.Z` and move the `v1` tag (`git tag -f v1 && git push -f origin v1`).
