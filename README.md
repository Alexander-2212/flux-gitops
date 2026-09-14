# flux-gitops

GitOps repository reconciled by [Flux](https://fluxcd.io/) in the kbot clusters.
Created and bootstrapped by Terraform from
[devops101-m07-infrastructure](https://github.com/Alexander-2212/devops101-m07-infrastructure).

## Layout

```
clusters/
  kind/flux-system/   Flux components for the local kind cluster (written by bootstrap)
  kind/apps.yaml      -> apps/kbot
  gke/flux-system/    Flux components for GKE (written by bootstrap)
  gke/apps.yaml       -> apps/kbot
apps/kbot/
  namespace.yaml
  gitrepository.yaml  kbot repo, branch develop
  helmrelease.yaml    chart ./helm from that repo
```

## How a code change reaches the cluster

1. Push to `develop` in [kbot](https://github.com/Alexander-2212/kbot).
2. GitHub Actions tests, builds and pushes
   `ghcr.io/alexander-2212/kbot:<version>-linux-amd64`, then commits the new
   tag to `helm/values.yaml`.
3. The `kbot` GitRepository sees a new revision; `reconcileStrategy: Revision`
   makes the HelmRelease upgrade even though the chart version did not change.
4. Helm rolls the deployment to the new image.

## Secret

The chart expects a `kbot-token` Secret with the Telegram token. It is created
by hand in each cluster and never committed:

```bash
kubectl -n kbot create secret generic kbot-token --from-literal=token="$TELE_TOKEN"
```

Flux authenticates to this repository with an SSH deploy key; the private key
lives only in Terraform state and the `flux-system` Secret in the cluster.
