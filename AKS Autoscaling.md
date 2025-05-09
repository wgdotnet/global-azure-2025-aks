Like any Kubernetes cluster, AKS can autoscale it's workload. In addition - thanks to leveraging Azure constructs, AKS can autoscale it's nodepools as needed in case it is no longer able to schedule pods. 

The following topics make up the AKS autoscaling.

## Autoscaling Prerequisites

A refresher on basic k8s topics is required in order to understand how the pods need to behave before they start autoscaling.

- [[K8S Pod Lifecycle]]
- [[K8S QoS Classes]]
- [[K8S - LoadBalancing]]

## Autoscaling 

- [[K8S Autoscaling - Pods]]
- [[AKS Autoscaling - Nodes]]
