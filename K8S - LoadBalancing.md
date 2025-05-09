---
tags:
  - k8s
---
**Source:** https://learnk8s.io/kubernetes-long-lived-connections

> **TL;DR**
> Kubernetes doesn't load balance long-lived connections, and some Pods might receive more requests than others. Consider client-side load balancing or a proxy if you're using HTTP/2, gRPC, RSockets, AMQP, or any other long-lived database connection.

## IPTables

![iptables rules for three Pods](https://learnk8s.io/a/fbbcbf56da96099c15314b9af4a0dd77.svg)
By default, Kubernetes uses iptables to implement Services.
- Does iptables use round-robin for load balancing?_
- No, iptables are primarily used for firewalls and are not designed for load balancing.

However, you could [craft an intelligent set of rules to make iptables behave like a load balancer](https://scalingo.com/blog/iptables#load-balancing)
And this is precisely what happens in Kubernetes.
If you have three Pods, kube-proxy writes the following rules:

1. With a likelihood of 33%, select Pod 1 as the destination. Otherwise, proceed to the following rule.
2. With a probability of 50%, choose Pod 2 as the destination. Otherwise, proceed to the following rule.
3. Select Pod 3 as the destination (no probability).

The compound probability is that Pod 1, Pod 2, and Pod 3 have a one-third chance (33%) of being selected.

> **⚠️ Warning**
> When the workload starts to autoscale. You might see an uneven distribution of RPS per pod, due to some of the pods handling more connections than others.

## Solution

| Solution                                                                 | Notes                                                                                                                                                                        |
| ------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Client-side loadbalancing                                                | Expensive to implement                                                                                                                                                       |
| [IPVS](https://learn.microsoft.com/en-us/azure/aks/configure-kube-proxy) | Needs to be enabled as an extension. Does not support Azure Network Policy.                                                                                                  |
| ServiceMesh                                                              | Delegates network traffic-handling responsibility to the service mesh. May be initially expensive to implement, however this opens up much more possibilities in the future. |
