**Source:** https://learn.microsoft.com/en-us/azure/aks/cluster-autoscaler-overview

![[aks-cluster-autoscaler.png]]

## Cluster Autoscaler 

To keep up with application demands in Azure Kubernetes Service (AKS), you might need to adjust the number of nodes that run your workloads. The cluster autoscaler component watches for pods in your cluster that can't be scheduled because of resource constraints. When the cluster autoscaler detects unscheduled pods, it scales up the number of nodes in the node pool to meet the application demand. It also regularly checks nodes that don't have any scheduled pods and scales down the number of nodes as needed.

> **⚠️ Caution**
> The cluster autoscaler **scales up based on pending pods** rather than CPU/Memory pressure on nodes.

### Cluster Autoscaler Settings

**Source:** https://learn.microsoft.com/en-us/azure/aks/cluster-autoscaler?tabs=azure-cli#cluster-autoscaler-profile-settings

> **Note**
> The cluster autoscaler profile affects **all node pools** that use the cluster autoscaler. You can't set an autoscaler profile per node pool. When you set the profile, any existing node pools with the cluster autoscaler enabled immediately start using the profile.

## Cluster Autoscaler & Burstable Workloads

- For **clusters concurrently hosting both long-running workloads, like web apps, and short/bursty job workloads**, we recommend separating them into distinct node pools with [Affinity Rules](https://learn.microsoft.com/en-us/azure/aks/operator-best-practices-advanced-scheduler#node-affinity)/[expanders](https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/FAQ.md#what-are-expanders) or using [PodDisruptionBudget](https://kubernetes.io/docs/tasks/run-application/configure-pdb/) to help prevent unnecessary node drain or scale down operations. Specifying the annotation [cluster-autoscaler.kubernetes.io/safe-to-evict: "false"](https://kubernetes.io/docs/reference/labels-annotations-taints/#cluster-autoscaler-kubernetes-io-safe-to-evict) on the Pod spec will also prevent the pods from being evicted. Use this annotation with caution, as it may cause the Cluster Autoscaler encounter issues when draining a node with a running Pod that includes this annotation.

> **⚠️ Caution**
> The cluster autoscaler may be unable to scale down if the pods from the node are unable to move

## Cluster Autoscaler Expanders

**Source:** https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/FAQ.md#what-are-expanders

Expanders provide different strategies for selecting the node group to which new nodes will be added.

Currently Cluster Autoscaler has 5 expanders:

- `random` - should be used when you don't have a particular need for the node groups to scale differently.
- `most-pods` - selects the node group that would be able to schedule the most pods when scaling up. This is useful when you are using nodeSelector to make sure certain pods land on certain nodes. Note that this won't cause the autoscaler to select bigger nodes vs. smaller, as it can add multiple smaller nodes at once.
- `least-waste` - this is the default expander, selects the node group that will have the least idle CPU (if tied, unused memory) after scale-up. This is useful when you have different classes of nodes, for example, high CPU or high memory nodes, and only want to expand those when there are pending pods that need a lot of those resources.
- `least-nodes` - selects the node group that will use the least number of nodes after scale-up. This is useful when you want to minimize the number of nodes in the cluster and instead opt for fewer larger nodes. Useful when chained with the `most-pods` expander before it to ensure that the node group selected can fit the most pods on the fewest nodes.
- `price` - select the node group that will cost the least and, at the same time, whose machines would match the cluster size. This expander is described in more details [HERE](https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/proposals/pricing.md). Currently it works only for GCE, GKE and Equinix Metal (patches welcome.)
- `priority` - selects the node group that has the highest priority assigned by the user. It's configuration is described in more details [here](https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/expander/priority/readme.md)