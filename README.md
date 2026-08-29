# Amazon EKS control plane parameters

Amazon EKS (12 August 2026) lets you set four kube-apiserver, kube-scheduler, and kube-controller-manager fields through the EKS API. Existing clusters keep their previous behaviour until you call `UpdateClusterConfig`.

**Live site:** [eks-control-plane-config.johna.kiwi](https://eks-control-plane-config.johna.kiwi/)

This repo adds context to the [What's New announcement](https://aws.amazon.com/about-aws/whats-new/2026/08/amazon-eks-control-plane-configuration-parameters/) and the [Amazon EKS user guide](https://docs.aws.amazon.com/eks/latest/userguide/control-plane-configuration.html). The AWS walkthrough shows how to enable `MostAllocated`. These pages describe what that change does and does not do.

## At a glance

| Component | Parameter | Default | Range |
|---|---|---|---|
| kube-scheduler | Scoring strategy | `LeastAllocated` | `LeastAllocated` · `MostAllocated` |
| kube-controller-manager | HPA sync period | `15s` | `10s`–`15s` (Provisioned only) |
| kube-apiserver | Event TTL | `60m` | `10m`–`60m` |
| kube-apiserver | NodePort range | `30000`–`32767` | `10260`–`32767` |

Requires Kubernetes **1.31** or later. Updates apply with a rolling control plane update and affect only new scheduling decisions and new events.

AWS CLI **2.36.21** or later is required for `--kube-scheduler-config` / `--kube-api-server-config` / `--kube-controller-manager-config`. The console does not depend on the CLI version.

## What this does not do

- **Move running pods.** Scoring applies to new binds. Pods already on a node stay there.
- **Replace a full `KubeSchedulerConfiguration`.** Only the four fields above are supported. `RequestedToCapacityRatio` and arbitrary flags are still unavailable.
- **Apply automatically.** A cluster created before 12 August 2026 keeps its defaults until you update it.
- **Create or delete nodes.** `MostAllocated` places pods on nodes that already exist. Karpenter, Auto Mode, or Cluster Autoscaler still decide whether nodes should exist.

## Pages

| Page | Purpose |
|------|---------|
| [docs/index.md](docs/index.md) | Overview, requirements, reading order |
| [docs/prior.md](docs/prior.md) | Before this feature: spread-only default and custom schedulers |
| [docs/existing.md](docs/existing.md) | Existing clusters: `null` API fields vs unchanged behaviour |
| [docs/parameters.md](docs/parameters.md) | Defaults, ranges, CLI examples, operational limits |
| [docs/layers.md](docs/layers.md) | Scheduler vs provisioner |

## Preview locally

```bash
python3 -m venv .venv
.venv/bin/pip install -r requirements.txt
.venv/bin/zensical serve
```

Open the URL printed by the server (usually `http://127.0.0.1:8000`). Push to `main` deploys GitHub Pages (source: GitHub Actions).
