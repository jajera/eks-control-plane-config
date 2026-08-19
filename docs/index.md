---
icon: lucide/cpu
---

<div class="patina-hero" markdown>

<p class="patina-kicker">Amazon EKS · 12 August 2026</p>

# Amazon EKS control plane parameters

<p class="patina-lede">
  Amazon EKS now lets you configure four parameters on kube-apiserver,
  kube-scheduler, and kube-controller-manager through the EKS API.
  Existing clusters keep their previous behaviour until you explicitly
  call <code>UpdateClusterConfig</code>.
</p>

<div class="patina-actions">
<a class="patina-btn patina-btn--primary" href="parameters/">See the parameters</a>
<a class="patina-btn patina-btn--ghost" href="existing/">Existing clusters</a>
</div>

</div>

## At a glance

| Component | Parameter | Default | Range |
|---|---|---|---|
| kube-scheduler | Scoring strategy | `LeastAllocated` | `LeastAllocated` · `MostAllocated` |
| kube-controller-manager | HPA sync period | `15s` | `10s`–`15s` (Provisioned only) |
| kube-apiserver | Event TTL | `60m` | `10m`–`60m` |
| kube-apiserver | NodePort range | `30000`–`32767` | `10260`–`32767` |

Requires Kubernetes **1.31** or later. Updates apply with a rolling control plane update and affect only new scheduling decisions and new events.

## Requirements

| Requirement | Minimum |
|---|---|
| Cluster Kubernetes version | **1.31** |
| AWS CLI (only if you use the CLI, not the console) | **2.36.21** — first release with `--kube-scheduler-config` / `--kube-api-server-config` / `--kube-controller-manager-config` |
| Provisioned Control Plane | Only if you set HPA sync period below **15s** |

The console does not depend on the CLI version. Upgrade AWS CLI v2 with the [bundled installer](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) (`./aws/install --update`). Do not use `pip install awscli` for v2.

## Why this matters

<div class="path-grid">
<a class="path-card" href="prior/">
<span class="path-card__label">Before</span>
<strong>Spread was the only option</strong>
<p>The EKS scheduler used <code>LeastAllocated</code>. Bin-packing required running a second scheduler and annotating every pod with <code>spec.schedulerName</code>.</p>
<span class="path-card__meta">roadmap #1468</span>
</a>
<a class="path-card" href="existing/">
<span class="path-card__label path-card__label--warn">Existing clusters</span>
<strong>No behaviour change on launch day</strong>
<p><code>DescribeCluster</code> returns the new fields as <code>null</code> until you Save. Scheduling does not change on launch day.</p>
<span class="path-card__meta">new binds only</span>
</a>
<a class="path-card" href="layers/">
<span class="path-card__label">Interaction</span>
<strong>Scheduler ≠ provisioner</strong>
<p><code>MostAllocated</code> places pods on existing nodes. Karpenter or Auto Mode decides whether nodes should exist at all.</p>
<span class="path-card__meta">two layers</span>
</a>
</div>

## What this does not do

- **Move running pods.** Scoring applies to new scheduling decisions. Pods already bound to a node stay there.
- **Replace a full `KubeSchedulerConfiguration`.** As of August 2026, only the four fields above are supported. `RequestedToCapacityRatio` and arbitrary flags are still unavailable.
- **Apply automatically.** A cluster created before 12 August 2026 keeps its defaults until you explicitly update it.

## Contents

<div class="nav-grid">
<a class="nav-card" href="prior/">
<strong>1. Before this feature</strong>
<span>Custom schedulers, GKE comparison, and what was unsupported</span>
</a>
<a class="nav-card" href="existing/">
<strong>2. Existing clusters</strong>
<span>API response versus scheduler behaviour; running pods stay bound</span>
</a>
<a class="nav-card" href="parameters/">
<strong>3. Parameters</strong>
<span>Defaults, validated ranges, CLI examples, and operational limits</span>
</a>
<a class="nav-card" href="layers/">
<strong>4. Scheduler and provisioner</strong>
<span><code>MostAllocated</code> places pods; it does not create or delete nodes</span>
</a>
</div>

!!! note "Relation to AWS documentation"
    This site adds context to the [What's New announcement](https://aws.amazon.com/about-aws/whats-new/2026/08/amazon-eks-control-plane-configuration-parameters/) and the [Amazon EKS user guide](https://docs.aws.amazon.com/eks/latest/userguide/control-plane-configuration.html). The AWS walkthrough shows how to enable `MostAllocated`. These pages describe what that change does and does not do.
