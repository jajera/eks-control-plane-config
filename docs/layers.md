---
icon: lucide/layers
---

# Scheduler and provisioner

`MostAllocated` scores **nodes that already exist**. Karpenter, EKS Auto Mode, and Cluster Autoscaler operate at a different layer: they create and delete nodes.

<div class="layers">
<div class="layer">
<span>Layer 2 — provisioner</span>
<strong>Whether a node should exist</strong>
<p>Karpenter, Auto Mode, or Cluster Autoscaler. An empty node with consolidation enabled can be removed.</p>
</div>
<div class="layer">
<span>Layer 1 — scheduler</span>
<strong>Which existing node receives the pod</strong>
<p><code>LeastAllocated</code> prefers nodes with more unused capacity. <code>MostAllocated</code> prefers nodes that are already more allocated. Node affinity, taints, and topology spread constraints still take precedence.</p>
</div>
</div>

Packing without consolidation uses existing nodes more fully but does not reduce node count. Consolidation without packing continues to place pods on nodes that would otherwise become empty.

## After enabling MostAllocated

<div class="occ">
<p class="occ__label">Node A — 90% requested — preferred for the next pod</p>
<div class="occ__bar"><span class="occ--pack" style="width:90%"></span></div>
</div>
<div class="occ">
<p class="occ__label">Node B — 28% requested — lower score</p>
<div class="occ__bar"><span style="width:28%"></span></div>
</div>
<div class="occ">
<p class="occ__label">Node C — 0% requested — eligible for consolidation</p>
<div class="occ__bar"><span class="occ--empty" style="width:0%"></span></div>
</div>

Limits:

- Filtering is unchanged. A pod that requests 4 CPU is not placed on a node with 500m remaining, under either strategy.
- More pods per node increases the impact of a node or Availability Zone failure.
- Densely packed nodes reach capacity sooner. During high pod churn, pods may stay `Pending` until the provisioner adds nodes.
- Pods that are already running stay on their current nodes until they are evicted or recreated.
