---
tags:
  - k8s/autoscaling
---
Horizontal scaling means that the response to increased load is to deploy more [Pods](https://kubernetes.io/docs/concepts/workloads/pods/). This is different from _vertical_ scaling, which for Kubernetes would mean assigning more resources (for example: memory or CPU) to the Pods that are already running for the workload.

Horizontal workload scaling in K8S can be achieved in mutiple ways. Most popular methods are: 
- [[K8S Horizontal Pod Autoscaler (HPA)]]
- [[KEDA]]
