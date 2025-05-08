---
tags:
  - k8s
---

## Quality of Service classes

**Source:** https://kubernetes.io/docs/concepts/workloads/pods/pod-qos/#quality-of-service-classes

Kubernetes classifies the Pods that you run and allocates each Pod into a specific _quality of service (QoS) class_. Kubernetes uses that classification to influence how different pods are handled. Kubernetes does this classification based on the [resource requests](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/) of the [Containers](https://kubernetes.io/docs/concepts/containers/) in that Pod, along with how those requests relate to resource limits. This is known as [Quality of Service](https://kubernetes.io/docs/concepts/workloads/pods/pod-qos/) (QoS) class. Kubernetes assigns every Pod a QoS class based on the resource requests and limits of its component Containers. QoS classes are used by Kubernetes to decide which Pods to evict from a Node experiencing [Node Pressure](https://kubernetes.io/docs/concepts/scheduling-eviction/node-pressure-eviction/). The possible QoS classes are `Guaranteed`, `Burstable`, and `BestEffort`. When a Node runs out of resources, Kubernetes will first evict `BestEffort` Pods running on that Node, followed by `Burstable` and finally `Guaranteed` Pods. When this eviction is due to resource pressure, only Pods exceeding resource requests are candidates for eviction.

## Criteria


|                | Guaranteed                        | Burstable                             | BestEffort |
| -------------- | --------------------------------- | ------------------------------------- | ---------- |
| CPU request    | required                          | required                              | missing    |
| CPU limit      | required, equal to CPU request    | optional, not equal to CPU request    | missing    |
| Memory request | required                          | required                              | missing    |
| Memory limit   | required, equal to Memory request | optional, not equal to Memory request | missing    |
## Eviction Order

```d2
shape: step

best-effort: BestEffort {
	shape: step
}
burstable: Burstable {
	shape: step
}

guaranteed: Guaranteed {
	shape: step
}
```