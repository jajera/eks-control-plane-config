---
icon: lucide/sliders-horizontal
---

# Parameters

Amazon EKS continues to run the control plane processes. You can set four fields. Each has a validated range. Amazon EKS applies the change with a rolling control plane update and records it in AWS CloudTrail.

!!! note "AWS CLI"
    Commands on this page require AWS CLI **2.36.21** or later. See [Requirements](index.md#requirements).

| Component | Field | Default | Allowed values |
|---|---|---|---|
| kube-scheduler | `nodeResourcesFit.scoringStrategy` | `LeastAllocated` | `LeastAllocated` or `MostAllocated`; optional weights 1–100 |
| kube-controller-manager | `horizontalPodAutoscalerSyncPeriod` | `15s` | `10s`–`15s`; Provisioned Control Plane only |
| kube-apiserver | `eventTtl` | `60m` | `10m`–`60m` |
| kube-apiserver | `serviceNodePortRange` | `30000`–`32767` | `10260`–`32767` |

Resource weights may include `cpu`, `memory`, `nvidia.com/gpu`, `aws.amazon.com/neuron`, and `aws.amazon.com/neuroncore`. If the `resources` list contains only `cpu`, memory is excluded from scoring. That is not the same as `cpu: 100` and `memory: 1`. Accelerator weights apply only to pods that request those resources. Devices scheduled through Dynamic Resource Allocation are handled by a different plugin.

Settings apply to the whole cluster, not to a namespace. The minimum Kubernetes version is **1.31**. Updates merge: omitted fields keep their current values. There is no reset operation; set a field back to its default explicitly. Use `DescribeClusterVersions` for the defaults and constraints of each Kubernetes version.

## Event TTL

Kubernetes events such as `Scheduled`, `Pulled`, and `FailedScheduling` are stored in etcd. The default retention is 60 minutes. Reducing `eventTtl` does not change events that already exist. Expiry is set when the event is created.

If you rely on `kubectl get events` for investigation, export events elsewhere before shortening retention. Deleted events cannot be restored.

## NodePort range

The lower bound of **10260** avoids kubelet port `10248` and kube-proxy health port `10256`. The upper bound of **32767** stays below the typical Linux ephemeral port range. Narrowing the range does not change ports already assigned to Services. Recreating a Service allocates a port from the current range only.

## HPA sync period

The minimum interval is `10s`. The setting is available only on **Provisioned Control Plane**. Setting it on Standard fails. A non-default value prevents moving the control plane back to Standard until you restore `15s`.

Amazon EKS accepts the API call even if the cluster has more `HorizontalPodAutoscaler` objects than a 10-second loop can reconcile. In that case some objects skip cycles and scale more slowly than intended.

On Provisioned Control Plane, Amazon EKS also increased HPA sync concurrency (up to 40 times the upstream default). A 10-second period is an additional adjustment, not a substitute for that change.

## CLI

```bash
aws eks describe-cluster-versions --cluster-versions 1.36 \
  --query 'clusterVersions[0].controlPlaneComponentConfig'

aws eks describe-cluster --name "$CLUSTER" \
  --query 'cluster.{
      version: version,
      createdAt: createdAt,
      scheduler: kubeSchedulerConfig,
      controller: kubeControllerManagerConfig,
      apiserver: kubeApiServerConfig
    }'

aws eks update-cluster-config --name "$CLUSTER" \
  --kube-api-server-config '{"eventTtl":"45m"}'

aws eks wait cluster-active --name "$CLUSTER"
```

Live `describe-cluster` output, before and after Save, is on [Existing clusters](existing.md).
