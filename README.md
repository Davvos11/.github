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

Releases: tag `vX.Y.Z` and move the `v1` tag (`git tag -f v1 && git push -f origin v1`).
