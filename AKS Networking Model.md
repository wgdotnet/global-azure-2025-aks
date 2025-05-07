## Available AKS Networking models 

- [[AKS Legacy Networking - Kubenet]]
- [[AKS Legacy Networkling - Azure CNI]]
- [[AKS Networking - CNI Overlay]]

## Summary

### Features
| Model             | Azure Network Policies | Calico Network Policies | Cilium Network Policies | Pods reachable directly | Windows node pools | Virtual Nodes | Nodepool per subnet |
| ----------------- | ---------------------- | ----------------------- | ----------------------- | ----------------------- | ------------------ | ------------- | ------------------- |
| kubenet           | -                      | +                       | -                       | -                       | -                  | -             | -                   |
| azure-cni         | +                      | +                       | -                       | +                       | +                  | +             | +                   |
| azure-cni-overlay | +                      | +                       | +                       | -                       | +                  | -             | +                   |
### When to use 

| **Model**         | **IP Address Space** | **Pod communication**                      | Pods reachable from the outside | Advanced AKS  features |
| ----------------- | -------------------- | ------------------------------------------ | ------------------------------- | ---------------------- |
| kubenet           | limited              | mostly internal                            | not needed                      | not needed             |
| azure-cni         | available            | mostly to resources outside of the cluster | needed                          | needed                 |
| azure-cni-overlay | limited              | mostly internal                            | not needed                      | needed                 |

### Scalability

| Model             | Max Pods/Node (default) | Max Pods/Node (Maximum) | Max nodes              | Max pods               |
| ----------------- | ----------------------- | ----------------------- | ---------------------- | ---------------------- |
| kubenet           | 110                     | 250                     | 400                    | 44,000-100,000         |
| azure-cni         | 30                      | 250                     | depends on subnet size | depends on subnet size |
| azure-cni-overlay | 250                     | 250                     | 5000                   | 1,250,000              |
